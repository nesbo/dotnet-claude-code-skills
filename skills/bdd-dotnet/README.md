# BDD-Style Unit Testing for .NET Domain Layer

**Purpose**: Unit testing patterns for domain layer handlers using Behavior-Driven Development (BDD) style with NUnit and EF Core InMemory database.

## Overview

This skill provides comprehensive patterns for writing unit tests that focus on domain behavior rather than implementation details. It uses a hybrid testing approach with real repository implementations backed by EF Core InMemory database, combined with fake implementations for other dependencies.

## Key Concepts

### Testing Philosophy

- **Ports Testing Only**: Tests focus on handlers (command and query handlers) which represent use cases
- **Hybrid Approach**: Uses real repository implementations with InMemory database instead of mocks
- **No Mock Libraries**: Fake implementations with realistic behavior instead of mocking frameworks
- **BDD-Style Naming**: Test names follow `Given[Condition]_When[Action]_Then[Outcome]` pattern
- **TDD Support**: Write tests first, then implement domain logic and configurations

### Core Components

1. **TestDataBuilder**: Orchestrates test setup with real repositories and fluent data methods
2. **TestContextFactory**: Creates EF Core InMemory database contexts
3. **Aggregate Builders**: Fluent builders for constructing test domain objects
4. **Fake Implementations**: Simple, controllable fakes (FakeClock, FakeUnitOfWork, etc.)

## What This Skill Covers

- ✅ Test project structure and configuration
- ✅ TestDataBuilder pattern for managing test data
- ✅ Fluent aggregate builders
- ✅ Fake service implementations
- ✅ Arrange-Act-Assert test structure
- ✅ NUnit conventions and assertions
- ✅ Testing command handlers
- ✅ Testing query handlers
- ✅ Testing workflows and state transitions
- ✅ Time-dependent testing with FakeClock
- ✅ Parameterized tests with TestCase
- ✅ CSV-driven complex scenario testing

## When to Use This Skill

Use this skill when:

- Writing unit tests for command handlers
- Writing unit tests for query handlers
- Testing business logic in aggregates
- Testing workflow state transitions
- Testing validation and error handling
- Setting up test infrastructure for a new project
- Creating test data builders for domain aggregates
- Testing time-dependent business logic
- Implementing Test-Driven Development (TDD)

## Quick Example

```csharp
[TestFixture]
public class CreateProductCommandHandlerTests
{
    private CreateProductCommandHandler _handler;
    private TestDataBuilder _testDataBuilder;
    private FakeClock _clock;

    [SetUp]
    public void SetUp()
    {
        _clock = new FakeClock();
        _testDataBuilder = new TestDataBuilder();

        _handler = new CreateProductCommandHandler(
            _testDataBuilder.ProductRepository,
            _clock,
            new FakeUnitOfWork()
        );
    }

    [Test]
    public async Task GivenValidCommand_WhenHandled_ThenProductIsCreated()
    {
        // Arrange
        var command = new CreateProductCommand
        {
            ProductNo = "12345",
            Name = "Test Product"
        };

        // Act
        var result = await _handler.HandleAsync(command, CancellationToken.None);

        // Assert
        Assert.That(result.Success, Is.True, "Command should succeed");

        var product = await _testDataBuilder.ProductQueryRepository
            .GetAsync("12345", CancellationToken.None);
        Assert.IsNotNull(product, "Product should be created");
        Assert.That(product.Name, Is.EqualTo("Test Product"));
    }
}
```

## Technology Stack

- **Testing Framework**: NUnit 3.x
- **Database**: EF Core InMemory provider
- **Pattern**: Arrange-Act-Assert
- **Style**: BDD naming conventions
- **.NET Version**: .NET 6+

## Related Skills

- **ddd-dotnet/**: Defines the domain handlers and aggregates being tested
- **data-dotnet/**: Provides repository implementations used in tests

## Structure

```
bdd-dotnet/
├── README.md              # This file - overview
└── SKILL.md               # Comprehensive testing guide with patterns and examples
```

## Key Patterns Included

### Test Patterns
- Command handler testing
- Query handler testing
- Workflow state transition testing
- Error condition testing
- Time-dependent testing
- Complex scenario testing with CSV data

### Infrastructure Patterns
- TestDataBuilder setup
- Aggregate builder creation
- Fake implementation creation
- Test fixture structure
- Parameterized test cases

## Getting Started

1. **Read the full guide**: See [SKILL.md](SKILL.md) for comprehensive patterns and examples
2. **Set up test project**: Configure NUnit and EF Core InMemory packages
3. **Create TestContextFactory**: Set up in-memory database creation
4. **Create TestDataBuilder**: Implement the main test orchestrator
5. **Build aggregate builders**: Create fluent builders for your domain objects
6. **Write fake implementations**: Create simple fakes for dependencies
7. **Write tests**: Follow the patterns and conventions in SKILL.md

## Best Practices Summary

✅ **DO**:
- Test handlers (use cases), not repository implementations
- Use real repositories with InMemory database
- Create fluent builders for complex aggregates
- Use FakeClock for time-dependent tests
- Follow Given-When-Then naming convention
- Include descriptive assertion messages
- Make tests independent with fresh setup per test

❌ **DON'T**:
- Use mocking frameworks - create simple fakes instead
- Test infrastructure implementation details
- Share state between tests
- Test framework or third-party library behavior
- Skip assertion messages

## Example Prompts for Claude Code

```
"Using the bdd-dotnet skill, write unit tests for the CreateProduct command handler"

"Following the bdd-dotnet patterns, create a test data builder for the Order aggregate"

"Using the bdd-dotnet skill, write tests for the workflow state transitions in the ImportWorkflow"

"Review my unit tests against the bdd-dotnet best practices"
```

---

**For full documentation**, see [SKILL.md](SKILL.md)

**Last Updated**: 2025-10-23
