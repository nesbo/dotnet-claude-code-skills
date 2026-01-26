# .NET Unit Testing Skill

Use this skill when writing unit tests for the Domain.Shared project. This skill captures the unique testing patterns, conventions, and philosophy used in this codebase.

## Testing Philosophy

### Core Principles
- **Ports Testing Only**: Unit tests focus exclusively on testing domain ports (handlers). Each handler represents one user story or use case.
- **Hybrid Testing Approach**: We use real repository implementations with EF Core InMemory database instead of mocks. This "shifts left" to catch issues earlier with more coverage for less effort.
- **No Mock Libraries**: Instead of using mocking frameworks, we create fake implementations for dependencies with realistic behavior.
- **Realistic Behaviors**: Tests cover realistic use cases and behaviors, not artificial scenarios.
- **TDD Support**: The pattern supports Test-Driven Development - start from the test and model, then write ports/handlers and even entity data configurations from tests.

### What to Test
- Handler behavior (command handlers and query handlers)
- Business logic in domain aggregates
- Repository interactions through real implementations
- Workflow and state transitions
- Validation and error handling
- Edge cases and boundary conditions

### What NOT to Test
- Infrastructure implementation details
- Database queries directly
- Third-party library behavior
- Framework features

## Project Structure

### Test Project Configuration
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <!-- NUnit Testing Framework -->
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.1.0"/>
    <PackageReference Include="NUnit" Version="3.13.3"/>
    <PackageReference Include="NUnit3TestAdapter" Version="4.2.1"/>
    <PackageReference Include="NUnit.Analyzers" Version="3.3.0"/>

    <!-- InMemory Database for Testing -->
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="6.0.32" />
  </ItemGroup>
</Project>
```

### Directory Structure
```
RMS.Domain.Tests/
├── Builders/
│   ├── TestContextFactory.cs      # Creates test database context
│   ├── TestDataBuilder.cs         # Main test data builder with repositories
│   ├── ProductBuilder.cs          # Fluent builder for Product aggregate
│   ├── FamilyBuilder.cs     # Fluent builder for Family aggregate
│   └── [Other aggregate builders]
├── Fakes/
│   ├── FakeClock.cs               # Fake time service
│   ├── FakeUnitOfWork.cs          # Fake transaction coordinator
│   ├── FakeBlobStorageService.cs  # Fake blob storage
│   └── [Other fake services]
├── Product/
│   ├── SetFamilyToProductCommandHandlerTests.cs
│   └── [Other product handler tests]
├── Competitive/
│   └── [Competitive agreement handler tests]
└── TestData/
    └── [CSV files for test data]
```

## Core Testing Components

### 1. TestContextFactory

Creates an EF Core InMemory database context for testing.

```csharp
public class TestContextFactory
{
    private readonly DbContextType _contextType;
    private readonly string _inMemoryDatabaseName;

    public TestContextFactory(
        DbContextType contextType = DbContextType.InMemory,
        string? inMemoryDatabaseName = null)
    {
        _contextType = contextType;
        _inMemoryDatabaseName = inMemoryDatabaseName ?? Guid.NewGuid().ToString();
    }

    public MyContext Context => CreateContext(_contextType, _inMemoryDatabaseName);
}
```

**Key Points:**
- Each test gets a unique in-memory database by default
- Can optionally test against LocalSqlServer for integration scenarios
- Database is created automatically via `EnsureCreated()`

### 2. TestDataBuilder

The main orchestrator for test setup. It:
- Creates and holds real repository implementations
- Manages the DbContext lifecycle
- Provides fluent methods to add test data
- Registers services in a ServiceCollection

**Example from SetFamilyToProductCommandHandlerTests.cs:**
```csharp
[SetUp]
public void SetUp()
{
    _clock = new FakeClock();
    _testDataBuilder = new TestDataBuilder();

    _handler = new SetFamilyToProductCommandHandler(
        _clock,
        _testDataBuilder.ProductRepository,  // Real repository!
        new NullLogger<SetFamilyToProductCommandHandler>(),
        _testDataBuilder.FamilyRepository,
        new FakeUnitOfWork());
}

