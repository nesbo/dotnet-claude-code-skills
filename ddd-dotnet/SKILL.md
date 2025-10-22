# Domain-Driven Design (DDD) in .NET - Basics & Patterns

## Overview

This skill provides a comprehensive guide to implementing Domain-Driven Design patterns in .NET, based on real-world production code from the LMP project. These patterns promote clean architecture, maintainability, and testability.

### Key Benefits
- **Clear Separation of Concerns**: Domain logic isolated from infrastructure
- **CQRS Pattern**: Optimized read and write operations
- **Hexagonal Architecture**: Ports (domain) and Adapters (infrastructure)
- **Testability**: Domain logic testable without dependencies
- **Time Control**: Deterministic time handling for testing

---

## Core Building Blocks

### 1. Domain Object Hierarchy

All domain objects implement a marker interface at the root:

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IDomainObject;
```

### 2. Aggregates

Aggregates are consistency boundaries and entry points for domain operations.

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IAggregate : IDomainObject
{
    int Id { get; }
    DateTimeOffset CreatedAt { get; }
    DateTimeOffset InsertedAt { get; }
}
```

**Versioned Aggregates** track changes over time:

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IAggregateVersioned : IAggregate
{
    int Version { get; }
}
```

**Key Principles:**
- Aggregates are the only objects that can be retrieved from repositories
- They maintain invariants and enforce business rules
- They control access to their entities
- Use `CreatedAt` for business time, `InsertedAt` for audit/database time

### 3. Entities

Entities are objects with identity that belong to aggregates:

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IEntity : IDomainObject
{
    int Id { get; }
    DateTimeOffset CreatedAt { get; }
    DateTimeOffset InsertedAt { get; }
}
```

**Example Entity:**

```csharp
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Domain.Shipment.ShipperOrders;

public class Package : IEntity
{
    protected Package() { } // For EF Core

    public Package(decimal kg, DateTimeOffset createdAt, DateTimeOffset insertedAt, byte numeration)
    {
        Guid = Guid.NewGuid();
        Kg = kg;
        CreatedAt = createdAt;
        InsertedAt = insertedAt;
        Numeration = numeration;
        Status = PackageStatus.Pending;
    }

    public int Id { get; protected init; }
    public Guid Guid { get; protected init; }
    public DateTimeOffset CreatedAt { get; }
    public DateTimeOffset InsertedAt { get; }
    public decimal Kg { get; private set; }
    public byte Numeration { get; private set; }
    public PackageStatus Status { get; private set; }
    public int ShipperOrderId { get; protected init; }

    // Business logic methods
    public void UpdateWeight(decimal newKg)
    {
        Kg = newKg;
    }

    public void SetNumeration(byte numeration)
    {
        Numeration = numeration;
    }

    public void SetStatus(PackageStatus status)
    {
        Status = status;
    }
}
```

---

## Aggregate Root Pattern

**Example: ShipperOrder Aggregate**

