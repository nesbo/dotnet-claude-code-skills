# EF Core Data Persistence Layer - Hexagonal Architecture Adapter

## Overview

This skill provides a comprehensive guide to implementing the data persistence layer using Entity Framework Core as an adapter in a hexagonal architecture (ports and adapters pattern). The data layer is a **library that encapsulates its implementation internally** and **exposes DI registration publicly** for wiring into applications.

This skill complements the `ddd-dotnet-basics.md` skill and assumes familiarity with Domain layer patterns.

### Key Architecture Principles

```
┌─────────────────────────────────────────────┐
│        Domain (Ports)                       │
│  - IRepository<TAggregate, TKey>            │
│  - IQueryRepository<TAggregate>             │
│  - IUnitOfWork                              │
│  - Aggregates & Entities                    │
└─────────────────────────────────────────────┘
                     ▲
                     │ (implements)
                     │
┌─────────────────────────────────────────────┐
│        Data (Adapter - Infrastructure)      │
│  - DbContext                                │
│  - Entity Configurations                    │
│  - Repository Implementations               │
│  - UnitOfWork Implementation                │
│  - Service Registrations (DI)               │
└─────────────────────────────────────────────┘
```

**Key Benefits:**
- **Testability**: In-memory database for unit/integration tests
- **Separation of Concerns**: Domain knows nothing about EF Core
- **CQRS Support**: Separate read/write repository base classes
- **Schema Organization**: Multiple database schemas for bounded contexts
- **Convention-based Registration**: Automatic repository discovery and registration

---

## Project Structure

```
Pikot.LMP.Adapters.Data/
├── DataContext.cs                    # Main DbContext
├── UnitOfWork.cs                     # Transaction coordinator
├── ServiceRegistrations.cs           # DI setup (public API)
├── EntityConfigurations/             # Fluent configurations
│   ├── ConfigHelper.cs               # Reusable config utilities
│   ├── ShipperOrderConfiguration.cs
│   ├── PackageConfiguration.cs
│   └── ...
├── Repositories/                     # Repository implementations
│   ├── RepositoryBase.cs             # Write repository base
│   ├── QueryRepositoryBase.cs        # Read repository base
│   ├── ShipperOrderRepository.cs
│   ├── ShipperOrderQueryRepository.cs
│   └── ...
└── Migrations/                       # EF Core migrations
    └── ...
```

---

## DbContext Configuration

### Main DbContext

The `DataContext` is the central EF Core context with specific patterns for production and testing.

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

namespace Pikot.LMP.Adapters.Data;

public class DataContext : DbContext
{
    public DataContext(DbContextOptions options) : base(options) { }

