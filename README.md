# Claude Code - Complete .NET Engineering Team

**Your AI-powered engineering team for building production-ready .NET applications with Domain-Driven Design, CQRS, and Hexagonal Architecture.**

This repository provides specialized AI agents and comprehensive skill documentation that work together like a complete engineering team. Each agent is an expert in their domain, following established patterns and best practices.

> **Universal Applicability**: While demonstrated with C# and .NET, the DDD principles and hexagonal architecture patterns apply to any object-oriented programming language.

> **Quick Start**: See [CLAUDE.md](CLAUDE.md) for installation and usage with Claude Code.

## Your Engineering Team

### Specialized Agents (`dotnet-agents/`)

Four expert agents that form your complete .NET engineering team:

**🟢 Domain Engineer** (`ddd-domain-engineer`)
- Implements domain logic, aggregates, and business rules
- Creates command and query handlers following CQRS patterns
- Ensures hexagonal architecture compliance
- **Model**: Sonnet (fast, high-quality implementation)

**🔴 QA Specialist** (`bdd-qa-specialist`)
- Writes comprehensive BDD-style unit tests
- Plans test scenarios before implementation (shift-left approach)
- Creates fake implementations instead of mocks
- **Model**: Sonnet (efficient test generation)

**🟡 Data Engineer** (`data-layer-engineer`)
- Implements repositories and entity configurations
- Optimizes queries and prevents N+1 problems
- Enforces strict aggregate boundary rules
- **Model**: Sonnet (focused data layer work)

**🟣 Software Architect** (`hexagonal-architecture-modeler`)
- Provides architectural guidance and design reviews
- Ensures proper separation of concerns
- Applies DDD and SOLID principles
- **Model**: Opus (highest quality for architectural decisions)

**How the team works together:**
1. **Architect** designs the feature architecture
2. **Domain Engineer** implements domain logic
3. **Data Engineer** implements persistence layer
4. **QA Specialist** writes comprehensive tests

**Read more**: [dotnet-agents/README.md](dotnet-agents/README.md)

---

## Knowledge Base (Skills)

The engineering team references these comprehensive skill guides to ensure consistent, production-ready implementations. Each skill provides patterns, templates, and best practices that agents automatically apply.

### 1. Domain-Driven Design (`skills/ddd-dotnet/`)

**Purpose**: Core DDD patterns for building the domain layer (ports) in hexagonal architecture.
**Used by**: Domain Engineer, Software Architect

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

### 2. Data Persistence (`skills/data-dotnet/`)

**Purpose**: Implementing the data persistence layer as an adapter in hexagonal architecture. This skill demonstrates one approach using Entity Framework Core, but the patterns apply to any ORM or data access technology.
**Used by**: Data Engineer, Domain Engineer

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

### 3. BDD-Style Unit Testing (`skills/bdd-dotnet/`)

**Purpose**: Unit testing patterns for domain layer handlers using Behavior-Driven Development style with in-memory database support.
**Used by**: QA Specialist

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

## Repository Organization

```
dotnet-claude-code-skills/
├── README.md                    # This file - engineering team overview
├── CLAUDE.md                    # Installation and usage guide
├── dotnet-agents/               # Your engineering team (4 specialized agents)
│   ├── agents/
│   │   ├── ddd-domain-engineer.md          # 🟢 Domain Engineer
│   │   ├── bdd-qa-specialist.md            # 🔴 QA Specialist
│   │   ├── data-layer-engineer.md          # 🟡 Data Engineer
│   │   └── hexagonal-architecture-modeler.md # 🟣 Software Architect
│   ├── commands/                # Future: Quick commands
│   └── README.md                # Agents documentation
└── skills/                      # Knowledge base (referenced by agents)
    ├── ddd-dotnet/              # Domain-Driven Design patterns
    │   ├── SKILL.md             # Comprehensive DDD guide
    │   └── README.md            # DDD overview
    ├── data-dotnet/             # Data persistence patterns
    │   ├── SKILL.md             # Data layer guide
    │   └── README.md            # Data overview
    └── bdd-dotnet/              # BDD testing patterns
        ├── SKILL.md             # Testing guide
        └── README.md            # Testing overview
```

---

## How to Use Your Engineering Team

> **Full Guide**: See [CLAUDE.md](CLAUDE.md) for detailed usage instructions

### Installation

Install your complete engineering team in Claude Code:

```bash
/plugin marketplace add https://github.com/nesbo/dotnet-claude-code-skills
```

This installs:
- **4 specialized agents** (your engineering team)
- **3 comprehensive skills** (knowledge base for the team)

### Quick Start Examples

**Ask agents to work on features:**
```
"Use the ddd-domain-engineer agent to create a new Order aggregate"
"Use the data-layer-engineer agent to implement repositories for Product"
"Use the bdd-qa-specialist agent to write tests for CreateProduct handler"
"Use the hexagonal-architecture-modeler agent to review my architecture"
```

**Complete feature workflow:**
```
"Use the hexagonal-architecture-modeler to design a payment feature,
then the ddd-domain-engineer to implement domain logic,
the data-layer-engineer for persistence,
and the bdd-qa-specialist to write tests"
```

**Reference skills directly** (for manual implementation):
```
"Following the ddd-dotnet skill, create an Order aggregate"
"Using the data-dotnet patterns, implement a repository"
```

---

## How the Knowledge Base Works

The engineering team references the skill documentation to ensure consistent implementations:

```
skills/ddd-dotnet/       ← Referenced by: Domain Engineer, Architect
        ▲                                ▲
        │                                │
        │ (implements)                   │ (tests)
        │                                │
skills/data-dotnet/      ← Referenced by: Data Engineer
                                         │
                    skills/bdd-dotnet/   ← Referenced by: QA Specialist
```

**Key relationships:**
- **Data Engineer** implements domain interfaces defined by the **Domain Engineer**
- **QA Specialist** tests domain logic using repositories from **Data Engineer**
- **Architect** reviews all implementations for architectural compliance

---

## Architecture

Your engineering team builds applications using **hexagonal architecture** (ports and adapters):

```
┌─────────────────────────────────────────────┐
│           Adapters (Infrastructure)         │
│  - API (WebAPI)                             │
│  - Blazor Apps                              │
│  - Data Persistence ◄─ 🟡 Data Engineer     │
│  - ServiceClients      (data-dotnet skill)  │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│             Ports (Domain)                  │
│  - Aggregates & Entities ◄─ 🟢 Domain       │
│  - Command Handlers           Engineer      │
│  - Query Handlers          (ddd-dotnet)     │
│  - Repository Interfaces                    │
│                                             │
│  Architecture Review ◄───── 🟣 Architect    │
│                           (all skills)      │
└─────────────────────────────────────────────┘
                     ▲
                     │ (tests)
                     │
           ┌─────────────────────┐
           │  Unit Tests         │
           │  🔴 QA Specialist   │
           │  (bdd-dotnet skill) │
           └─────────────────────┘
```

**Core Principle:** Adapters depend on Ports, never the reverse.

**How it works:**
- **Agents** are your engineering team (the "who")
- **Skills** are their knowledge base (the "what")
- Agents automatically load and apply relevant skills when working

---

## Pattern Summary

### Domain Layer (ddd-dotnet/)
- ✅ Aggregates control entities
- ✅ Business logic in aggregates
- ✅ Commands for writes
- ✅ Queries for reads (ViewModels)
- ✅ IClock for time
- ✅ Domain exceptions

### Data Persistence Layer (data-dotnet/)
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

2. **Persistence Second** (using `data-dotnet/`):
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