[Test]
public async Task GivenSetProductToFamilyRequest_WhenHandled_ThenFamilyAndProductAreUpdated()
{
    // arrange
    var Family = new FamilyBuilder().Build();
    var productNo = "1234";
    var product = new ProductBuilder(productNo, "5678").Build();

    _testDataBuilder
        .WithFamily(Family)
        .WithProduct(product);

    var command = new SetFamilyToProductCommand
    {
        ProductNo = productNo,
        FamilyId = Family.Id,
        Type = SetFamilyToProductType.SetFamilyCode,
        UserId = "userId"
    };

    // act
    var result = await _handler.HandleAsync(command, CancellationToken.None);

    // assert
    Assert.That(result.Success, Is.True, "Command should be successful");

    var updatedProduct = await _testDataBuilder.ProductQueryRepository
        .GetAsync(productNo, CancellationToken.None);
    Assert.IsNotNull(updatedProduct, "Product should be found");
    Assert.That(updatedProduct.FamilyId, Is.EqualTo(Family.Id));
}
```

**TestDataBuilder Key Methods:**
```csharp
// Setup methods (fluent API)
.WithProduct(product)
.WithFamily(Family)
.WithAuthorizedProduct(authorizedProduct)
.WithDistributorSite(distributorSite)
.WithImportProductRequest(request)
.WithCompetitiveAgreement(agreement)

// Retrieval methods
.GetAll<T>()          // Get all entities of type T
.CountAll<T>()        // Count entities of type T

// Repository access
.ProductRepository
.ProductQueryRepository
.FamilyRepository
.CompetitiveAgreementRepository
// ... many more
```

### 3. Aggregate Builders (Fluent API)

Builders use the fluent pattern to construct domain aggregates with test data.

**ProductBuilder Example:**
```csharp
public class ProductBuilder
{
    private readonly string _productNo;
    private readonly string? _productClassNo;
    private decimal? _listPrice;
    private DateTime? _validFrom;
    private string? _scaleType;

    public ProductBuilder(string productNo, string productClassNo)
    {
        _productNo = productNo;
        _productClassNo = productClassNo;
    }

    public ProductBuilder WithListPrice(decimal listPrice)
    {
        _listPrice = listPrice;
        return this;
    }

    public ProductBuilder WithValidFrom(DateTime validFrom)
    {
        _validFrom = validFrom;
        return this;
    }

    public ProductBuilder WithScaleType(string scaleType)
    {
        _scaleType = scaleType;
        return this;
    }

    public Product Build()
    {
        var clock = new FakeClock();
        return new Product(
            _productNo,
            _listPrice,
            _validFrom ?? clock.UtcNow(),
            _productFamilyNo ?? "fam",
            null,
            _scaleType ?? "MOTORS",
            _productClassNo,
            // ... other parameters
        );
    }
}

// Usage in tests:
var product = new ProductBuilder("12345", "classNo")
    .WithListPrice(100.50m)
    .WithValidFrom(new DateTime(2024, 1, 1))
    .WithScaleType("INDUSTRIAL")
    .Build();
```


### 4. Fake Implementations

Create simple, controllable fake implementations instead of using mocking frameworks.

**FakeClock (Control Time):**
```csharp
public class FakeClock : IClock
{
    private DateTime _utcNow;

    public FakeClock(DateTime? utcNow = null)
    {
        _utcNow = utcNow ?? DateTime.UtcNow;
    }

    public DateTime UtcNow() => _utcNow;

    public DateTimeOffset ZeroOffsetUtcNow() => _utcNow;

    public void OverrideUtcNow(DateTime utcNow)
    {
        _utcNow = utcNow;
    }
}

// Usage:
var clock = new FakeClock(new DateTime(2024, 1, 1));
clock.OverrideUtcNow(new DateTime(2024, 6, 1)); // Simulate time passing
```

**FakeUnitOfWork (No-op transactions):**
```csharp
public class FakeUnitOfWork : IUnitOfWork
{
    public Task BeginTransactionAsync(CancellationToken cancellationToken)
        => Task.CompletedTask;

    public Task CommitTransactionAsync(CancellationToken cancellationToken)
        => Task.CompletedTask;