    public DataContext() { }  // Parameterless for design-time tools

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Fallback configuration for design-time tools (migrations)
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseNpgsql("Server=127.0.0.1;Database=lmp-nia;User Id=admin;Password=Password1!");
        }

        base.OnConfiguring(optionsBuilder);
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Auto-discover and apply all IEntityTypeConfiguration<T> from assembly
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(DataContext).Assembly);

        base.OnModelCreating(modelBuilder);
    }

    // Migration runner with retry logic (for application startup)
    public static async Task RunDbMigrationsAsync(IServiceProvider serviceProvider)
    {
        using var scope = serviceProvider.CreateScope();
        var logger = scope.ServiceProvider.GetRequiredService<ILogger<DataContext>>();

        const int maxAttempts = 5;
        var attempt = 1;
        var success = false;
        Exception? startupException = null;

        while (attempt < maxAttempts && !success)
        {
            try
            {
                logger.LogInformation("Attempting db migration run on startup. Attempt [{StartupMigrationAttempt}]",
                    attempt);
                using var dbScope = scope.ServiceProvider.CreateScope();
                var db = dbScope.ServiceProvider.GetRequiredService<DataContext>();
                await db.Database.MigrateAsync();
                logger.LogInformation("Db migration run successful");
                success = true;
                startupException = null;
            }
            catch (Exception ex)
            {
                startupException = ex;
                logger.LogError(ex, "Error accessing db for startup migration run on attempt [{StartupMigrationAttempt}]",
                    attempt);
                attempt++;
                await ExponentialBackoffAsync(attempt);
            }
        }

        if (attempt < maxAttempts || startupException == null)
        {
            return;
        }

        logger.LogCritical("Db migration run failed. Service run FAILED");
        throw new Exception("Critical stop", startupException);
    }

    private static Task ExponentialBackoffAsync(int attempt)
    {
        var waitInterval = TimeSpan.FromSeconds(Fibonacci(attempt));
        return Task.Delay(waitInterval);
    }

    private static int Fibonacci(int n)
    {
        if (n is 0 or 1)
        {
            return n;
        }
        return Fibonacci(n - 1) + Fibonacci(n - 2);
    }
}
```

**Key Patterns:**
- **Two constructors**: Parameterized for runtime, parameterless for design-time
- **Conditional configuration**: Fallback for migration tools
- **Assembly scanning**: Auto-discovers entity configurations
- **Migration runner**: Resilient startup with exponential backoff
- **Logging**: Track migration attempts and failures

---

## Entity Configurations

Entity configurations use the Fluent API to map domain objects to database tables. Each entity gets its own configuration class.

### Configuration Helper

Reusable utilities for common configuration patterns:

```csharp
using System.Linq.Expressions;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal static class ConfigHelper
{
    // Bulk set multiple properties as required
    public static void SetRequired<TEntityType>(this EntityTypeBuilder<TEntityType> builder,
        params Expression<Func<TEntityType, object>>[] setters) where TEntityType : class
    {
        foreach (var setter in setters)
        {
            var member = setter.Body;
            if (member == null)
            {
                throw new ArgumentNullException(nameof(setter.Body));
            }

            var propInfo = member is UnaryExpression expression
                ? expression.Operand as MemberExpression
                : member as MemberExpression;

            builder.Property(propInfo.Member.Name).IsRequired();
        }
    }

    public static void SetMaxLength<TEntityType, TProperty>(this EntityTypeBuilder<TEntityType> builder,
        Expression<Func<TEntityType, TProperty>> property, int maxLength)
        where TEntityType : class
    {
        builder.Property(property).HasMaxLength(maxLength);
    }

    // Database schemas for bounded contexts
    public const string DispatcherScheme  = "Dispatcher";
}
```

### Aggregate Configuration Example

Configuration for an aggregate root with child entities:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Pikot.LMP.Domain.Shipment;
using Pikot.LMP.Domain.Shipment.ShipperOrders;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal class ShipperOrderConfiguration : IEntityTypeConfiguration<ShipperOrder>
{
    public void Configure(EntityTypeBuilder<ShipperOrder> builder)
    {
        // Table mapping with schema
        builder.ToTable("ShipperOrders", ConfigHelper.ShipmentScheme)
            .HasKey(p => p.Id);

        // Primary key with custom column name
        builder.Property(p => p.Id)
            .HasColumnName("ShipperOrderId");

        // Bulk required properties
        builder.SetRequired(
            d => d.ShipperId,
            d => d.ShipperCompanyName);

        // String length constraints
        builder.Property(d => d.ShipperCompanyName).HasMaxLength(128);
        builder.Property(d => d.OrderNumber).HasMaxLength(64);

        // Nullable properties
        builder.Property(d => d.DispatcherId);
        builder.Property(d => d.DispatcherCompanyName).HasMaxLength(128);

        // One-to-many relationships (owned collections)
        builder.HasMany(d => d.Changes)
            .WithOne()
            .HasForeignKey(c => c.ShipperOrderId);

        builder.HasMany(d => d.Versions)
            .WithOne()
            .HasForeignKey(c => c.ShipperOrderId);

        builder.HasMany(d => d.Packages)
            .WithOne()
            .HasForeignKey(p => p.ShipperOrderId);

        // Indexes
        builder.HasIndex(d => new { d.ShipperId, d.OrderNumber }).IsUnique();
        builder.HasIndex(d => d.Guid);
    }
}
```

### Entity Configuration Example

