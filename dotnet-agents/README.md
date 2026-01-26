# dotnet-agents Plugin

Specialized development agents for .NET projects following Domain-Driven Design, CQRS, and Hexagonal Architecture patterns.

## Overview

This plugin provides four specialized agents that work together to implement complete features in .NET applications:

- **ddd-domain-engineer** (Green, Sonnet): Domain logic specialist
- **bdd-qa-specialist** (Red, Sonnet): BDD testing specialist
- **data-layer-engineer** (Yellow, Sonnet): Data persistence specialist
- **hexagonal-architecture-modeler** (Purple, Opus): Architecture consultant

## How It Works

These agents are designed to work with the skill plugins in this marketplace:

- **ddd-dotnet**: Domain-Driven Design patterns and best practices
- **data-dotnet**: Data persistence and repository patterns
- **bdd-dotnet**: BDD-style unit testing patterns

Each agent references the relevant skill plugins to ensure consistent implementation following project patterns.

## Installation

Install via Claude Code marketplace:

```bash
/plugin marketplace add https://github.com/nesbo/dotnet-claude-code-skills
/plugin install dotnet-agents
```

Or install the entire marketplace:

```bash
/plugin marketplace add https://github.com/nesbo/dotnet-claude-code-skills
/plugin install ddd-dotnet data-dotnet bdd-dotnet dotnet-agents
```

## Usage

### Available Agents

#### ddd-domain-engineer (Green)

Use for implementing domain logic, aggregates, entities, command handlers, and business rules.

**Example triggers:**
- "I need to create a new Order aggregate"
- "Add validation to ensure orders can only be picked if they're in Pending status"
- "Implement a command handler for UpdateInventory"

**What it does:**
- Loads ddd-dotnet skill for current patterns
- Implements aggregates and entities following DDD principles
- Creates command and query handlers
- Ensures hexagonal architecture compliance
- Applies SOLID principles to domain code

#### bdd-qa-specialist (Red)

Use for writing BDD-style unit tests for domain handlers.

**Example triggers:**
- "Write tests for the CreateShipmentOrderHandler"
- "I've implemented UpdateInventory handler, now I need test coverage"
- "Plan test scenarios for the Driver aggregate commands"

**What it does:**
- Loads bdd-dotnet skill for test patterns
- Creates behavior-focused test scenarios
- Implements fake dependencies (no mocking frameworks)
- Follows Given-When-Then structure
- Tests business rules and domain invariants

#### data-layer-engineer (Yellow)

Use for data access layer implementation with EF Core.

**Example triggers:**
- "Create a repository for the ShipperOrder aggregate"
- "I need database setup for the new Shipment aggregate"
- "The shipment list query is loading slowly"

**What it does:**
- Loads data-dotnet skill for repository patterns
- Creates repository implementations (write and query)
- Implements EF Core entity configurations
- Optimizes queries and prevents N+1 problems
- Enforces strict aggregate boundary rules

#### hexagonal-architecture-modeler (Purple, Opus)

Use for architectural guidance and design reviews.

**Example triggers:**
- "Design the architecture for a payment processing feature"
- "Review my UserService for architectural compliance"
- "Help me refactor this codebase to follow hexagonal architecture"

**What it does:**
- Designs hexagonal architecture solutions
- Reviews code for architectural compliance
- Applies DDD and SOLID principles
- Ensures testability
- Identifies and fixes architectural violations

### Workflow Examples

#### Complete Feature Implementation

```
1. Use hexagonal-architecture-modeler to design the feature
2. Use ddd-domain-engineer to implement domain logic
3. Use data-layer-engineer to implement data persistence
4. Use bdd-qa-specialist to write comprehensive tests
```

#### Test-First Development (Shift-Left)

```
1. Use bdd-qa-specialist to plan test scenarios first
2. Use ddd-domain-engineer to implement handlers (tests guide design)
3. Use data-layer-engineer to implement repositories
4. Use bdd-qa-specialist again to verify test coverage
```

#### Refactoring Existing Code

```
1. Use hexagonal-architecture-modeler to analyze current architecture
2. Use ddd-domain-engineer to refactor domain logic
3. Use data-layer-engineer to refactor data layer
4. Use bdd-qa-specialist to ensure tests cover refactored code
```

## Agent Colors and Models

- **Green (ddd-domain-engineer)**: Sonnet model - balance of speed and quality for implementation
- **Red (bdd-qa-specialist)**: Sonnet model - efficient test generation
- **Yellow (data-layer-engineer)**: Sonnet model - focused data layer work
- **Purple (hexagonal-architecture-modeler)**: Opus model - architectural reasoning requires highest quality

## Relationship to Skill Plugins

These agents are workflow specialists that **reference** the skill plugins:

- Agents provide specialized workflows and task execution
- Skills provide patterns, conventions, and best practices
- Agents load skills as needed to stay current with project patterns

You can use skills directly (for reference) or agents (for execution):

**Direct skill reference:**
```
"Following the ddd-dotnet skill, create an Order aggregate"
```

**Agent usage:**
```
"Use the ddd-domain-engineer agent to create an Order aggregate"
```

Agents provide more specialized workflows and context-aware implementation.

## Future Enhancements

The `commands/` directory is reserved for future command-based shortcuts:

- `/ddd-create-aggregate` - Quick aggregate scaffolding
- `/ddd-create-handler` - Quick command handler creation
- `/data-create-repo` - Quick repository creation
- `/test-handler` - Quick test generation for a handler

## Contributing

To add new agents or enhance existing ones, follow the agent markdown format with YAML front matter:

```markdown
---
name: agent-name
description: "Agent description with examples"
model: sonnet|opus
color: green|red|yellow|purple|blue
---

Agent instructions here...
```

## License

Same as dotnet-claude-code-skills repository.

---

**Made for**: Software Engineers using Claude Code with .NET
**Works with**: ddd-dotnet, data-dotnet, bdd-dotnet skill plugins
**Focus**: Specialized workflows for DDD, CQRS, and Hexagonal Architecture