    public Task RollbackTransactionAsync(CancellationToken cancellationToken)
        => Task.CompletedTask;

    public void Dispose() { }
}
```

**FakeBlobStorageService (Arrange return values):**
```csharp
public class FakeBlobStorageService : IBlobStorageService
{
    private Stream? _streamToReturn;

    public void SetStreamToReturn(Stream stream)
    {
        _streamToReturn = stream;
    }

    public Task<Stream> GetDownloadStreamAsync(
        string blobName,
        string containerName,
        CancellationToken cancellationToken)
    {
        if (_streamToReturn == null)
            throw new InvalidOperationException("Stream to return is not set");

        return Task.FromResult(_streamToReturn);
    }

    // Other methods throw NotImplementedException or return defaults
}
```

// In test:
Assert.That(_notifyService.IsSendAsyncCalled, Is.True);

## Test Structure and Conventions

### NUnit Test Class Structure

```csharp
[TestFixture]
public class SetFamilyToProductCommandHandlerTests
{
    // System Under Test
    private SetFamilyToProductCommandHandler _handler;

    // Dependencies
    private FakeClock _clock;
    private TestDataBuilder _testDataBuilder;

    [SetUp]
    public void SetUp()
    {
        // Initialize fresh for each test
        _clock = new FakeClock();
        _testDataBuilder = new TestDataBuilder();

        _handler = new SetFamilyToProductCommandHandler(
            _clock,
            _testDataBuilder.ProductRepository,
            new NullLogger<SetFamilyToProductCommandHandler>(),
            _testDataBuilder.FamilyRepository,
            new FakeUnitOfWork()
        );
    }

    [Test]
    public async Task GivenCondition_WhenAction_ThenExpectedOutcome()
    {
        // arrange
        // ... setup test data

        // act
        // ... execute the handler

        // assert
        // ... verify results
    }

    [TestCase("value1", ExpectedResult)]
    [TestCase("value2", ExpectedResult)]
    public async Task ParameterizedTest(string input, object expected)
    {
        // ...
    }
}
```

### Naming Conventions

**Test Method Names:**
Use the pattern: `Given[InitialCondition]_When[Action]_Then[ExpectedOutcome]`

Examples:
- `GivenSetProductToFamilyRequest_WhenHandled_ThenFamilyAndProductAreUpdated`
- `GivenRunningWorkflowExists_WhenHandled_ReturnsError`
- `GivenExistingListPrice_WhenListPriceIsChangedForExistingPeriod_ThenPreviousListPriceIsExpiredAndNewInserted`

**Variable Names:**
- Be descriptive and clear
- Use full words, not abbreviations
- Match domain language

### Arrange-Act-Assert Pattern

Always use clear sections with comments:

```csharp
[Test]
public async Task GivenExistingProduct_WhenUpdated_ThenChangesPersisted()
{
    // Arrange
    var productNo = "12345";
    var product = new ProductBuilder(productNo, "classNo")
        .WithListPrice(100m)
        .Build();

    _testDataBuilder.WithProduct(product);

    var command = new UpdateProductCommand { ProductNo = productNo, NewPrice = 150m };

    // Act
    var result = await _handler.HandleAsync(command, CancellationToken.None);

    // Assert
    Assert.That(result.Success, Is.True, "Command should succeed");

    var updatedProduct = await _testDataBuilder.ProductQueryRepository
        .GetAsync(productNo, CancellationToken.None);
    Assert.That(updatedProduct.ListPrice, Is.EqualTo(150m), "Price should be updated");
}
```

### NUnit Assertions

**Common Assertion Patterns:**

```csharp
// Success/Failure
Assert.That(result.Success, Is.True, "Reason for expectation");
Assert.That(result.Success, Is.False, "Should fail when...");

// Null checks
Assert.IsNotNull(entity, "Entity should be found");
Assert.IsNull(entity, "Entity should not exist");

// Equality
Assert.That(actual, Is.EqualTo(expected), "Description");

// Collections
Assert.That(collection, Has.Length.EqualTo(5));
Assert.That(collection, Has.Count.EqualTo(5));
Assert.That(collection, Is.Empty);
Assert.That(collection, Has.All.Property("Status").EqualTo("Active"));