Configuration for child entities:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Pikot.LMP.Domain.Shipment;
using Pikot.LMP.Domain.Shipment.ShipperOrders;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal class PackageConfiguration : IEntityTypeConfiguration<Package>
{
    public void Configure(EntityTypeBuilder<Package> builder)
    {
        builder.ToTable("Packages", ConfigHelper.ShipmentScheme)
            .HasKey(p => p.Id);

        builder.Property(p => p.Id)
            .HasColumnName("PackageId");

        // Database-generated GUID
        builder.Property(p => p.Guid)
            .HasDefaultValueSql("gen_random_uuid()");

        // Custom column types
        builder.Property(p => p.Numeration)
            .HasColumnType("smallint");

        // Enum to byte conversion
        builder.Property(p => p.Status)
            .HasColumnType("smallint")
            .HasConversion<byte>();

        builder.SetRequired(
            d => d.Kg,
            d => d.ShipperOrderId,
            d => d.Guid,
            d => d.Numeration,
            d => d.Status);

        // Decimal precision
        builder.Property(d => d.Kg)
            .HasColumnType("decimal(10,2)");
    }
}
```

### Simple Aggregate Configuration

Configuration for aggregates without child entities:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Pikot.LMP.Domain.Dispatcher;
using Pikot.LMP.Domain.Dispatcher.Dispatchers;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal class DispatcherConfiguration : IEntityTypeConfiguration<Dispatcher>
{
    public void Configure(EntityTypeBuilder<Dispatcher> builder)
    {
        builder.ToTable("Dispatchers", ConfigHelper.DispatcherScheme)
            .HasKey(p => p.Id);

        builder.Property(p => p.Id)
            .HasColumnName("DispatcherId");

        builder.SetRequired(
            d => d.Name,
            d => d.Itin,
            d => d.CreatedAt,
            d => d.InsertedAt,
            d => d.IsActive);

        builder.Property(d => d.Name).HasMaxLength(128);
        builder.Property(d => d.Itin).HasMaxLength(15);
        builder.Property(d => d.ContactEmail).HasMaxLength(128);
        builder.Property(d => d.ContactName).HasMaxLength(64);
        builder.Property(d => d.ContactPhone).HasMaxLength(16);

        builder.HasIndex(d => d.Itin).IsUnique();
    }
}
```

### One-to-One Relationship Configuration

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Pikot.LMP.Domain.Shipper;
using Pikot.LMP.Domain.Shipper.Shippers;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal class ShipperConfiguration : IEntityTypeConfiguration<Shipper>
{
    public void Configure(EntityTypeBuilder<Shipper> builder)
    {
        builder.ToTable("Shippers", ConfigHelper.ShipperScheme)
            .HasKey(p => p.Id);

        builder.Property(p => p.Id)
            .HasColumnName("ShipperId");

        builder.SetRequired(
            d => d.Name,
            d => d.Itin,
            d => d.CreatedAt,
            d => d.InsertedAt);

        builder.Property(d => d.Name).HasMaxLength(128);
        builder.Property(d => d.Itin).HasMaxLength(15);

        builder.HasIndex(d => d.Itin).IsUnique();

        // One-to-one relationship
        builder.HasOne(d => d.Settings)
            .WithOne()
            .HasForeignKey<ShipperSettings>(s => s.ShipperId);
    }
}
```

**Configuration Best Practices:**
1. **One class per entity** - Separate configuration files
2. **Internal visibility** - Configurations are implementation details
3. **Schema organization** - Group by bounded context
4. **Custom column names** for IDs - `EntityNameId` convention
5. **Bulk required properties** - Use `SetRequired` helper
6. **Explicit string lengths** - Prevent unlimited `nvarchar(max)`
7. **Type conversions** - Enums, decimals, custom types
8. **Indexes** - Unique constraints and performance indexes
9. **Database defaults** - Use `HasDefaultValueSql` for GUIDs, timestamps

---

## Repository Implementations

### Write Repository Base Class

Base class for command-side (write) repositories:

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal abstract class RepositoryBase<TAggregate, TExistsKey>(DataContext context)
    : IRepository<TAggregate, TExistsKey>
    where TAggregate : class, IAggregate
{
    protected DataContext Context => context;

    protected virtual IQueryable<TAggregate> Query => Context.Set<TAggregate>();

    public abstract Task<bool> ExistsAsync(TExistsKey key, CancellationToken cancellationToken);

    public virtual Task SaveChangesAsync(CancellationToken cancellationToken)
    {
        return Context.SaveChangesAsync(cancellationToken);
    }

    public virtual Task<TAggregate?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query.SingleOrDefaultAsync(x => x.Id == id, cancellationToken);
    }

    public virtual void Add(TAggregate entity)
    {
        Context.Add(entity);
    }
}
```

