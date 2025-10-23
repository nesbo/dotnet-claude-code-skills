# Using .NET Claude Code Skills with Claude Code

This repository contains production-ready .NET skills designed specifically for use with Claude Code, Anthropic's official CLI for Claude.

## What This Repository Provides

This is a **skills library** for .NET development with Claude Code, focusing on:

- **Domain-Driven Design (DDD)** patterns for .NET
- **Entity Framework Core** data persistence implementations
- **BDD-Style Unit Testing** with NUnit and InMemory database
- **Hexagonal Architecture** (Ports and Adapters) patterns
- **CQRS** command and query separation
- **Production-tested code** from real-world projects

## Repository Purpose

This repository serves as a **reference and pattern library** for Claude Code when working on .NET projects. The skills contained here provide:

1. **Comprehensive documentation** of DDD, EF Core, and unit testing patterns
2. **Code templates** for common scenarios
3. **Best practices** and anti-patterns to avoid
4. **Quick reference guides** for implementation
5. **Production-proven examples** from real projects

## How to Use with Claude Code

### Option 1: Reference Skills Directly

When working with Claude Code on your .NET project, reference skills from this repository:

```
"Using the patterns from dotnet-claude-code-skills/ddd-dotnet, create a new aggregate for Product"

"Following the ef-core-ddd skill, implement the repository for Product"

"Using the bdd-dotnet skill, write unit tests for the CreateProduct command handler"

"Review this code against the patterns in dotnet-claude-code-skills"
```

### Option 2: Clone Locally for Reference

Clone this repository to have the skills available as reference material:

```bash
git clone https://github.com/nesbo/dotnet-claude-code-skills.git
cd dotnet-claude-code-skills
```

Then reference the skills in your Claude Code sessions by providing the path to the skill files.

### Option 3: Copy Patterns into Your Project

Copy relevant skill documentation into your own project's documentation folder for project-specific reference.

## What's Inside

### 📁 `ddd-dotnet/`
Complete guide to implementing Domain-Driven Design in .NET with:
- Aggregate and Entity patterns
- Command and Query handlers (CQRS)
- Repository interfaces
- Domain exceptions
- Time control patterns (IClock)
- Quick reference templates

**Read**: [ddd-dotnet/SKILL.md](ddd-dotnet/SKILL.md)

### 📁 `ef-core-ddd/`
Complete guide to implementing Entity Framework Core as a data layer adapter:
- DbContext configuration
- Entity configurations (Fluent API)
- Repository implementations
- Unit of Work pattern
- Auto-registration with DI
- Migration workflows

**Read**: [ef-core-ddd/SKILL.md](ef-core-ddd/SKILL.md)

### 📁 `bdd-dotnet/`
Complete guide to BDD-style unit testing for domain layer handlers:
- Testing philosophy (hybrid approach with real repositories)
- TestDataBuilder and TestContextFactory patterns
- Fluent aggregate builders for test data
- Fake implementations (FakeClock, FakeUnitOfWork, etc.)
- Arrange-Act-Assert pattern
- NUnit conventions and assertions
- Testing patterns for commands, queries, workflows, and errors

**Read**: [bdd-dotnet/SKILL.md](bdd-dotnet/SKILL.md)

## Architecture Context

These skills implement **Hexagonal Architecture** (Ports and Adapters):

```
┌─────────────────────────────────┐
│   Adapters (Infrastructure)    │
│   - Web API                     │
│   - Blazor Apps                 │
│   - Data Layer (EF Core) ◄──────┼─── ef-core-ddd/
│   - Service Clients             │
└─────────────────────────────────┘
              ▲
              │ implements
              ▼
┌─────────────────────────────────┐
│   Ports (Domain Layer)          │
│   - Aggregates & Entities ◄─────┼─── ddd-dotnet/
│   - Command Handlers            │
│   - Query Handlers              │
│   - Repository Interfaces       │
└─────────────────────────────────┘
              ▲
              │ tests
              │
      ┌───────────────┐
      │  Unit Tests   │ ◄──────────────── bdd-dotnet/
      └───────────────┘
```

**Key Principle**: The domain layer (ports) defines interfaces. The infrastructure layer (adapters) implements them. Dependencies flow inward. Tests verify domain behavior using real repositories.

## Technology Stack

These skills are designed for:

- **.NET 8+** (C# 12 features including primary constructors)
- **Entity Framework Core 8+**
- **NUnit 3+** for unit testing
- **Paramore.Brighter** for CQRS command handling
- **PostgreSQL** (patterns adaptable to other databases)

## Example Claude Code Prompts

Here are example prompts to use with Claude Code when referencing these skills:

### Domain Layer Development
```
"Using the ddd-dotnet skill patterns, create a new Order aggregate that tracks order status changes"

"Review my ShoppingCart aggregate against the ddd-dotnet best practices"

"Implement a command handler for UpdateInventory following the ddd-dotnet patterns"
```

### Data Layer Development
```
"Following ef-core-ddd patterns, create entity configurations for the Order aggregate"

"Implement write and query repositories for Product using the ef-core-ddd skill"

"Review my EF Core configurations against ef-core-ddd best practices"
```

### Unit Testing
```
"Using the bdd-dotnet skill, write unit tests for the CreateProduct command handler"

"Following bdd-dotnet patterns, create a test data builder for the Order aggregate"

"Using the bdd-dotnet skill, write tests for workflow state transitions in ImportWorkflow"

"Review my unit tests against the bdd-dotnet best practices"
```

### Full Feature Implementation
```
"Using ddd-dotnet, ef-core-ddd, and bdd-dotnet skills, implement a complete feature for managing product inventory including:
1. Domain aggregate and entities
2. Command and query handlers
3. EF Core configurations
4. Repositories
5. Unit tests for all handlers"
```

## Skills Are Living Documents

These skills are based on production code and are continuously refined. They reflect:

- Modern .NET best practices
- Real-world architectural patterns
- Lessons learned from production systems
- Industry-standard approaches to DDD and CQRS

## Not a Framework or Library

**Important**: This repository is NOT a NuGet package or framework to install. It's a **knowledge base and pattern library** designed to:

- Guide Claude Code in generating consistent, high-quality .NET code
- Provide reference implementations for common patterns
- Document best practices and anti-patterns
- Serve as templates for your own implementations

## Contributing

These skills can be extended with additional .NET patterns:

- API layer patterns (controllers, minimal APIs)
- Authentication and authorization patterns
- Background job processing
- Event-driven architectures
- Integration and E2E testing patterns

Follow the existing format when adding new skills:
1. Create a new folder with descriptive name
2. Add `SKILL.md` with comprehensive documentation
3. Add `README.md` with overview
4. Update main `README.md` to reference the new skill

## License

This repository is provided for educational and professional development purposes. The patterns and code examples are based on real production code but are presented as generic, reusable patterns.

---

**Made for**: .NET Engineers using Claude Code
**Focus**: Domain-Driven Design, CQRS, Hexagonal Architecture
**Tech**: .NET 8+, EF Core 8+, C# 12
