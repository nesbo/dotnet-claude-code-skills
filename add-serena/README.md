# Add Serena MCP to Your Project

A comprehensive Claude Code skill for integrating Serena MCP (Model Context Protocol) into any software project with true multi-project support.

## What This Skill Covers

This skill provides step-by-step guidance for adding Serena MCP to your project, enabling:

- **IDE-like semantic code understanding** through Language Server Protocol (LSP) integration
- **Symbol-level navigation** - find definitions, references, and implementations
- **Precise code editing** - insert and replace code at the symbol level
- **Project-aware search** and intelligent file operations
- **Memory system** for storing and retrieving project-specific context
- **Multi-language support** - C#, Python, TypeScript, JavaScript, Go, Rust, Java, and more
- **Independent multi-project setup** - Add Serena to any number of projects without interference

## Multi-Project Philosophy

**Each project gets its own independent Serena configuration.**

Key principles:
- ✅ Per-project `.mcp.json` configuration
- ✅ Complete project isolation (separate memory, state, and context)
- ✅ No global hardcoded paths
- ✅ Version controllable for team sharing
- ✅ Portable helper script approach

## Key Features

- **Interactive setup** with language, version control, and permissions preferences
- **Helper script approach** for portable, team-friendly configuration
- **Complete prerequisite checking guide**
- **Project configuration templates** for all major languages
- **Decision guides** for version control and permissions
- **Verification and testing procedures**
- **Comprehensive troubleshooting guide**
- **Advanced configurations** for monorepos and read-only mode
- **Migration guide** from old global installation approach

## Based On

This skill is based on official Claude Code MCP documentation and Serena best practices. It implements the recommended **per-project MCP server approach** using `.mcp.json` files, ensuring reliable multi-project support.

## Usage

When working with Claude Code, reference this skill to:

1. Add Serena MCP to any project (independently)
2. Configure project-specific language and settings
3. Enable semantic code understanding across languages
4. Set up team-friendly version controlled configurations
5. Troubleshoot installation issues
6. Configure advanced features (read-only mode, excluded tools, etc.)

For full documentation, see [SKILL.md](SKILL.md).

## Quick Start

### For Any Project

```bash
# 1. Navigate to your project
cd /path/to/your/project

# 2. Start Claude Code
claude .

# 3. In Claude Code, use the skill:
# "Using the add-serena skill, add Serena MCP to my current project"

# 4. Answer interactive questions:
# - Programming language (C#, Python, TypeScript, etc.)
# - Version control preference (commit or gitignore)
# - Permissions preference (pre-approve or interactive)

# 5. Restart Claude Code
# Exit (Ctrl+C) and restart: claude .

# 6. Approve project-scoped MCP server
# Choose "Always for this project"

# 7. Verify with /mcp command
```

### What Gets Created

The skill creates these files in your project:

```
your-project/
├── .mcp.json                      # MCP server configuration
├── .serena/
│   ├── start-mcp.sh              # Portable helper script
│   └── project.yml               # Project configuration
└── .claude/
    └── settings.local.json       # Permissions (optional)
```

## Multi-Project Example

Add Serena to multiple projects independently:

```bash
# Project A
cd ~/projects/ecommerce-api
claude .
# "Using add-serena, add Serena to this project"
# Answer questions, restart, verify with /mcp

# Project B
cd ~/projects/analytics-dashboard
claude .
# "Using add-serena, add Serena to this project"
# Answer questions, restart, verify with /mcp

# Both projects now have independent Serena instances!
```

## Language Support

Supported with LSP integration:
- **C#** (.sln files)
- **Python** (pyproject.toml / setup.py)
- **TypeScript/JavaScript** (tsconfig.json / package.json)
- **Go** (go.mod)
- **Rust** (Cargo.toml)
- **Java** (Maven/Gradle)
- And more

Each language has specific prerequisites and configuration requirements detailed in the full skill guide.

## Interactive Questions

The skill asks you three questions to customize the setup:

### 1. Programming Language
Choose your project's primary language for LSP configuration.

### 2. Version Control
**Option A:** Commit Serena config (recommended for team projects)
- Team gets the same setup
- Faster onboarding

**Option B:** Gitignore Serena config (recommended for personal projects)
- Optional personal tooling
- Each developer configures independently

### 3. Permissions
**Option A:** Pre-approve Serena tools (less prompting)
**Option B:** Approve interactively (more explicit control)

## Why Per-Project Approach?

### Advantages

- ✅ **True isolation:** Each project maintains separate memory and state
- ✅ **No reconfiguration:** Switch between projects seamlessly
- ✅ **Team-friendly:** Version control the setup, everyone gets it
- ✅ **Portable:** Helper script auto-detects project root
- ✅ **Reliable:** No dependency on global config paths

### Compared to Global Installation

The older global installation approach (`claude mcp add` with hardcoded path) had limitations:
- ❌ Only worked for one specific project path
- ❌ Required manual editing of `~/.claude.json` for each project
- ❌ Projects interfered with each other
- ❌ Not team-friendly (everyone had different paths)

The new per-project approach solves all these issues.

## Decision Guides

### Should I Version Control Serena Config?

**Yes, commit it when:**
- Working on team project
- Want consistent tooling across team
- Serena is essential to workflow

**No, gitignore it when:**
- Solo or personal project
- Team doesn't use Claude Code
- Optional tooling preference

### Should I Pre-Approve Serena Tools?

**Yes, pre-approve when:**
- You trust Serena and use frequently
- Want faster workflow

**No, approve interactively when:**
- First time using Serena
- Working on sensitive codebase
- Prefer explicit control

See [SKILL.md](SKILL.md) for detailed decision guides.

## Troubleshooting

Common issues and solutions:

- **Serena not loading:** Check files exist, verify script executable, restart Claude Code
- **Project-scoped server denied:** Run `claude mcp reset-project-choices` and approve
- **uv/uvx not found:** Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **LSP not working:** Verify language-specific files (.sln, pyproject.toml, etc.) exist

See [SKILL.md](SKILL.md) for comprehensive troubleshooting guide.

## Migration from Global Installation

If you previously used `claude mcp add` with hardcoded path:

```bash
# Optional: Remove global Serena
claude mcp remove serena

# Add Serena to each project independently
cd /path/to/project
claude .
# "Using add-serena, add Serena to this project"
```

Benefits:
- All projects now work independently
- No manual config editing
- Team-friendly version control

## References

- **Full Skill Documentation:** [SKILL.md](SKILL.md)
- **Claude Code MCP Docs:** https://docs.claude.com/en/docs/claude-code/mcp
- **Claude Code Project-Scoped Servers:** https://docs.claude.com/en/docs/claude-code/mcp/project-scoped-servers
- **Serena Project:** https://github.com/oraios/serena
- **uv Installation:** https://github.com/astral-sh/uv

## Summary

This skill enables you to easily add Serena MCP to **any project independently**, with:
- Interactive setup (language, version control, permissions)
- Portable helper script approach
- Complete project isolation
- Team-friendly version control support
- Comprehensive documentation and troubleshooting

Simply say: **"Using add-serena, add Serena MCP to my current project"** and follow the interactive prompts.
