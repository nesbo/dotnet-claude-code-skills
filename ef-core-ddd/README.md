# EF Core Data Persistence Layer for .NET DDD

A comprehensive Claude Code skill for implementing Entity Framework Core as an adapter in hexagonal architecture with Domain-Driven Design.

## What This Skill Covers

This skill provides production-proven patterns for implementing the data persistence layer using EF Core, including:

- **DbContext Configuration**: Production and design-time setup with migration support
- **Entity Configurations**: Fluent API patterns for mapping domain objects to database tables
- **Repository Implementations**: Write and Query repository base classes with CQRS support
- **Unit of Work**: Transaction management across multiple repositories
- **Dependency Injection**: Convention-based auto-registration with validation
- **Schema Organization**: Multi-schema database design for bounded contexts
- **Migration Workflow**: Best practices for creating and applying migrations
- **Performance Optimization**: Eager loading, indexing, and query optimization patterns

## Key Features

- Real-world code examples from production systems
- Internal implementation with public DI registration API
- CQRS separation with dedicated repository types
- Automatic repository discovery and validation
- In-memory database support for testing
- Quick reference templates for common scenarios
- Best practices and common pitfalls
- Complete integration with domain layer patterns

## Complements

This skill builds on the `ddd-dotnet` skill and implements the infrastructure layer (adapter) for the domain layer (ports). The two skills together provide a complete DDD + EF Core implementation guide.

```
Domain Layer (ddd-dotnet)
         ▲
         │ implements
         │
Data Layer (ef-core-ddd)
```

## Based On

This skill is based on real production code from the LMP project, implementing modern .NET practices with:

- C# 12 features (primary constructors, collection expressions)
- Entity Framework Core 8+
- PostgreSQL (easily adaptable to other databases)
- Hexagonal Architecture (ports and adapters)
- CQRS pattern with separate read/write repositories
- Convention-based dependency injection

## Usage

When working on .NET projects with Claude Code, this skill helps you:

1. Configure EF Core DbContext for production and testing
2. Create entity configurations using Fluent API
3. Implement repositories following CQRS pattern
4. Set up automatic repository registration
5. Manage database schemas and migrations
6. Optimize database queries and performance
7. Write testable data layer code

## Architecture Context

This skill implements the **Data adapter** in hexagonal architecture:

```
┌─────────────────────────────────┐
│     Domain (Ports)              │
│  - IRepository<T, TKey>         │
│  - IQueryRepository<T>          │
│  - IUnitOfWork                  │
│  - Aggregates & Entities        │
└─────────────────────────────────┘
              ▲
              │ implements
              │
┌─────────────────────────────────┐
│     Data (Adapter)              │
│  - DbContext                    │
│  - Entity Configurations        │
│  - Repository Implementations   │
│  - Service Registrations        │
└─────────────────────────────────┘
```

## Quick Start

Read [SKILL.md](SKILL.md) for the complete documentation, including:

- Full DbContext setup with migration support
- Entity configuration examples for all scenarios
- Repository base classes and implementations
- Dependency injection setup
- Migration workflow
- Performance optimization patterns
- Quick reference templates
- Complete checklists

For full documentation, see [SKILL.md](SKILL.md).
