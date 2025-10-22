# .NET Claude Code Skills

State-of-the-art .NET development skills for Claude Code, featuring Domain-Driven Design patterns and best practices for professional software engineering.

## Overview

This repository contains Claude Code skills specifically designed for .NET engineers. These skills provide comprehensive guidance on implementing modern software architecture patterns, clean code principles, and industry best practices in .NET applications.

## Available Skills

### Domain-Driven Design for .NET

**Path:** `ddd-dotnet/`

A comprehensive guide to implementing Domain-Driven Design patterns in .NET, based on real-world production code. This skill covers:

- **Core DDD Building Blocks**: Aggregates, Entities, Value Objects, and Domain Services
- **CQRS Pattern**: Separate read and write operations for optimized performance
- **Hexagonal Architecture**: Ports (domain) and Adapters (infrastructure) separation
- **Repository Pattern**: Write and Query repositories with proper abstraction
- **Command & Query Handlers**: Using Paramore.Brighter for commands and simple services for queries
- **Time Control Pattern**: Deterministic time handling using IClock interface
- **Domain Exceptions**: Proper error handling with custom domain exceptions
- **Unit of Work**: Transaction management across multiple repositories
- **Best Practices**: Patterns from production code including versioned aggregates, change tracking, and more

**Key Benefits:**
- Clear separation of concerns
- Highly testable domain logic
- Maintainable and scalable architecture
- Production-proven patterns

## Installation

### Using Claude Code Plugin Command (Recommended)

Install this skill marketplace directly from GitHub:

```bash
/plugin marketplace add nesbo/dotnet-claude-code-skills
```

Then install the DDD skill:

```bash
/plugin install ddd-dotnet@nesbo/dotnet-claude-code-skills
```

### Manual Installation

Alternatively, you can manually clone and reference the skills:

1. Clone this repository
2. Reference the skill files in your Claude Code configuration
3. The skills will be available for Claude to use when working on .NET projects

## Usage

When working with Claude Code on .NET projects, these skills provide:

- Architecture guidance and patterns
- Code examples and templates
- Best practices and common pitfalls
- Quick reference guides

Claude will use these skills to help you implement clean, maintainable, and production-ready .NET applications following DDD principles and modern architectural patterns.

## Skill Structure

Each skill folder contains:
- `SKILL.md` - The main skill documentation with patterns, examples, and guidance
- `README.md` - Brief overview for GitHub preview

## Contributing

Feel free to contribute additional skills or improvements to existing ones. Skills should:

- Be based on real-world, production-tested patterns
- Include clear code examples
- Follow .NET and C# best practices
- Provide both guidance and practical templates

## License

This repository is provided as-is for educational and professional development purposes.

## About

Created for .NET engineers who want to leverage Claude Code with deep domain expertise in modern .NET architecture patterns, particularly Domain-Driven Design, CQRS, and Hexagonal Architecture.