```csharp
using Pikot.LMP.Domain.Common;
using Pikot.LMP.Domain.Common.Exceptions;

namespace Pikot.LMP.Domain.Shipment.ShipperOrders;

public class ShipperOrder : IAggregateVersioned
{
    protected ShipperOrder() { } // For EF Core

    public ShipperOrder(DateTimeOffset createdAt, DateTimeOffset insertedAt,
        int shipperId, string shipperCompanyName, /* ... other params */)
    {
        Guid = Guid.NewGuid();
        CreatedAt = createdAt;
        InsertedAt = insertedAt;
        Version = 1;

        // Initialize collections
        _changeEvents.Add(new ShipperOrderChangeEvent(/* ... */));
        _versions.Add(new ShipperOrderVersion(/* ... */));
    }

    // Properties
    public int Id { get; protected init; }
    public Guid Guid { get; protected init; }
    public int Version { get; private set; }
    public DateTimeOffset CreatedAt { get; }
    public DateTimeOffset InsertedAt { get; }
    public ShipperOrderStatusType Status { get; private set; }

    // Collections - private backing fields with public readonly access
    private readonly List<ShipperOrderChangeEvent> _changeEvents = [];
    public IReadOnlyCollection<ShipperOrderChangeEvent> Changes
    {
        get => _changeEvents;
        init => _changeEvents = value.ToList(); // For EF Core materialization
    }

    private readonly List<Package> _packages = [];
    public IReadOnlyCollection<Package> Packages
    {
        get => _packages.OrderBy(p => p.Numeration).ToList();
        init => _packages = value.ToList();
    }

    // Business methods with validation
    public void Pickup(int dispatcherUserId, DateTimeOffset timestamp, IClock clock,
        int dispatcherId, string dispatcherCompanyName)
    {
        if (Status is not (ShipperOrderStatusType.Created or ShipperOrderStatusType.Ready))
        {
            throw new InvalidTransitionException();
        }

        Status = ShipperOrderStatusType.PickedUp;
        DispatcherId = dispatcherId;
        DispatcherCompanyName = dispatcherCompanyName;

        foreach (var package in _packages)
        {
            package.SetStatus(PackageStatus.OutForDelivery);
        }

        var change = new ShipperOrderChangeEvent(timestamp, clock.Now, dispatcherUserId,
            ShipperOrderStatusType.PickedUp, ShipperOrderChangeEventType.StatusChanged,
            ShipperOrderChangeEventSourceType.Dispatcher);
        _changeEvents.Add(change);
    }

    public Package AddPackage(decimal kg, DateTimeOffset createdAt, DateTimeOffset insertedAt)
    {
        var numeration = (byte)(_packages.Count + 1);
        var package = new Package(kg, createdAt, insertedAt, numeration);
        _packages.Add(package);
        return package;
    }

    public void RemovePackage(int packageId)
    {
        var package = _packages.FirstOrDefault(p => p.Id == packageId);
        if (package == null)
            throw new NotFoundException("Package not found");

        _packages.Remove(package);
        RenumberPackages();
    }

    private void RenumberPackages()
    {
        byte numeration = 1;
        foreach (var package in _packages.OrderBy(p => p.Numeration))
        {
            package.SetNumeration(numeration);
            numeration++;
        }
    }
}
```

**Key Patterns:**
- Protected parameterless constructor for EF Core
- Public constructor with all required data
- Private setters on properties
- Private backing fields for collections with `IReadOnlyCollection<T>` exposure
- Business logic encapsulated in methods
- Validation within business methods
- Track domain events in collections

---

## Repository Pattern (CQRS)

### Write Repository

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IRepository<TAggregate, in TUniqueKey> where TAggregate : class, IAggregate
{
    Task<TAggregate?> GetAsync(int id, CancellationToken cancellationToken);
    Task<bool> ExistsAsync(TUniqueKey key, CancellationToken cancellationToken);
    Task SaveChangesAsync(CancellationToken cancellationToken);
    void Add(TAggregate entity);
}
```

### Query Repository

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IQueryRepository<TAggregate> where TAggregate : class, IAggregate
{
    Task<TAggregate?> GetAsync(int id, CancellationToken cancellationToken);
}
```

**Usage:**
- **Write operations**: Use `IRepository<TAggregate, TUniqueKey>` in command handlers
- **Read operations**: Use `IQueryRepository<TAggregate>` in query handlers
- Write repositories can check existence and add entities
- Query repositories are simpler and optimized for reads

### Specific Repository Interfaces

Define domain-specific repository interfaces in the Domain project:

```csharp
namespace Pikot.LMP.Domain.Shipment.Repositories;

public interface IShipperOrderRepository : IRepository<ShipperOrder, ShipperOrderUniqueKey>
{
    // Additional write-specific methods if needed
}

public interface IShipperOrderQueryRepository : IQueryRepository<ShipperOrder>
{
    Task<ShipperOrder?> GetByGuidAsync(Guid guid, CancellationToken cancellationToken);
    // Additional query methods
}
```

Implementations live in the Data project (adapter layer).

---

## Time Control Pattern

**NEVER use `DateTime.Now` or `DateTime.UtcNow` directly in domain code!**

