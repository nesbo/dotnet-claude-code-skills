# Data Persistence Layer for DDD

A comprehensive Claude Code skill for implementing data persistence as an adapter in hexagonal architecture with Domain-Driven Design.

> **Implementation Note**: This skill demonstrates one approach to data persistence using an ORM. The patterns shown here (repository implementation, entity mapping, unit of work) apply regardless of the specific ORM or data access technology you use.

## What This Skill Covers

This skill provides production-proven patterns for implementing the data persistence layer, including:

- **Data Context Configuration**: Production and design-time setup with migration support
- **Entity Mapping**: Patterns for mapping domain objects to database tables
- **Repository Implementations**: Write and Query repository base classes with CQRS support
- **Unit of Work**: Transaction management across multiple repositories
- **Dependency Injection**: Convention-based auto-registration with validation
- **Schema Organization**: Multi-schema database design for bounded contexts
- **Migration Workflow**: Best practices for creating and applying schema changes
- **Performance Optimization**: Loading strategies, indexing, and query optimization patterns

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

This skill builds on the `ddd-dotnet` skill and implements the infrastructure layer (adapter) for the domain layer (ports). The two skills together provide a complete DDD + data persistence implementation guide.

```
Domain Layer (ddd-dotnet)
         ▲
         │ implements
         │
Data Layer (data-dotnet)
```

## Based On

This skill is based on real production code, implementing modern practices with:

- C# 12 features (primary constructors, collection expressions)
- ORM for data persistence (demonstrated with one specific implementation)
- PostgreSQL examples (patterns work with any database)
- Hexagonal Architecture (ports and adapters)
- CQRS pattern with separate read/write repositories
- Convention-based dependency injection

The patterns shown apply universally, regardless of your specific ORM or programming language.

## Usage

When working on object-oriented projects with Claude Code, this skill helps you:

1. Configure data context for production and testing
2. Create entity-to-storage mappings
3. Implement repositories following CQRS pattern
4. Set up automatic repository registration
5. Manage database schemas and migrations
6. Optimize database queries and performance
7. Write testable data persistence code

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
│  - Data Context                 │
│  - Entity Mappings              │
│  - Repository Implementations   │
│  - Service Registrations        │
└─────────────────────────────────┘
```

## Quick Start

Read [SKILL.md](SKILL.md) for the complete documentation, including:

- Full data context setup with migration support
- Entity mapping examples for all scenarios
- Repository base classes and implementations
- Dependency injection setup
- Migration workflow
- Performance optimization patterns
- Quick reference templates
- Complete checklists

For full documentation, see [SKILL.md](SKILL.md).