**Key Features:**
- **Internal visibility** - Implementation detail
- **Primary constructor** - Injects DataContext
- **Protected Query** - Subclasses can customize
- **Virtual methods** - Override for eager loading
- **Abstract ExistsAsync** - Each repository defines its own unique key check

### Query Repository Base Class

Base class for query-side (read) repositories:

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal abstract class QueryRepositoryBase<TAggregate>(DataContext dataContext) : IQueryRepository<TAggregate>
    where TAggregate : class, IAggregate
{
    protected DataContext DataContext => dataContext;

    protected IQueryable<TAggregate> Query => DataContext.Set<TAggregate>()
        .AsNoTracking();  // CRITICAL: No change tracking for reads

    public virtual Task<TAggregate?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query.SingleOrDefaultAsync(x => x.Id == id, cancellationToken);
    }
}
```

**Key Features:**
- **AsNoTracking()** - Critical for read performance
- **Simpler interface** - No writes or existence checks
- **Virtual Get** - Override for eager loading and includes

### Write Repository Implementation

Example repository for write operations:

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain.Shipment;
using Pikot.LMP.Domain.Shipment.Repositories;
using Pikot.LMP.Domain.Shipment.ShipperOrders;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal class ShipperOrderRepository(DataContext context)
    : RepositoryBase<ShipperOrder, int>(context), IShipperOrderRepository
{
    public override Task<bool> ExistsAsync(int key, CancellationToken cancellationToken)
    {
        return Query.AnyAsync(s => s.Id == key, cancellationToken);
    }

    public void Delete(ShipperOrder order)
    {
        Context.Set<ShipperOrder>().Remove(order);
    }

    // Override to include child collections
    public override Task<ShipperOrder?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query
            .Include(s => s.Changes)
            .Include(s => s.Versions)
            .Include(s => s.Packages)
            .SingleOrDefaultAsync(s => s.Id == id, cancellationToken);
    }

    public Task<ShipperOrder[]> ListAsync(int[] ids, CancellationToken cancellationToken)
    {
        return Query
            .Where(s => ids.Contains(s.Id))
            .Include(s => s.Changes)
            .Include(s => s.Versions)
            .Include(s => s.Packages)
            .ToArrayAsync(cancellationToken);
    }
}
```

### Query Repository Implementation

Example repository for read operations with complex queries:

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain.Common;
using Pikot.LMP.Domain.Shipment;
using Pikot.LMP.Domain.Shipment.Repositories;
using Pikot.LMP.Domain.Shipment.ShipperOrders;
using Pikot.LMP.Domain.Shipment.ShipperOrders.ListShipperOrders;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal class ShipperOrderQueryRepository(DataContext dataContext)
    : QueryRepositoryBase<ShipperOrder>(dataContext), IShipperOrderQueryRepository
{
    public async Task<ShipperOrderPagedResult> ListByStatusAndShipperAsync(
        ShipperOrderStatusType status,
        ,int shipperId,
        QueryPaging paging,
        CancellationToken token)
    {
        var query = Query
            .Where(s => s.ShipperId == shipperId)
            .Where(s => s.Status == status);

        var total = await query.CountAsync(token);

        var result = await query
            .Include(c => c.Changes)
            .Include(c => c.Versions)
            .OrderBy(s => s.CreatedAt)
            .Skip(paging.PageSize * (paging.PageNumber - 1))
            .Take(paging.PageSize)
            .ToArrayAsync(token);

        return new ShipperOrderPagedResult(paging.PageNumber,
            paging.PageSize, total, result);
    }

    public override Task<ShipperOrder?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query
            .Include(s => s.Versions)
            .Include(s => s.Changes)
            .Include(s => s.Packages)
            .SingleOrDefaultAsync(s => s.Id == id, cancellationToken);
    }

    public Task<ShipperOrder?> GetByGuidAsync(Guid guid, CancellationToken cancellationToken)
    {
        return Query
            .Include(s => s.Versions)
            .Include(s => s.Changes)
            .Include(s => s.Packages)
            .SingleOrDefaultAsync(s => s.Guid == guid, cancellationToken);
    }
}
```

**Repository Best Practices:**
1. **Override GetAsync** - Always include necessary child collections
2. **Use Include()** - Eager load related entities to avoid N+1 queries
3. **AsNoTracking** - Query repositories should never track changes
4. **Specific methods** - Add domain-specific query methods
5. **Pagination** - Use Skip/Take for large result sets
6. **Filter early** - Apply Where clauses before Include
7. **Count separately** - Get total before pagination for page info

---

## Unit of Work Implementation

Handles transactions across multiple repositories:

```csharp
using Microsoft.EntityFrameworkCore.Storage;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Adapters.Data;