### IClock Interface

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IClock
{
    DateTimeOffset Now { get; }
}
```

### Usage in Domain Code

```csharp
public void Complete(int completedByUserId, DateTimeOffset timestamp, IClock clock)
{
    Status = ShipperOrderStatusType.Delivered;

    var change = new ShipperOrderChangeEvent(
        timestamp,        // Business time (from command/user action)
        clock.Now,        // Audit time (when it was processed)
        completedByUserId,
        ShipperOrderStatusType.Delivered,
        ShipperOrderChangeEventType.StatusChanged,
        ShipperOrderChangeEventSourceType.Dispatcher);
    _changeEvents.Add(change);
}
```

**Benefits:**
- **Testable**: Inject mock clock for deterministic tests
- **Auditable**: Separate business time from processing time
- **Consistent**: All time handling centralized

---

## Unit of Work Pattern

For transactions spanning multiple repositories:

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IUnitOfWork
{
    Task BeginTransactionAsync(CancellationToken cancellationToken);
    Task CommitTransactionAsync(CancellationToken cancellationToken);
    Task RollbackTransactionAsync(CancellationToken cancellationToken);
}
```

**Usage in Command Handlers:**

```csharp
public override async Task<CreateShipperOrderCommand> HandleAsync(
    CreateShipperOrderCommand command, CancellationToken cancellationToken)
{
    await unitOfWork.BeginTransactionAsync(cancellationToken);

    try
    {
        // Multiple repository operations
        shipperAddressRepository.Add(shipperAddress);
        receiverAddressRepository.Add(receiverAddress);
        shipperOrderRepository.Add(shipperOrder);

        await shipperOrderRepository.SaveChangesAsync(cancellationToken);
        await unitOfWork.CommitTransactionAsync(cancellationToken);
    }
    catch (Exception e)
    {
        logger.LogError(e, "Failed to create shipper order");
        await unitOfWork.RollbackTransactionAsync(cancellationToken);
        throw;
    }

    return await base.HandleAsync(command, cancellationToken);
}
```

---

## Command Pattern (Paramore.Brighter)

### Domain Command Base Class

```csharp
using System.Diagnostics;
using Paramore.Brighter;

namespace Pikot.LMP.Domain.Common;

public abstract class DomainCommand : IRequest
{
    protected DomainCommand() { }

    protected DomainCommand(DateTimeOffset timestamp)
    {
        Timestamp = timestamp;
        Span = new Activity(GetType().Name);
        Span.SetStartTime(timestamp.UtcDateTime);
        Span.Start();
    }

    public Guid Id { get; set; } = Guid.NewGuid();
    public Activity Span { get; set; } = null!;
    public DateTimeOffset Timestamp { get; set; }
}
```

### Command Implementation

Commands and their handlers are in the **same file**:

```csharp
using Paramore.Brighter;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Domain.Shipment.ShipperOrders.CreateShipperOrder;

// Command handler
public class CreateShipperOrderCommandHandler(
    IShipperOrderRepository shipperOrderRepository,
    ILogger<CreateShipperOrderCommandHandler> logger,
    IClock clock,
    IUnitOfWork unitOfWork)
    : RequestHandlerAsync<CreateShipperOrderCommand>
{
    public override async Task<CreateShipperOrderCommand> HandleAsync(
        CreateShipperOrderCommand command, CancellationToken cancellationToken = new())
    {
        logger.LogInformation("Creating new shipper order");

        // Validation
        ValidateCommandParams(command);

        await unitOfWork.BeginTransactionAsync(cancellationToken);

        try
        {
            // Business logic
            var shipperOrder = new ShipperOrder(
                command.Timestamp, clock.Now,
                command.ShipperId, /* ... */);

            shipperOrderRepository.Add(shipperOrder);
            await shipperOrderRepository.SaveChangesAsync(cancellationToken);
            await unitOfWork.CommitTransactionAsync(cancellationToken);

            command.CreatedShipperOrderId = shipperOrder.Id;
        }
        catch (Exception e)
        {
            logger.LogError(e, "Failed to create shipper order");
            await unitOfWork.RollbackTransactionAsync(cancellationToken);
            throw;
        }

        logger.LogInformation("New shipper order created with ID {ShipperOrderId}",
            command.CreatedShipperOrderId);
        return await base.HandleAsync(command, cancellationToken);
    }

    private static void ValidateCommandParams(CreateShipperOrderCommand command)
    {
        ValidationException.ThrowIfIsNullOrEmptyOrLongerThan(command.ShipperAddress.Name, 128);
        ValidationException.ThrowIfZeroOrLess(command.ShipperId);
    }
}

// Command definition in same file
public class CreateShipperOrderCommand(DateTimeOffset timestamp) : DomainCommand(timestamp)
{
    public ShipperAddressSubCommand ShipperAddress { get; init; } = new();
    public ReceiverAddressSubCommand ReceiverAddress { get; init; } = new();
    public int ShipperId { get; init; }
    public string? Comment { get; init; }
    public List<decimal> PackageWeightsKg { get; init; } = new();
    public int? CreatedShipperOrderId { get; set; } // Output property

    public class ShipperAddressSubCommand
    {
        public int? Id { get; init; }
        public string Name { get; init; }
        public string Country { get; init; }
        public string City { get; init; }
        // ... other properties
    }
}
```

