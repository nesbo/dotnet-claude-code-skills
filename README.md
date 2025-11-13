# .NET Claude Code Skills Library

Production-ready Domain-Driven Design patterns and data persistence implementations for .NET projects using Claude Code.

> **Applicability**: While these patterns are explained using C# and .NET, the DDD principles and hexagonal architecture concepts apply to any object-oriented programming language.

> **For Claude Code Users**: See [CLAUDE.md](CLAUDE.md) for information about using this repository with Claude Code.

## Available Skills

### 1. Domain-Driven Design Basics (`ddd-dotnet/`)

**Purpose**: Core DDD patterns for building the domain layer (ports) in hexagonal architecture.

**What it covers:**
- Domain object hierarchy (IDomainObject, IAggregate, IEntity)
- Aggregate root patterns with versioning
- Repository pattern with CQRS separation
- Command pattern using Paramore.Brighter
- Query pattern with ViewModels
- Time control pattern (IClock)
- Unit of Work pattern
- Domain exceptions
- Best practices and anti-patterns

**When to use:**
- Creating new aggregates or entities
- Implementing command handlers
- Implementing query handlers
- Understanding domain layer structure
- Code reviews for domain logic

**Key templates included:**
- New aggregate class
- Command & handler
- Query & handler
- Domain exceptions

---

### 2. Data Persistence Layer (`ef-core-ddd/`)

**Purpose**: Implementing the data persistence layer as an adapter in hexagonal architecture. This skill demonstrates one approach using Entity Framework Core, but the patterns apply to any ORM or data access technology.

**What it covers:**
- Data context configuration (production & testing)
- Entity-to-table mapping and configurations
- Repository implementations (write & query)
- Unit of Work implementation
- Dependency injection auto-registration
- In-memory database testing support
- Schema migration workflow
- Database schema organization
- Performance optimization patterns

**When to use:**
- Adding new aggregates to the database
- Creating entity configurations
- Implementing repositories
- Setting up testing infrastructure
- Configuring relationships and indexes
- Optimizing query performance
- Migration management

**Key templates included:**
- Entity configuration class
- Write repository implementation
- Query repository implementation
- In-memory test setup

---

### 3. BDD-Style Unit Testing (`bdd-dotnet/`)

**Purpose**: Unit testing patterns for domain layer handlers using Behavior-Driven Development style with in-memory database support.

**What it covers:**
- Testing philosophy (hybrid approach with real repositories)
- Test project structure and organization
- TestDataBuilder and TestContextFactory patterns
- Fluent aggregate builders for test data
- Fake implementations instead of mocking frameworks
- Arrange-Act-Assert testing pattern
- NUnit conventions and assertions
- Time-dependent testing with FakeClock
- Parameterized tests with TestCase
- CSV-driven complex scenario testing

**When to use:**
- Writing unit tests for command handlers
- Writing unit tests for query handlers
- Testing workflow state transitions
- Testing validation and error handling
- Setting up test infrastructure
- Creating test data builders for aggregates
- Testing time-dependent business logic

**Key templates included:**
- Test fixture structure
- TestDataBuilder setup
- Aggregate builder classes
- Fake service implementations
- Common test patterns (commands, queries, workflows, errors)

---

### 4. Add Serena MCP to Your Project (`add-serena/`)

**Purpose**: Integration guide for adding Serena MCP (Model Context Protocol) to any software project for IDE-like semantic code understanding.