internal class UnitOfWork(DataContext context) : IUnitOfWork
{
    private IDbContextTransaction? _transaction;

    public async Task BeginTransactionAsync(CancellationToken cancellationToken)
    {
        _transaction = await context.Database
            .BeginTransactionAsync(cancellationToken);
    }

    public Task CommitTransactionAsync(CancellationToken cancellationToken)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("Transaction has not been started");
        }

        return _transaction.CommitAsync(cancellationToken);
    }

    public Task RollbackTransactionAsync(CancellationToken cancellationToken)
    {
        if (_transaction == null)
        {
            throw new InvalidOperationException("Transaction has not been started");
        }

        return _transaction.RollbackAsync(cancellationToken);
    }

    public void Dispose()
    {
        _transaction?.Dispose();
        _transaction = null;
    }
}
```

**Key Patterns:**
- **Internal visibility** - Implementation detail
- **Null checks** - Validate transaction state
- **IDisposable** - Proper transaction cleanup

---

## Dependency Injection Registration

The **public API** of the data layer - the only public class:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Pikot.LMP.Adapters.Data.Repositories;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Adapters.Data;

public static class ServiceRegistrations
{
    private static IServiceCollection RegisterRepositories(this IServiceCollection services)
    {
        var missingRegistrations = new List<string>();

        // Find all repository interfaces in Domain assembly
        var domainRepositoryInterfaces = typeof(IRepository<,>).Assembly.GetTypes()
            .Where(t => (typeof(IRepository<,>).IsAssignableFrom(t) || typeof(IQueryRepository<>).IsAssignableFrom(t))
                        && t is { IsInterface: true, IsGenericType: false })
            .ToArray();

        // Find all repository implementations in Data assembly
        var repositoryTypes = typeof(ServiceRegistrations).Assembly.GetTypes()
            .Where(t => t is { IsAbstract: false, IsInterface: false, BaseType.IsGenericType: true, IsNotPublic: true }
                        && (t.BaseType.GetGenericTypeDefinition() == typeof(RepositoryBase<,>)
                            || t.BaseType.GetGenericTypeDefinition() == typeof(QueryRepositoryBase<>)))
            .ToArray();

        // Verify all domain interfaces have implementations
        foreach (var repositoryInterface in domainRepositoryInterfaces)
        {
            if (!repositoryTypes.Any(r => r.IsAssignableFrom(repositoryInterface)))
            {
                missingRegistrations.Add($"{repositoryInterface.Name} has no registered implementations.");
            }
        }

        if (missingRegistrations.Count != 0)
        {
            throw new Exception($"Missing repository implementations: {string.Join(", ", missingRegistrations)}");
        }

        // Auto-register all implementations
        foreach (var repositoryType in repositoryTypes)
        {
            var domainType = repositoryType
                .GetInterfaces()
                .First(i => !i.IsGenericType);
            services.AddScoped(domainType, repositoryType);
        }

        return services;
    }

    public static IServiceCollection RegisterDataServices(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("Postgres");

        return services
            .AddDbContext<DataContext>(options => options.UseNpgsql(connectionString))
            .RegisterRepositories()
            .AddScoped<IUnitOfWork, UnitOfWork>();
    }
}
```

**DI Registration Features:**
1. **Convention-based** - Auto-discovers repositories by reflection
2. **Validation** - Ensures all domain interfaces have implementations
3. **Fail-fast** - Throws on startup if implementations are missing
4. **Public extension method** - Only public API surface
5. **Configuration-driven** - Connection string from appsettings
6. **Scoped lifetime** - Proper DbContext scope management