// Numeric comparisons
Assert.That(value, Is.GreaterThan(10));
Assert.That(value, Is.LessThanOrEqualTo(100));

// Multiple assertions
Assert.Multiple(() =>
{
    Assert.That(result.Success, Is.True, "Should succeed");
    Assert.That(result.Message, Is.EqualTo("Expected message"));
    Assert.That(_service.IsCalled, Is.True, "Service should be called");
});
```

### TestCase for Parameterized Tests

```csharp
[TestCase("2024-05-01", null, "2024-05-01", null, "2024-05-01", null)]
[TestCase("2024-05-01", "2025-05-31", "2024-08-01", null, "2024-05-01", null)]
public async Task GivenExistingPriceHistory_WhenNewDatesAreSet_ThenValidToDateIsUpdated(
    string currentValidFromText,
    string? currentValidToText,
    string? newValidFromText,
    string? newValidToText,
    string expectedValidFromText,
    string? expectedValidToText)
{
    var validFrom = ParseDate(currentValidFromText);
    var validTo = currentValidToText is null ? null : ParseDate(currentValidToText);
    // ... test logic
}
```

## Common Testing Patterns

### Pattern 1: Testing Command Handlers

```csharp
[Test]
public async Task GivenValidCommand_WhenHandled_ThenEntityUpdatedAndResultSuccess()
{
    // Arrange
    var entity = new EntityBuilder()
        .WithProperty("value")
        .Build();

    _testDataBuilder.WithEntity(entity);

    var command = new UpdateEntityCommand
    {
        Id = entity.Id,
        NewValue = "updated"
    };

    // Act
    var result = await _handler.HandleAsync(command, CancellationToken.None);

    // Assert
    Assert.That(result.Success, Is.True);

    var updated = await _testDataBuilder.EntityQueryRepository
        .GetAsync(entity.Id, CancellationToken.None);
    Assert.That(updated.Property, Is.EqualTo("updated"));
}
```

### Pattern 2: Testing Query Handlers

```csharp
[Test]
public async Task GivenEntitiesExist_WhenQueried_ThenReturnsFilteredResults()
{
    // Arrange
    var entity1 = new EntityBuilder().WithStatus("Active").Build();
    var entity2 = new EntityBuilder().WithStatus("Inactive").Build();

    _testDataBuilder
        .WithEntity(entity1)
        .WithEntity(entity2);

    var query = new ListEntitiesQuery { Status = "Active" };

    // Act
    var result = await _handler.HandleAsync(query, CancellationToken.None);

    // Assert
    Assert.That(result.Items, Has.Count.EqualTo(1));
    Assert.That(result.Items.First().Id, Is.EqualTo(entity1.Id));
}
```

### Pattern 3: Testing Workflow State Transitions

```csharp
[Test]
public async Task GivenWorkflowInProgress_WhenCompleted_ThenStatusUpdatedToSucceeded()
{
    // Arrange
    var workflow = new WorkflowBuilder()
        .WithStatus(WorkflowStatus.InProgress)
        .Build();

    _testDataBuilder.WithWorkflow(workflow);

    var command = new CompleteWorkflowCommand { WorkflowId = workflow.Id };

    // Act
    await _handler.HandleAsync(command, CancellationToken.None);

    // Assert
    var updated = _testDataBuilder.GetAll<Workflow>().Single();
    Assert.That(updated.Status, Is.EqualTo(WorkflowStatus.Succeeded));
    Assert.IsNotNull(updated.CompletedAt);
}
```

### Pattern 4: Testing Error Conditions

```csharp
[Test]
public async Task GivenRunningWorkflowExists_WhenNewWorkflowInitialized_ThenReturnsError()
{
    // Arrange
    var existingWorkflow = new WorkflowBuilder()
        .WithStatus(WorkflowStatus.InProgress)
        .Build();

    _testDataBuilder.WithWorkflow(existingWorkflow);

    var command = new InitializeWorkflowCommand();

    // Act
    var result = await _handler.HandleAsync(command, CancellationToken.None);

    // Assert
    Assert.That(result.Success, Is.False, "Should fail when workflow exists");
    Assert.That(result.Message,
        Is.EqualTo("An active workflow already exists"));
}
```

### Pattern 5: Testing with Time-Dependent Logic

```csharp
[Test]
public async Task GivenOutdatedRequest_WhenHandled_ThenRequestIsNotApplied()
{
    // Arrange
    var validFrom = new DateTime(2024, 5, 1, 0, 0, 0, DateTimeKind.Utc);
    var product = new ProductBuilder(productNo, "classNo")
        .WithValidFrom(validFrom)
        .Build();

    _testDataBuilder.WithProduct(product);

    // Set clock to future date (request is outdated)
    _testClock.OverrideUtcNow(validFrom.AddMonths(3));

    var command = new UpdateCommand { ValidFrom = validFrom };

    // Act
    var result = await _handler.HandleAsync(command, CancellationToken.None);

    // Assert
    Assert.That(result.Success, Is.False);
    var request = _testDataBuilder.GetAll<Request>().Single();
    Assert.IsNotNull(request.ErrorMessage, "Should have error message");
}
```

### Pattern 6: Testing Complex Data Scenarios with CSV

```csharp
[Test]
public async Task GivenComplexScenarios_WhenProcessed_ThenAllHandledCorrectly()
{
    // Load test cases from CSV
    var testCases = TestDataReader.ReadCsv<TestScenarioRow>("test_scenarios.csv");

    var testCaseNo = 1;
    foreach (var testCase in testCases)
    {
        // Reinitialize for each case
        SetUp();

        // Arrange from test case data
        var entity = new EntityBuilder()
            .WithValue(testCase.InputValue)
            .Build();
        _testDataBuilder.WithEntity(entity);

        // Act
        await _handler.HandleAsync(command, CancellationToken.None);

        // Assert with context
        var result = _testDataBuilder.GetAll<Entity>().Single();
        Assert.That(result.Value, Is.EqualTo(testCase.ExpectedValue),
            $"Test case {testCaseNo} failed");

        testCaseNo++;
    }
}
```

## Testing Handler with Multiple Dependencies

Example from InitializeImportCompetitiveAgreementsCommandHandlerTests.cs:

```csharp
[TestFixture]
public class InitializeImportCompetitiveAgreementsCommandHandlerTests
{
    private InitializeImportCompetitiveAgreementsCommandHandler _handler;
    private TestDataBuilder _testDataBuilder;
    private FakeCommandSender _commandSender;
    private FakeBlobStorageService _fakeBlobStorageService;
    private FakeNotifyService _fakeNotifyService;
    private FakeDistributedLockService _fakeDistributedLockService;