**What it covers:**
- Serena MCP capabilities and benefits
- Prerequisites checking for your environment
- Global installation using `claude mcp add` command
- Project configuration templates (C#, Python, TypeScript, and more)
- Verification and testing procedures
- Troubleshooting guide
- Advanced configurations (monorepos, read-only mode)
- Common mistakes to avoid

**When to use:**
- Setting up semantic code understanding in your project
- Enabling symbol-level code navigation
- Adding IDE-like capabilities to Claude Code
- Configuring LSP integration for your language
- Troubleshooting Serena installation issues
- Setting up advanced configurations

**Key templates included:**
- Global installation command
- Claude Code settings configuration
- Project-specific YAML configurations
- Language-specific setup examples
- Verification commands

---

## Skill Organization

```
dotnet-claude-code-skills/
├── README.md                    # This file - skills catalog
├── CLAUDE.md                    # Guide for using with Claude Code
├── ddd-dotnet/                  # Domain layer patterns
│   ├── SKILL.md                 # Comprehensive DDD guide
│   └── README.md                # DDD overview
├── ef-core-ddd/                 # Data persistence patterns
│   ├── SKILL.md                 # Comprehensive EF Core guide
│   └── README.md                # EF Core overview
├── bdd-dotnet/                  # BDD-style unit testing patterns
│   ├── SKILL.md                 # Comprehensive testing guide
│   └── README.md                # Testing overview
└── add-serena/                  # Serena MCP integration guide
    ├── SKILL.md                 # Complete installation and configuration guide
    └── README.md                # Serena MCP overview
```

---

## How to Use These Skills

> **Claude Code Users**: For detailed usage instructions with Claude Code, see [CLAUDE.md](CLAUDE.md)

### Quick Reference

**In Code Sessions:**
```
"Using the ddd-dotnet skill, create a new aggregate for Product"
"Following the ef-core-ddd skill, implement the repository for Product"
"Using the bdd-dotnet skill, write unit tests for the CreateProduct command handler"
"Using the add-serena skill, set up Serena MCP for semantic code understanding"
```

**For Code Reviews:**
- Validate new code follows documented patterns
- Identify anti-patterns using "Common Pitfalls" sections
- Ensure consistency across the codebase

**As Templates:**
- Copy templates as starting points for new code
- Use checklists before creating pull requests
- Share patterns with team members

---

## Skill Dependencies

```
ddd-dotnet/
        ▲                     ▲
        │                     │
        │ (implements)        │ (tests)
        │                     │
ef-core-ddd/            bdd-dotnet/
```

- **ef-core-ddd** builds on concepts from **ddd-dotnet** - implements domain interfaces
- **bdd-dotnet** tests handlers and aggregates from **ddd-dotnet** - uses real repositories from **ef-core-ddd**
- Read `ddd-dotnet/SKILL.md` first to understand core domain patterns

---

## Architecture Context

These skills support a **hexagonal architecture** (ports and adapters):

```
┌─────────────────────────────────────────────┐
│           Adapters (Infrastructure)         │
│  - API (WebAPI)                             │
│  - Blazor Apps                              │
│  - Data Persistence ◄── ef-core-ddd        │
│  - ServiceClients                           │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│             Ports (Domain)                  │
│  - Aggregates & Entities  ◄── ddd-dotnet   │
│  - Command Handlers       ◄── ddd-dotnet   │
│  - Query Handlers         ◄── ddd-dotnet   │
│  - Repository Interfaces  ◄── ddd-dotnet   │
└─────────────────────────────────────────────┘
                     ▲
                     │ (tests)
                     │
           ┌─────────────────────┐
           │  Unit Tests         │
           │  (bdd-dotnet)       │
           └─────────────────────┘
```

**Key principle:** Adapters depend on Ports, never the reverse.

---

## Pattern Summary

### Domain Layer (ddd-dotnet/)
- ✅ Aggregates control entities
- ✅ Business logic in aggregates
- ✅ Commands for writes
- ✅ Queries for reads (ViewModels)
- ✅ IClock for time
- ✅ Domain exceptions

### Data Persistence Layer (ef-core-ddd/)
- ✅ Entity-to-storage mapping
- ✅ Repository base classes
- ✅ CQRS separation (write/query)
- ✅ Auto-registration via reflection
- ✅ In-memory testing support
- ✅ Schema organization

### Testing Layer (bdd-dotnet/)
- ✅ BDD-style test naming
- ✅ Real repositories with InMemory DB
- ✅ Fake implementations (no mocking frameworks)
- ✅ Fluent test data builders
- ✅ Arrange-Act-Assert pattern
- ✅ Time control with FakeClock

---

## Quick Start Workflow

### Adding a New Feature with All Skills

1. **Domain First** (using `ddd-dotnet/`):
   ```
   Create aggregate → Create entity → Create command & handler → Create query & handler
   ```

2. **Persistence Second** (using `ef-core-ddd/`):
   ```
   Create entity config → Create write repo → Create query repo → Create migration
   ```

3. **Testing Third** (using `bdd-dotnet/`):
   ```
   Create test fixture → Create aggregate builders → Write handler tests → Run tests
   ```

4. **Verification**:
   - Check domain best practices
   - Check data best practices
   - Run unit tests with in-memory database
   - Review migration SQL
   - Create pull request

---

## Extending the Skills Library

When adding new skills:

1. **Create new `.md` file** in this directory
2. **Update this README** with:
   - Skill name and purpose
   - What it covers
   - When to use
   - Key templates
   - Dependencies on other skills
3. **Follow the existing format**:
   - Overview section
   - Detailed patterns with code examples
   - Best practices section
   - Common pitfalls section
   - Quick reference templates
   - Checklist

---

## Contributing

When updating skills:

- ✅ Keep examples based on real production code
- ✅ Include both good and bad examples
- ✅ Provide working code snippets
- ✅ Add explanatory comments
- ✅ Update this README if adding new skills
- ✅ Keep skills focused and cohesive
- ✅ Cross-reference related skills

---

## Skill Maintenance

These skills are living documents. Update them when:

- Discovering new patterns worth documenting
- Finding common mistakes to warn against
- Refactoring improves existing patterns
- Technology updates change best practices
- Team feedback suggests improvements

---

## About This Repository

**Purpose**: Skills library for object-oriented development with Claude Code
**Focus**: Domain-Driven Design, CQRS, Hexagonal Architecture
**Implementation**: Demonstrated in .NET 8+ and C# 12
**Applicability**: Patterns apply to any OOP language (Java, TypeScript, Python, etc.)
**Usage**: Reference patterns, code templates, best practices

> These skills are explained using C# and .NET, but the DDD and hexagonal architecture patterns apply universally to object-oriented programming.

---

**Last Updated**: 2025-10-22
**Skills Version**: 1.0