**Usage in Application Startup:**

```csharp
// In Program.cs or Startup.cs
builder.Services.RegisterDataServices(builder.Configuration);
```

## Migration Workflow

### Creating Migrations

```bash
# From the data project directory
dotnet ef migrations add MigrationName

# From solution root
dotnet ef migrations add MigrationName --project Pikot.LMP.Adapters.Data
```

### Applying Migrations

**Option 1: Automatic on Startup (Recommended for Development)**

```csharp
// In Program.cs
await DataContext.RunDbMigrationsAsync(app.Services);
```

**Option 2: Manual via CLI**

```bash
dotnet ef database update
```

### Removing Last Migration

```bash
dotnet ef migrations remove
```

---

## Database Schema Organization

Use schemas to organize tables by bounded context:

```csharp
// In ConfigHelper.cs
public const string DispatcherScheme = "Dispatcher";

// In entity configuration
builder.ToTable("Orders", ConfigHelper.ShipmentScheme);
// Results in: Shipment.Orders table
```

**Benefits:**
- **Logical grouping** - Related tables together
- **Clear boundaries** - Visualizes bounded contexts
- **Permission management** - Schema-level database permissions
- **Naming conflicts** - Same table name in different schemas

---

## Common Patterns & Best Practices

### 1. Eager Loading Child Collections

**Always override GetAsync in repositories that need child collections:**

```csharp
public override Task<ShipperOrder?> GetAsync(int id, CancellationToken cancellationToken)
{
    return Query
        .Include(s => s.Changes)      // Load child collection
        .Include(s => s.Versions)     // Load child collection
        .Include(s => s.Packages)     // Load child collection
        .SingleOrDefaultAsync(s => s.Id == id, cancellationToken);
}
```

### 2. Handling Enums

```csharp
// Store as byte/smallint
builder.Property(p => p.Status)
    .HasColumnType("smallint")
    .HasConversion<byte>();
```

### 3. Decimal Precision

```csharp
builder.Property(d => d.Kg)
    .HasColumnType("decimal(10,2)");  // 10 digits, 2 decimal places
```

### 4. Database-Generated Values

```csharp
// PostgreSQL UUID generation
builder.Property(p => p.Guid)
    .HasDefaultValueSql("gen_random_uuid()");

// SQL Server
builder.Property(p => p.Guid)
    .HasDefaultValueSql("NEWID()");
```

### 5. Unique Indexes

```csharp
// Single column
builder.HasIndex(d => d.Itin).IsUnique();

// Composite
builder.HasIndex(d => new { d.ShipperId, d.OrderNumber }).IsUnique();
```

### 6. Query Performance Optimization

```csharp
// BAD: N+1 queries
var orders = await context.ShipperOrders.ToListAsync();
foreach (var order in orders)
{
    var packages = order.Packages;  // Lazy load - separate query per order!
}

// GOOD: Single query with eager loading
var orders = await context.ShipperOrders
    .Include(o => o.Packages)
    .ToListAsync();
```

### 7. Filtering with Navigation Properties

```csharp
// Access nested properties in queries
query = ordering.OrderBy switch
{
    OrderByType.PickupCity => query.OrderBy(
        s => s.Versions.OrderByDescending(v => v.Version).First().ShipperCity),
    _ => query.OrderBy(s => s.CreatedAt)
};
```

---

## Quick Reference Templates

### New Entity Configuration

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Pikot.LMP.Domain.YourContext;

namespace Pikot.LMP.Adapters.Data.EntityConfigurations;

internal class YourEntityConfiguration : IEntityTypeConfiguration<YourEntity>
{
    public void Configure(EntityTypeBuilder<YourEntity> builder)
    {
        builder.ToTable("YourEntities", ConfigHelper.YourScheme)
            .HasKey(p => p.Id);

        builder.Property(p => p.Id)
            .HasColumnName("YourEntityId");

        builder.SetRequired(
            d => d.CreatedAt,
            d => d.InsertedAt);

        // Add specific configurations
    }
}
```

### New Write Repository

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain.YourContext;
using Pikot.LMP.Domain.YourContext.Repositories;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal class YourEntityRepository(DataContext context)
    : RepositoryBase<YourEntity, YourUniqueKey>(context), IYourEntityRepository
{
    public override Task<bool> ExistsAsync(YourUniqueKey key, CancellationToken cancellationToken)
    {
        return Query.AnyAsync(e => e.SomeProperty == key.Value, cancellationToken);
    }

    public override Task<YourEntity?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query
            .Include(e => e.ChildCollection)
            .SingleOrDefaultAsync(e => e.Id == id, cancellationToken);
    }
}
```

