# Claude Code Skills Library

This directory contains reusable skills for implementing consistent patterns across the LMP project and similar .NET/DDD projects.

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

### 2. EF Core Data Layer (`ef-core-ddd/`)

**Purpose**: Implementing the data persistence layer as an adapter in hexagonal architecture using Entity Framework Core.

**What it covers:**
- DbContext configuration (production & testing)
- Entity configurations using Fluent API
- Repository implementations (write & query)
- Unit of Work implementation
- Dependency injection auto-registration
- In-memory database testing support
- Migration workflow
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

## Skill Organization

```
dotnet-claude-code-skills/
├── README.md                    # This file - skill index
├── ddd-dotnet/                  # Domain layer patterns
│   ├── SKILL.md
│   └── README.md
└── ef-core-ddd/                 # Data persistence patterns
    ├── SKILL.md
    └── README.md
```

---

## How to Use These Skills

### In Claude Code Sessions

Reference skills directly in your prompts:

```
"Using the ddd-dotnet skill, create a new aggregate for Product"

"Following the ef-core-ddd skill, implement the repository for Product"

"Review this code against the patterns in ddd-dotnet/SKILL.md"
```

### As Reference Material

- Open skill files in your IDE for quick reference
- Copy templates as starting points for new code
- Use checklists before creating pull requests
- Share with team members for consistency

### For Code Reviews

Use the "Best Practices" and "Common Pitfalls" sections to:
- Validate new code follows patterns
- Identify anti-patterns
- Suggest improvements
- Ensure consistency across the codebase

---

## Skill Dependencies

```
ddd-dotnet/
        ▲
        │ (implements interfaces from)
        │
ef-core-ddd/
```

The data layer skill builds on concepts from the DDD basics skill. Read `ddd-dotnet/SKILL.md` first to understand the domain interfaces that the data layer implements.

---

## Architecture Context

These skills support a **hexagonal architecture** (ports and adapters):

```
┌─────────────────────────────────────────────┐
│           Adapters (Infrastructure)         │
│  - API (WebAPI)                             │
│  - Blazor Apps                              │
│  - Data (EF Core) ◄── ef-core-ddd          │
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

### Data Layer (ef-core-ddd/)
- ✅ Entity configurations (Fluent API)
- ✅ Repository base classes
- ✅ CQRS separation (write/query)
- ✅ Auto-registration via reflection
- ✅ In-memory testing support
- ✅ Schema organization

---

## Quick Start Workflow

### Adding a New Feature with Both Skills

1. **Domain First** (using `ddd-dotnet/`):
   ```
   Create aggregate → Create entity → Create command & handler → Create query & handler
   ```

2. **Persistence Second** (using `ef-core-ddd/`):
   ```
   Create entity config → Create write repo → Create query repo → Create migration → Test
   ```

3. **Verification**:
   - Check domain best practices
   - Check data best practices
   - Run tests with in-memory database
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

## License & Usage

These skills are internal to the LMP project but can be adapted for other .NET/DDD projects following similar architectural patterns.

---

**Last Updated**: 2025-10-22
**Skills Version**: 1.0
**Applicable to**: .NET 8+, EF Core 8+, Hexagonal Architecture, DDD