**Key Patterns:**
- Handler derives from `RequestHandlerAsync<TCommand>`
- Command derives from `DomainCommand`
- Handler injected via primary constructor (C# 12)
- Validation before any business logic
- Use UnitOfWork for transactions
- Set output properties on command object
- Always call `base.HandleAsync()` at the end

---

## Query Pattern

### Query Base Classes

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IQuery<TQueryResult> : IQuery where TQueryResult : IQueryResult;

public interface IQuery
{
    DateTimeOffset Timestamp { get; }
}

// For paginated results
public abstract class PagedQuery<TQueryResult>(
    DateTimeOffset timestamp,
    int pageNumber = 1,
    int pageSize = QueryConstants.DefaultPageSize)
    : IQuery<TQueryResult>
    where TQueryResult : IPagedQueryResult
{
    public DateTimeOffset Timestamp { get; } = timestamp;
    public int PageNumber { get; } = pageNumber;
    public int PageSize { get; } = pageSize;
    public QueryPaging Paging => new(PageNumber, PageSize);
}
```

### Query Handler Implementation

Query handlers are **simple services**, not using Paramore.Brighter:

```csharp
using Microsoft.Extensions.Logging;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Domain.Shipment.ShipperOrders.GetShipperOrder;

public class GetShipperOrderByShipperQueryHandler(
    ILogger<GetShipperOrderByShipperQueryHandler> logger,
    IShipperOrderQueryRepository shipperOrderQueryRepository,
    IShipperUserQueryRepository shipperUserQueryRepository) : IQueryHandler
{
    public async Task<ShipperOrderViewModel?> GetAsync(
        GetShipperOrderByShipperQuery query,
        CancellationToken cancellationToken)
    {
        logger.LogInformation("Getting shipper order {ShipperOrderId} for shipper {ShipperId}",
            query.ShipperOrderId, query.ShipperId);

        var shipperOrder = await shipperOrderQueryRepository.GetAsync(
            query.ShipperOrderId, cancellationToken);

        if (shipperOrder is null)
            return null;

        if (shipperOrder.ShipperId != query.ShipperId)
        {
            logger.LogWarning("Shipper order {ShipperOrderId} not visible to shipper {ShipperId}",
                query.ShipperOrderId, query.ShipperId);
            return null;
        }

        var orderVersion = shipperOrder.GetVersion(query.ShipperOrderVersion);
        if (orderVersion is null)
            return null;

        // Build and return ViewModel
        return new ShipperOrderViewModel(
            shipperOrder.Id,
            orderVersion.Version,
            shipperOrder.Status,
            /* ... */);
    }
}

// Query definition (record for immutability)
public record GetShipperOrderByShipperQuery(
    int ShipperOrderId,
    int ShipperId,
    int? ShipperOrderVersion = null);
```

**Key Patterns:**
- Query handlers implement `IQueryHandler` marker interface
- Simple method with query parameter and cancellation token
- Return `IViewModel` implementations
- Use query repositories (read-only)
- Log the query operation
- Use records for query definitions

---

## ViewModels

```csharp
namespace Pikot.LMP.Domain.Common;

public interface IViewModel : IDomainObject
{
    int Id { get; }
}
```

**Example ViewModel:**

```csharp
public record ShipperOrderViewModel(
    int Id,
    int Version,
    bool IsLatestVersion,
    DateTimeOffset CreatedAt,
    DateTimeOffset InsertedAt,
    int CreatedByShipperUserId,
    string CreatedByShipperUserName,
    string OrderNumber,
    string? Comment,
    ShipperOrderStatusType Status,
    int ShipperId,
    string ShipperCompanyName,
    ShipperAddressViewModel ShipperAddress,
    ReceiverAddressViewModel ReceiverAddress,
    int? DispatcherId,
    string? DispatcherCompanyName,
    ShipperOrderChangeEventViewModel[] Changes,
    int LatestVersion,
    PackageViewModel[] Packages,
    bool IsPickedUp) : IViewModel;
```

**Best Practices:**
- Use records for immutability
- Include all data needed for the view
- Flatten complex object graphs
- Include computed properties from domain logic

---

## Domain Exceptions

### Base Exception

```csharp
namespace Pikot.LMP.Domain.Common.Exceptions;

public abstract class DomainException : Exception
{
    protected DomainException() { }
    protected DomainException(string message) : base(message) { }
}
```

### Common Exceptions

```csharp
// Not found
public class NotFoundException : DomainException
{
    public NotFoundException(string message) : base(message) { }
}

// Invalid state transition
public class InvalidTransitionException : DomainException
{
    public InvalidTransitionException() : base("Invalid state transition") { }
}

// Entity not active
public class NotActiveException : DomainException
{
    public NotActiveException(string message) : base(message) { }
}

// Validation errors
public class ValidationException : DomainException
{
    public string[] Errors { get; }

    public ValidationException(string[] errors) : base(string.Join(", ", errors))
    {
        Errors = errors;
    }

    public static void ThrowIfIsNullOrEmptyOrLongerThan(string? value, int maxLength)
    {
        if (string.IsNullOrWhiteSpace(value) || value.Length > maxLength)
            throw new ValidationException([$"Value must be between 1 and {maxLength} characters"]);
    }

    public static void ThrowIfZeroOrLess(int value)
    {
        if (value <= 0)
            throw new ValidationException(["Value must be greater than zero"]);
    }
}

// Already exists
public class AlreadyExistsException : DomainException
{
    public AlreadyExistsException(string message) : base(message) { }
}

// Versioning conflict
public class InvalidVersionException : DomainException
{
    public InvalidVersionException(string message) : base(message) { }
}

// Unauthorized access
public class UnauthorizedException : DomainException
{
    public UnauthorizedException(string message) : base(message) { }
}
```

---

## Quick Reference Templates

### Creating a New Aggregate

```csharp
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Domain.YourContext;

public class YourAggregate : IAggregate // or IAggregateVersioned
{
    protected YourAggregate() { } // For EF Core

    public YourAggregate(DateTimeOffset createdAt, DateTimeOffset insertedAt, /* params */)
    {
        Id = 0; // Set by database
        CreatedAt = createdAt;
        InsertedAt = insertedAt;

        // Initialize state
    }

    public int Id { get; protected init; }
    public DateTimeOffset CreatedAt { get; }
    public DateTimeOffset InsertedAt { get; }

    // Business properties

    // Business methods
    public void DoSomething(IClock clock, /* params */)
    {
        // Validate state
        // Update state
        // Track domain events if needed
    }
}
```

### Creating a Command & Handler

```csharp
using Microsoft.Extensions.Logging;
using Paramore.Brighter;
using Pikot.LMP.Domain.Common;
using Pikot.LMP.Domain.Common.Exceptions;

namespace Pikot.LMP.Domain.YourContext.Operations;

public class YourOperationCommandHandler(
    IYourRepository repository,
    ILogger<YourOperationCommandHandler> logger,
    IClock clock)
    : RequestHandlerAsync<YourOperationCommand>
{
    public override async Task<YourOperationCommand> HandleAsync(
        YourOperationCommand command, CancellationToken cancellationToken = new())
    {
        logger.LogInformation("Executing operation");

        Validate(command);

        var aggregate = await repository.GetAsync(command.AggregateId, cancellationToken);
        if (aggregate is null)
            throw new NotFoundException("Aggregate not found");

        aggregate.DoSomething(clock, command.SomeParam);

        await repository.SaveChangesAsync(cancellationToken);

        logger.LogInformation("Operation completed");
        return await base.HandleAsync(command, cancellationToken);
    }

    private static void Validate(YourOperationCommand command)
    {
        ValidationException.ThrowIfZeroOrLess(command.AggregateId);
    }
}

public class YourOperationCommand(DateTimeOffset timestamp) : DomainCommand(timestamp)
{
    public int AggregateId { get; init; }
    public string SomeParam { get; init; } = string.Empty;
}
```

### Creating a Query & Handler

```csharp
using Microsoft.Extensions.Logging;
using Pikot.LMP.Domain.Common;

namespace Pikot.LMP.Domain.YourContext.Queries;

public class GetYourAggregateQueryHandler(
    ILogger<GetYourAggregateQueryHandler> logger,
    IYourQueryRepository repository) : IQueryHandler
{
    public async Task<YourAggregateViewModel?> GetAsync(
        GetYourAggregateQuery query,
        CancellationToken cancellationToken)
    {
        logger.LogInformation("Getting aggregate {Id}", query.Id);

        var aggregate = await repository.GetAsync(query.Id, cancellationToken);
        if (aggregate is null)
            return null;

        return new YourAggregateViewModel(
            aggregate.Id,
            aggregate.CreatedAt,
            /* map other properties */);
    }
}

public record GetYourAggregateQuery(int Id);

public record YourAggregateViewModel(
    int Id,
    DateTimeOffset CreatedAt,
    /* other properties */) : IViewModel;
```

---

## Best Practices Summary

1. **Always use IClock** instead of DateTime.Now/UtcNow
2. **Protected parameterless constructor** for EF Core in aggregates and entities
3. **Private setters** on all properties except those initialized in constructor
4. **IReadOnlyCollection** for exposing collections, private List backing field
5. **Validate in command handlers** before calling domain methods
6. **Business logic in aggregates**, not in command handlers
7. **Commands and handlers in the same file**
8. **Query handlers are simple services**, not using Paramore.Brighter
9. **ViewModels for queries**, not aggregates
10. **Use UnitOfWork** when multiple repositories are involved
11. **Log at info level** for successful operations, warning for business errors
12. **Throw domain exceptions** for business rule violations
13. **Records for DTOs** (commands, queries, viewModels)
14. **Primary constructors** in C# 12 for dependency injection
15. **CancellationToken** in all async methods

---

## Architecture Layers

```
┌─────────────────────────────────────────────┐
│           Adapters (Infrastructure)         │
│  - API (WebAPI controllers)                 │
│  - Blazor Apps (Shipper.Web, Dispatcher.Web)│
│  - Data (EF Core implementations)           │
│  - ServiceClients (API DTOs)                │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│             Ports (Domain)                  │
│  - Aggregates & Entities                    │
│  - Command Handlers                         │
│  - Query Handlers                           │
│  - Repository Interfaces                    │
│  - Domain Exceptions                        │
└─────────────────────────────────────────────┘
```

**Dependency Rule**: Adapters depend on Ports, never the reverse.

---

## Common Pitfalls to Avoid

1. Don't use DateTime.Now - use IClock
2. Don't put business logic in command handlers - put it in aggregates
3. Don't expose List<T> directly - use IReadOnlyCollection<T>
4. Don't forget protected parameterless constructor for EF Core
5. Don't make aggregate constructors public without all required data
6. Don't use aggregates in query handlers - use ViewModels
7. Don't skip validation in command handlers
8. Don't forget to call base.HandleAsync() in command handlers
9. Don't use Paramore.Brighter for query handlers
10. Don't write tests in domain project - tests are separate

---

This skill document is based on the LMP project's real implementation and follows hexagonal architecture principles with CQRS and DDD patterns.
