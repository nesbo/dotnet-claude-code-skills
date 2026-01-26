# Claude Code - Complete .NET Engineering Team

**Your AI-powered engineering team for production-ready .NET development.**

This repository provides a complete engineering team of specialized AI agents, each an expert in their domain, working together to build applications using Domain-Driven Design, CQRS, and Hexagonal Architecture.

> **Universal Applicability**: While demonstrated with C# and .NET, these patterns apply to any object-oriented programming language (Java, TypeScript, Python, etc.). The principles of DDD and hexagonal architecture are language-agnostic.

## Your Engineering Team

**4 specialized agents working as a cohesive team:**

- 🟢 **Domain Engineer** (Sonnet) - Implements domain logic and business rules
- 🔴 **QA Specialist** (Sonnet) - Writes comprehensive BDD-style tests
- 🟡 **Data Engineer** (Sonnet) - Implements data persistence layer
- 🟣 **Software Architect** (Opus) - Provides architectural guidance

**3 comprehensive skill guides** that agents reference:

- **ddd-dotnet** - Domain-Driven Design patterns
- **data-dotnet** - Data persistence implementations
- **bdd-dotnet** - BDD-style unit testing patterns

## What You Get

This engineering team provides:

1. **Specialized AI agents** - Each agent is an expert in their domain (domain logic, testing, data, architecture)
2. **Comprehensive knowledge base** - Production-proven patterns and best practices
3. **Guided workflows** - Agents work together on complete features
4. **Consistent implementation** - All code follows the same architectural patterns
5. **Quality assurance** - Built-in testing and architectural review

## Installation

### Install Your Engineering Team

Install the complete team in Claude Code using the `/plugin` command:

```bash
/plugin marketplace add https://github.com/nesbo/dotnet-claude-code-skills
```

This installs:
- **4 specialized agents** (dotnet-agents plugin)
- **3 comprehensive skills** (ddd-dotnet, data-dotnet, bdd-dotnet)

The agents will automatically reference the skills when working on your code.

## Knowledge Base

### 📁 `skills/ddd-dotnet/`
Complete guide to implementing Domain-Driven Design in .NET with:
- Aggregate and Entity patterns
- Command and Query handlers (CQRS)
- Repository interfaces
- Domain exceptions
- Time control patterns (IClock)
- Quick reference templates

**Read**: [skills/ddd-dotnet/SKILL.md](skills/ddd-dotnet/SKILL.md)

### 📁 `skills/data-dotnet/`
Complete guide to implementing data persistence as an adapter layer (demonstrated with one ORM approach):
- Data context configuration
- Entity-to-storage mapping
- Repository implementations
- Unit of Work pattern
- Auto-registration with DI
- Migration workflows

**Read**: [skills/data-dotnet/SKILL.md](skills/data-dotnet/SKILL.md)

### 📁 `skills/bdd-dotnet/`
Complete guide to BDD-style unit testing for domain layer handlers:
- Testing philosophy (hybrid approach with real repositories)
- TestDataBuilder and TestContextFactory patterns
- Fluent aggregate builders for test data
- Fake implementations (FakeClock, FakeUnitOfWork, etc.)
- Arrange-Act-Assert pattern
- NUnit conventions and assertions
- Testing patterns for commands, queries, workflows, and errors

**Read**: [skills/bdd-dotnet/SKILL.md](skills/bdd-dotnet/SKILL.md)

---

## Architecture

Your engineering team builds applications using **Hexagonal Architecture** (Ports and Adapters):

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

**Core Principle**: Adapters depend on Ports, never the reverse.

**How it works**: Each agent is responsible for their layer and automatically references the appropriate skill documentation.

## Implementation Stack

These skills are demonstrated using:

- **.NET 8+** with C# 12 (primary constructors, collection expressions)
- **One ORM approach** for data persistence examples
- **NUnit 3+** for unit testing examples
- **Paramore.Brighter** for CQRS command handling (one implementation approach)
- **PostgreSQL** examples (patterns work with any database)

The underlying DDD and hexagonal architecture patterns apply to any OOP language and technology stack.

## Example Claude Code Prompts

### Working with Your Engineering Team

#### Domain Implementation with Agents
```
"Use the ddd-domain-engineer agent to create a new Order aggregate"

"Use the ddd-domain-engineer agent to implement a command handler for UpdateInventory"

"Use the ddd-domain-engineer agent to refactor the ShoppingCart aggregate"
```

#### Testing with Agents
```
"Use the bdd-qa-specialist agent to write tests for CreateProduct handler"

"Use the bdd-qa-specialist agent to plan test scenarios for the Driver aggregate"

"Use the bdd-qa-specialist agent to create end-to-end tests for order fulfillment workflow"
```

#### Data Layer with Agents
```
"Use the data-layer-engineer agent to implement repositories for Product"

"Use the data-layer-engineer agent to optimize the shipment list query"

"Use the data-layer-engineer agent to create entity configurations for Order aggregate"
```

#### Architecture Review with Agents
```
"Use the hexagonal-architecture-modeler agent to review my architecture"

"Use the hexagonal-architecture-modeler agent to design a payment processing feature"

"Use the hexagonal-architecture-modeler agent to refactor this code to follow hexagonal architecture"
```

#### Complete Feature Workflow
```
"Use the hexagonal-architecture-modeler to design a payment feature,
then the ddd-domain-engineer to implement domain logic,
the data-layer-engineer for persistence,
and the bdd-qa-specialist to write comprehensive tests"
```

### Referencing Skills Directly (Manual Work)

If you prefer to implement manually, reference the skill documentation:

```
"Following the ddd-dotnet skill, create an Order aggregate"
"Using data-dotnet patterns, implement a repository for Product"
"Review my code against the bdd-dotnet testing patterns"
```

### When to Use What

**Use Agents when:**
- You want guided, semi-automated implementation
- You're implementing complete features
- You want the team to work together on complex tasks

**Reference Skills when:**
- You want to implement manually but follow patterns
- You're doing code reviews
- You need quick pattern lookups

## Production-Ready Patterns

The knowledge base and agents are built from real-world production code and reflect:

- Modern .NET best practices
- Real-world architectural patterns
- Lessons learned from production systems
- Industry-standard approaches to DDD and CQRS

All patterns apply to any OOP language (Java, TypeScript, Python, etc.)

## Future Enhancements

**Additional team members** could include:
- API Layer Engineer (REST/GraphQL endpoints)
- Security Engineer (authentication, authorization)
- Integration Engineer (external services, messaging)
- Performance Engineer (optimization, caching)

**Additional skills** could cover:
- API layer patterns
- Authentication and authorization
- Background job processing
- Event-driven architectures
- Performance optimization

## License

This repository is provided for educational and professional development purposes. The patterns and code examples are based on real production code but are presented as generic, reusable patterns.

---

**Your Engineering Team**: 4 specialized AI agents + 3 comprehensive skill guides
**Architecture**: Domain-Driven Design, CQRS, Hexagonal Architecture
**Demonstrated in**: .NET 8+ and C# 12
**Applicable to**: Any OOP language (Java, TypeScript, Python, etc.)