### New Query Repository

```csharp
using Microsoft.EntityFrameworkCore;
using Pikot.LMP.Domain.YourContext;
using Pikot.LMP.Domain.YourContext.Repositories;

namespace Pikot.LMP.Adapters.Data.Repositories;

internal class YourEntityQueryRepository(DataContext dataContext)
    : QueryRepositoryBase<YourEntity>(dataContext), IYourEntityQueryRepository
{
    public override Task<YourEntity?> GetAsync(int id, CancellationToken cancellationToken)
    {
        return Query
            .Include(e => e.ChildCollection)
            .SingleOrDefaultAsync(e => e.Id == id, cancellationToken);
    }

    public async Task<YourEntityPagedResult> ListAsync(
        QueryPaging paging,
        CancellationToken cancellationToken)
    {
        var query = Query
            .Where(e => e.IsActive);

        var total = await query.CountAsync(cancellationToken);

        var result = await query
            .OrderBy(e => e.CreatedAt)
            .Skip(paging.PageSize * (paging.PageNumber - 1))
            .Take(paging.PageSize)
            .ToArrayAsync(cancellationToken);

        return new YourEntityPagedResult(paging.PageNumber, paging.PageSize, total, result);
    }
}
```

---

## Checklist for New Aggregate

When adding a new aggregate to the data layer:

- [ ] Create `YourEntityConfiguration : IEntityTypeConfiguration<YourEntity>`
- [ ] Configure table name, schema, and primary key
- [ ] Configure all properties (required, max length, types)
- [ ] Configure relationships (HasMany, HasOne)
- [ ] Configure indexes (unique constraints, performance)
- [ ] Create `YourEntityRepository : RepositoryBase<YourEntity, TKey>`
- [ ] Implement `ExistsAsync` with unique key logic
- [ ] Override `GetAsync` with `.Include()` for child collections
- [ ] Create `YourEntityQueryRepository : QueryRepositoryBase<YourEntity>`
- [ ] Override `GetAsync` with `.Include()` for child collections
- [ ] Add domain-specific query methods
- [ ] Run `dotnet ef migrations add YourEntityAdded`
- [ ] Review generated migration SQL
- [ ] Test with in-memory database
- [ ] Verify auto-registration in `ServiceRegistrations`

---

## Common Pitfalls to Avoid

1. **Don't forget `.Include()`** - Always eager load child collections
2. **Don't track queries** - Use `AsNoTracking()` for query repositories
3. **Don't use lazy loading** - Leads to N+1 queries
4. **Don't make repositories public** - Keep them internal
5. **Don't skip unique indexes** - Enforce business uniqueness at DB level
6. **Don't use unlimited strings** - Always set `HasMaxLength()`
7. **Don't forget enum conversions** - Specify column type explicitly
8. **Don't ignore decimal precision** - Use `HasColumnType("decimal(x,y)")`
9. **Don't skip null checks in UnitOfWork** - Validate transaction state
10. **Don't forget parameterless constructor** - Needed for EF Core design-time tools

---

## Integration with Domain Layer

The data layer implements ports defined in the domain:

```
Domain (Ports)                     Data (Adapters)
──────────────                     ───────────────
IRepository<T, TKey>      ────►    RepositoryBase<T, TKey>
IQueryRepository<T>       ────►    QueryRepositoryBase<T>
IUnitOfWork               ────►    UnitOfWork
IShipperOrderRepository   ────►    ShipperOrderRepository
```

**Dependency flow:**
- Domain defines interfaces (ports)
- Data implements interfaces (adapters)
- Application/API depends on both, wires via DI

---

This skill document provides production-ready patterns for implementing EF Core data persistence in a hexagonal architecture with full CQRS support, testability, and clean separation of concerns.