    [SetUp]
    public void SetUp()
    {
        _testDataBuilder = new TestDataBuilder();
        _commandSender = new FakeCommandSender();
        _fakeBlobStorageService = new FakeBlobStorageService();
        _fakeNotifyService = new FakeNotifyService();
        _fakeDistributedLockService = new FakeDistributedLockService();

        _handler = new InitializeImportCompetitiveAgreementsCommandHandler(
            new FakeEventSender(),
            new NullLoggerFactory(),
            new FakeClock(),
            _commandSender,
            _fakeBlobStorageService,
            _testDataBuilder.ImportCompetitiveAgreementsRepository,
            _fakeNotifyService,
            new FakeUnitOfWork(),
            _fakeDistributedLockService
        );
    }

    [Test]
    public async Task GivenRunningWorkflowExists_WhenHandled_ReturnsError()
    {
        // Arrange
        var command = new InitializeImportCompetitiveAgreementsCommand(
            "test-file.csv", "user-id", DateTimeOffset.UtcNow, "original-file.csv");
        var message = new FakeCommandMessage<InitializeImportCompetitiveAgreementsCommand>(command);

        var existingWorkflow = new ImportCompetitiveAgreementWorkflow(
            "test-file.csv", "original-file.csv", "user-id",
            _fakeClock.ZeroOffsetUtcNow(), _fakeClock.ZeroOffsetUtcNow(), 0);
        _testDataBuilder.WithImportCompetitiveAgreementWorkflow(existingWorkflow);

        // Act
        var result = await _handler.ExternalHandleAsync(message, CancellationToken.None);

        // Assert
        Assert.Multiple(() =>
        {
            Assert.That(result.Success, Is.False, "Expected command to fail");
            Assert.That(result.Message,
                Is.EqualTo("An active import competitive agreements workflow already exists"));
            Assert.That(_fakeDistributedLockService
                .IsReleaseCalled(InitializeImportCompetitiveAgreementsCommand.LockKey),
                Is.True, "Expected lock to be released");
        });
    }
}
```

## Best Practices

### 1. Test Data Setup
- Use builders for complex domain objects
- Keep test data minimal but realistic
- Use meaningful values that make tests readable
- Default to sensible values in builders

### 2. Test Independence
- Each test should be completely independent
- Use `[SetUp]` to create fresh instances
- Don't rely on test execution order
- Each test gets its own in-memory database

### 3. Assertion Messages
- Always include descriptive assertion messages
- Messages should explain WHY the assertion should pass
- Example: `Assert.That(result.Success, Is.True, "Command should succeed when workflow is valid")`

### 4. Test Readability
- Use clear variable names
- Add comments for complex setup
- Keep arrange/act/assert sections distinct
- One logical assertion per test (or use Assert.Multiple)

### 5. Error Testing
- Test both success and failure paths
- Verify error messages are meaningful
- Test validation rules
- Test boundary conditions

### 6. Testing Realistic Scenarios
- Focus on use cases from user stories
- Test workflows end-to-end through handlers
- Verify state changes in the database
- Test side effects (events, notifications, etc.)

### 7. Avoid Over-Testing
- Don't test framework features
- Don't test third-party libraries
- Don't test property getters/setters
- Focus on business logic and behavior

## Quick Reference: Writing a New Test

```csharp
[TestFixture]
public class MyNewCommandHandlerTests
{
    private MyNewCommandHandler _handler;
    private TestDataBuilder _testDataBuilder;
    private FakeClock _clock;
    // ... other fakes as needed

    [SetUp]
    public void SetUp()
    {
        _clock = new FakeClock();
        _testDataBuilder = new TestDataBuilder();

        _handler = new MyNewCommandHandler(
            _testDataBuilder.MyRepository,
            _clock,
            new FakeUnitOfWork()
            // ... other dependencies
        );
    }

    [Test]
    public async Task GivenCondition_WhenAction_ThenExpectedResult()
    {
        // Arrange
        var entity = new EntityBuilder()
            .WithProperty(value)
            .Build();

        _testDataBuilder.WithEntity(entity);

        var command = new MyCommand { /* properties */ };

        // Act
        var result = await _handler.HandleAsync(command, CancellationToken.None);

        // Assert
        Assert.That(result.Success, Is.True, "Reason");

        var updated = await _testDataBuilder.EntityQueryRepository
            .GetAsync(entity.Id, CancellationToken.None);
        Assert.That(updated.Property, Is.EqualTo(expectedValue), "Reason");
    }
}
```

## Common Gotchas and Solutions

### Problem: "DbContext has been disposed"
**Solution:** Don't hold references to entities across multiple SaveChanges calls. Reload from repository.
**Solution** Don't call scoped DBContext concurrently. It is not thread safe.

### Problem: Test passes in isolation but fails when run with others
**Solution:** Ensure each test has its own TestDataBuilder instance in SetUp. Check for static state.

### Problem: InMemory database doesn't enforce all constraints
**Solution:** This is expected. Focus on testing business logic, not database constraints.

### Problem: DateTime comparison failures
**Solution:** Be explicit about DateTime.Kind. Use FakeClock. Compare only Date when time doesn't matter.

### Problem: Async test hangs
**Solution:** Always use `CancellationToken.None` or a real token. Don't use `.Result` or `.Wait()`.

## Summary

This testing approach provides:
- **Fast execution** (in-memory database)
- **High confidence** (real repository implementations)
- **Maintainability** (no brittle mocks)
- **Readability** (fluent builders, clear patterns)
- **TDD support** (write tests first, implement after)

When writing tests, remember:
1. Each handler = one use case
2. Use real repositories, fake services
3. Build test data with builders
4. Arrange-Act-Assert structure
5. Descriptive names and assertions
6. Focus on behavior, not implementation
