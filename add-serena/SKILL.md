# Add Serena MCP to Any Project

This skill helps you add Serena MCP (Model Context Protocol) to any software project, providing IDE-like semantic code understanding and navigation capabilities with true multi-project support.

## What is Serena MCP?

Serena is a coding agent toolkit that provides:
- **Language Server Protocol (LSP)** integration for semantic code understanding
- **Symbol-level navigation** (find definitions, references, implementations)
- **Precise code editing tools** (insert after symbol, replace symbol body)
- **Project-aware search and file operations**
- **Memory system** for storing project-specific context
- **Onboarding and project structure analysis**

Supported languages: C#, Python, TypeScript, JavaScript, Go, Rust, Java, and more.

## Multi-Project Philosophy

**This skill enables you to add Serena to ANY project independently.**

Key principles:
- **Project-specific configuration** - Each project has its own `.mcp.json` and `.serena/` folder
- **Complete isolation** - Projects don't interfere with each other
- **Separate memory and state** - Each project maintains its own context
- **No global installation** - No hardcoded paths in `~/.claude.json`
- **Team-friendly** - Configuration can be version controlled for team sharing

**How it works:**
1. You add Serena to Project A → Works in Project A
2. You add Serena to Project B → Works in Project B
3. Both projects maintain separate Serena instances with isolated state
4. You can work on multiple projects without reconfiguration

## Prerequisites Check

Before adding Serena, verify:

### 1. uv (Python package manager) is installed

```bash
which uv
# Should return: /Users/nesbo/.local/bin/uv or similar
```

If not installed, see: https://github.com/astral-sh/uv

### 2. Python 3.11 is available

```bash
python3 --version
# Should show Python 3.11.x (Serena requires >=3.11, <3.12)
```

### 3. Language-specific requirements

- **C#:** Requires .sln file in project directory
- **Python:** Requires pyproject.toml or setup.py
- **TypeScript/JavaScript:** Requires tsconfig.json or package.json
- **Go:** Requires go.mod
- **Rust:** Requires Cargo.toml
- **Java:** Requires Maven or Gradle project file

### 4. Git repository (optional but recommended)

For .gitignore integration and version control benefits.

## Installation Approach Overview

**Per-Project Configuration with Helper Script**

This approach creates three main components in your project:

1. **`.mcp.json`** - Tells Claude Code to load Serena MCP server for this project
2. **`.serena/start-mcp.sh`** - Portable helper script that auto-detects project root
3. **`.serena/project.yml`** - Project-specific Serena configuration

**Benefits:**
- ✅ Each project is self-contained and independent
- ✅ Completely portable across team members
- ✅ No manual path configuration needed
- ✅ Can be version controlled for team consistency
- ✅ Works reliably regardless of where you start Claude Code

**Why helper script?**
- MCP servers can't reliably detect current working directory when launched via `uvx`
- Helper script ensures Serena always knows the correct project root
- More portable than hardcoded absolute paths

## Interactive Setup Guide

When using this skill, Claude Code will ask you several questions to customize the setup:

### Question 1: Programming Language

**"Which programming language is this project?"**

Options: C#, Python, TypeScript, JavaScript, Go, Rust, Java, Other

This determines:
- Language server configuration in `.serena/project.yml`
- Which project files to look for (.sln, pyproject.toml, etc.)
- LSP features available

### Question 2: Version Control

**"Should Serena configuration be version controlled and committed to git?"**

**Option A: Yes, commit for team (Recommended for team projects)**
- `.mcp.json`, `.serena/start-mcp.sh`, and `.serena/project.yml` are committed
- Entire team gets the same Serena setup
- No manual configuration per developer
- Best for: Team projects, shared codebases

**Option B: No, keep it local (Recommended for personal tooling)**
- Add Serena files to `.gitignore`
- Each developer configures independently
- Optional tooling, not required for the project
- Best for: Solo projects, experimental tools

### Question 3: Permissions

**"Should Serena tools be pre-approved?"**

**Option A: Yes, create settings.local.json with common tools**
- Pre-approves frequently used Serena tools
- Less interactive prompting during use
- Still safe - only Serena tools are pre-approved
- Best for: Frequent Serena users

**Option B: No, I'll approve interactively**
- Claude Code prompts for each tool use
- More explicit control
- More clicking required
- Best for: Security-conscious users, occasional use

## Installation Steps

### Step 1: Navigate to Your Project

```bash
cd /path/to/your/project
```

Ensure you're in the project root directory where you want to add Serena.

### Step 2: Use the add-serena Skill

In Claude Code, say:

```
Using the add-serena skill, add Serena MCP to my current project
```

or

```
@add-serena set up Serena for this project
```

Claude Code will then ask you the interactive questions (language, version control, permissions) and create all necessary files.

### Step 3: Files Created

The skill creates the following structure:

```
your-project/
├── .mcp.json                      # MCP server configuration
├── .serena/
│   ├── start-mcp.sh              # Helper script (auto-detects project root)
│   └── project.yml               # Serena project configuration
└── .claude/
    └── settings.local.json       # Permissions (if you chose pre-approval)
```

#### .mcp.json

```json
{
  "mcpServers": {
    "serena": {
      "command": "bash",
      "args": [".serena/start-mcp.sh"]
    }
  }
}
```

This tells Claude Code: "When running in this directory, start the Serena MCP server using the helper script."

#### .serena/start-mcp.sh

```bash
#!/bin/bash
PROJECT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project "$PROJECT_DIR"
```

This script:
- Auto-detects the project root directory
- Launches Serena with `uvx` (downloads and runs from GitHub)
- Passes the correct project path to Serena

#### .serena/project.yml

```yaml
language: csharp  # or python, typescript, etc. based on your answer
ignore_all_files_in_gitignore: true
ignored_paths: []
read_only: false
excluded_tools: []
initial_prompt: ""
project_name: "YourProjectName"
```

Configuration options:
- `language`: Programming language for LSP integration
- `ignore_all_files_in_gitignore`: Respect .gitignore patterns
- `ignored_paths`: Additional paths to ignore
- `read_only`: If true, disables editing tools
- `excluded_tools`: List of tools to disable
- `project_name`: Human-readable project identifier

#### .claude/settings.local.json (Optional)

```json
{
  "permissions": {
    "allow": [
      "mcp__serena__list_dir",
      "mcp__serena__find_file",
      "mcp__serena__get_symbols_overview",
      "mcp__serena__search_for_pattern",
      "mcp__serena__find_symbol",
      "mcp__serena__find_referencing_symbols",
      "mcp__serena__read_file",
      "mcp__serena__write_memory",
      "mcp__serena__read_memory",
      "mcp__serena__think_about_collected_information",
      "mcp__serena__onboard"
    ]
  }
}
```

Only created if you chose to pre-approve Serena tools.

### Step 4: Update .gitignore

The skill will update (or create) `.gitignore` based on your version control choice:

**If you chose to version control Serena config:**

```gitignore
# Serena cache and state (project-specific, don't commit)
.serena/.cache/
.serena/.state/
.serena/memories/
```

**If you chose NOT to version control:**

```gitignore
# Serena MCP configuration (personal preference, not committed)
.mcp.json
.serena/

# Claude Code personal settings
.claude/settings.local.json
```

### Step 5: Restart Claude Code

After the skill creates the files:

1. **Exit Claude Code completely** (Ctrl+C or exit command)
2. **Navigate to your project directory:**
   ```bash
   cd /path/to/your/project
   ```
3. **Start Claude Code:**
   ```bash
   claude .
   ```
   or
   ```bash
   claude code
   ```

### Step 6: Approve Project-Scoped MCP Server

When Claude Code starts, you'll see a prompt:

```
Project-scoped MCP server 'serena' found in .mcp.json
Do you want to enable this server?
- Always for this project
- Just this session
- Never
```

Choose **"Always for this project"** to avoid being asked again.

**Security Note:** Claude Code prompts for approval because project-scoped MCP servers can execute code. Only approve servers you trust.

## Verification & Testing

### Check MCP Server Status

Run the `/mcp` command in Claude Code:

```
/mcp
```

You should see:
```
MCP Servers:
- serena (project-scoped, enabled)
```

### Test Basic Functionality

Try these commands to verify Serena is working:

1. **List directory contents:**
   ```
   Use mcp__serena__list_dir to list files in src/
   ```

2. **Find a symbol:**
   ```
   Use mcp__serena__find_symbol to find the definition of MyClass
   ```

3. **Search for pattern:**
   ```
   Use mcp__serena__search_for_pattern to search for "TODO" in the codebase
   ```

4. **Onboard project:**
   ```
   Use mcp__serena__onboard to analyze the project structure
   ```

### Verify Project Isolation

To confirm multiple projects work independently:

1. Add Serena to Project A
2. Add Serena to Project B
3. Start Claude Code in Project A - Serena should work
4. Start Claude Code in Project B - Serena should work independently
5. Check `.serena/.state/` in each project - should be different

## Available Serena Tools

Once configured, these tools are available through Claude Code:

### File & Project Navigation
- `mcp__serena__list_dir` - List directory contents
- `mcp__serena__find_file` - Find files by pattern
- `mcp__serena__read_file` - Read file contents
- `mcp__serena__search_for_pattern` - Search for text patterns

### Symbol-Level Code Understanding
- `mcp__serena__get_symbols_overview` - Get overview of symbols in a file
- `mcp__serena__find_symbol` - Find symbol definitions
- `mcp__serena__find_referencing_symbols` - Find all references to a symbol
- `mcp__serena__get_document_symbols` - Get all symbols in a document
- `mcp__serena__get_symbol_definition` - Get symbol definition

### Code Editing (if read_only: false)
- `mcp__serena__insert_after_symbol` - Insert code after a symbol
- `mcp__serena__replace_symbol_body` - Replace symbol implementation
- `mcp__serena__delete_symbol` - Delete a symbol

### Memory & Context
- `mcp__serena__write_memory` - Store project-specific information
- `mcp__serena__read_memory` - Retrieve stored information
- `mcp__serena__think_about_collected_information` - Analyze collected data

### Project Understanding
- `mcp__serena__onboard` - Analyze and understand project structure

## Decision Guides

### Should I Version Control Serena Config?

**Commit Serena config (.mcp.json, .serena/) when:**
- ✅ Working on a team project
- ✅ Want all developers to have same tools
- ✅ Serena is essential to project workflow
- ✅ Want to onboard new developers faster
- ✅ Team agrees on using Claude Code + Serena

**Gitignore Serena config when:**
- ❌ Solo project or personal experimentation
- ❌ Team doesn't use Claude Code
- ❌ Serena is optional tooling, not required
- ❌ Developers have different tool preferences

### Should I Pre-Approve Serena Tools?

**Pre-approve (create settings.local.json) when:**
- ✅ You trust Serena and use it frequently
- ✅ Want faster workflow with less clicking
- ✅ Comfortable with Serena's capabilities
- ✅ Project is your own or trusted team project

**Approve interactively (skip settings.local.json) when:**
- ❌ First time using Serena
- ❌ Want explicit control over each tool use
- ❌ Working on sensitive or unfamiliar codebase
- ❌ Prefer security through explicit approval

### What Should Be in .gitignore?

**Always gitignore:**
```gitignore
# Serena runtime state (never commit)
.serena/.cache/
.serena/.state/
.serena/memories/
```

**Conditionally gitignore:**
```gitignore
# Add these if you chose NOT to version control Serena:
.mcp.json
.serena/start-mcp.sh
.serena/project.yml

# Personal Claude Code settings (usually don't commit)
.claude/settings.local.json
```

## Troubleshooting

### Serena Not Loading

**Symptom:** `/mcp` command doesn't show Serena

**Solutions:**
1. Check files exist:
   ```bash
   ls -la .mcp.json .serena/start-mcp.sh .serena/project.yml
   ```

2. Verify helper script is executable:
   ```bash
   chmod +x .serena/start-mcp.sh
   ```

3. Check for syntax errors in JSON/YAML:
   ```bash
   cat .mcp.json
   cat .serena/project.yml
   ```

4. Completely exit and restart Claude Code

5. Check you approved the project-scoped MCP server

### "Project-scoped server denied" Error

**Symptom:** Serena doesn't load even though files exist

**Solution:**
You previously denied the Serena MCP server. Reset project choices:

```bash
claude mcp reset-project-choices
```

Then restart Claude Code and choose "Always for this project" when prompted.

### uv or uvx Not Found

**Symptom:** Error about `uvx: command not found`

**Solutions:**
1. Verify uv is installed:
   ```bash
   which uv
   ```

2. Install uv if missing:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

3. Ensure uv is in PATH:
   ```bash
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc  # or ~/.bashrc
   source ~/.zshrc
   ```

### Language Server Not Working

**Symptom:** Symbol navigation doesn't work, Serena loads but LSP features fail

**Solutions:**
1. Verify language-specific file exists:
   - **C#:** Check for .sln file: `ls *.sln`
   - **Python:** Check for pyproject.toml: `ls pyproject.toml`
   - **TypeScript:** Check for tsconfig.json: `ls tsconfig.json`

2. Verify correct language in `.serena/project.yml`:
   ```bash
   grep language .serena/project.yml
   ```

3. Check Serena logs (if available) for LSP initialization errors

4. Try restarting Claude Code to reinitialize language server

### Permission Errors

**Symptom:** Serena tool calls are blocked or require repeated approval

**Solutions:**
1. If you want pre-approval, create `.claude/settings.local.json`:
   ```json
   {
     "permissions": {
       "allow": [
         "mcp__serena__*"
       ]
     }
   }
   ```

2. Use wildcard `mcp__serena__*` to approve all Serena tools at once

3. Restart Claude Code after adding settings

### Multiple Projects Interfering

**Symptom:** Working in Project A but Serena shows symbols from Project B

**Cause:** This shouldn't happen with per-project `.mcp.json`. Each Claude Code session loads only the project-scoped Serena.

**Solutions:**
1. Verify you're starting Claude Code in the correct directory:
   ```bash
   pwd
   claude .
   ```

2. Check `.mcp.json` exists in the project you're working in

3. Completely exit Claude Code and restart in the correct project directory

### Helper Script Not Executing

**Symptom:** Serena fails to start, error about bash or script execution

**Solutions:**
1. Make script executable:
   ```bash
   chmod +x .serena/start-mcp.sh
   ```

2. Test script manually:
   ```bash
   bash .serena/start-mcp.sh
   ```

3. Check for syntax errors:
   ```bash
   cat .serena/start-mcp.sh
   ```

4. Verify bash is available:
   ```bash
   which bash
   ```

## Advanced Configuration

### Monorepo Setup

For monorepos with multiple sub-projects:

**Option 1: Root-level Serena (Recommended)**
- Add `.mcp.json` and `.serena/` at monorepo root
- Serena sees entire monorepo
- Single language server for all projects

**Option 2: Per-Project Serena**
- Add `.mcp.json` and `.serena/` in each sub-project
- Isolated Serena per sub-project
- Navigate to sub-project directory to start Claude Code

### Read-Only Mode

To prevent Serena from editing files, set in `.serena/project.yml`:

```yaml
read_only: true
```

This disables:
- `mcp__serena__insert_after_symbol`
- `mcp__serena__replace_symbol_body`
- `mcp__serena__delete_symbol`

Use when:
- You want Serena only for navigation and search
- Working on sensitive production code
- Prefer manual edits only

### Excluding Specific Tools

To disable specific Serena tools, add to `.serena/project.yml`:

```yaml
excluded_tools:
  - "delete_symbol"
  - "replace_symbol_body"
```

This prevents those tools from being available, even if user has permissions.

### Custom Initial Prompt

Add context shown when Serena starts:

```yaml
initial_prompt: "This is a microservices project using Domain-Driven Design. Focus on bounded contexts and aggregates when analyzing code."
```

Useful for:
- Providing architecture context
- Guiding Serena's analysis approach
- Documenting project-specific patterns

### Custom Ignored Paths

Add paths Serena should skip:

```yaml
ignored_paths:
  - "node_modules"
  - "dist"
  - "build"
  - "*.generated.cs"
```

`.gitignore` patterns are already respected if `ignore_all_files_in_gitignore: true`.

## Web Dashboard

Serena includes a web dashboard (default: http://localhost:24282) for:
- Viewing project structure
- Monitoring MCP server status
- Browsing memory/context
- Debugging tool calls

Access it in your browser after Serena starts with Claude Code.

## Example: Adding to a C# Project

Complete walkthrough:

```bash
# 1. Navigate to project root
cd /Users/yourname/projects/my-csharp-app

# 2. Verify prerequisites
which uv                    # Should show uv path
python3 --version           # Should show 3.11.x
ls *.sln                    # Should show .sln file

# 3. Start Claude Code
claude .

# 4. In Claude Code, say:
# "Using the add-serena skill, add Serena MCP to my current project"

# 5. Answer the questions:
# - Language: C#
# - Version control: Yes, commit for team
# - Permissions: Yes, create settings.local.json

# 6. Skill creates files:
# .mcp.json
# .serena/start-mcp.sh
# .serena/project.yml
# .claude/settings.local.json
# Updates .gitignore

# 7. Exit and restart Claude Code
# (Press Ctrl+C, then run `claude .` again)

# 8. Approve project-scoped MCP server when prompted:
# Choose "Always for this project"

# 9. Verify it works:
/mcp

# 10. Test Serena:
# "Use mcp__serena__onboard to analyze the project structure"
```

## Example: Adding to a Python Project

```bash
# Navigate to project
cd /Users/yourname/projects/my-python-app

# Verify prerequisites
which uv
python3 --version
ls pyproject.toml  # or setup.py

# Start Claude Code and use the skill
claude .
# "Using add-serena, set up Serena for this Python project"

# Answer questions:
# - Language: Python
# - Version control: No, keep it local
# - Permissions: No, I'll approve interactively

# Restart Claude Code
# Exit (Ctrl+C) and restart: claude .

# Approve MCP server, then test
/mcp
# "Use mcp__serena__find_symbol to find main function"
```

## Example: Adding to Multiple Projects

Demonstrate project isolation:

```bash
# Project A: E-commerce app
cd /Users/yourname/projects/ecommerce
claude .
# "Using add-serena, add Serena to this project"
# (Answer questions, restart, verify with /mcp)

# Exit Claude Code (Ctrl+C)

# Project B: Analytics dashboard
cd /Users/yourname/projects/analytics-dashboard
claude .
# "Using add-serena, add Serena to this project"
# (Answer questions, restart, verify with /mcp)

# Both projects now have independent Serena:
# - ecommerce/.mcp.json → Serena for ecommerce
# - analytics-dashboard/.mcp.json → Serena for analytics-dashboard
# - Different .serena/.state/ folders
# - Different memories and context
# - No interference between projects
```

## Migration from Old Global Installation

If you previously followed instructions to use `claude mcp add` with a hardcoded project path:

### Step 1: Remove Global Serena (Optional)

```bash
claude mcp remove serena
```

This removes the global Serena configuration from `~/.claude.json`.

**Note:** This is optional. The per-project `.mcp.json` will take precedence over global config when you're in a project directory.

### Step 2: Add Serena to Each Project

For each project where you want Serena:

```bash
cd /path/to/project
claude .
# "Using add-serena, add Serena to this project"
```

### Step 3: Benefits of Migration

- ✅ All projects now have independent Serena
- ✅ No need to manually edit `~/.claude.json` when switching projects
- ✅ Each project maintains separate state and memory
- ✅ Configuration can be version controlled
- ✅ Team members get same setup automatically

## Summary

The **per-project `.mcp.json` with helper script approach** enables true multi-project Serena MCP support:

- **No global installation** - Each project is self-contained
- **Complete isolation** - Projects don't interfere with each other
- **Truly portable** - Helper script auto-detects project root
- **Team-friendly** - Can be version controlled for consistent team setup
- **Easy to use** - Just run the skill in any project

**Key Files Created:**
- `.mcp.json` - MCP server configuration
- `.serena/start-mcp.sh` - Portable helper script
- `.serena/project.yml` - Project-specific Serena settings
- `.claude/settings.local.json` - Optional permissions (if you chose pre-approval)

**After Setup:**
1. Restart Claude Code in project directory
2. Approve project-scoped MCP server
3. Run `/mcp` to verify
4. Use Serena tools for semantic code understanding

This approach aligns with Claude Code's official MCP architecture and provides the most reliable, maintainable, and team-friendly way to use Serena across multiple projects.

## References

- **Claude Code MCP Documentation:** https://docs.claude.com/en/docs/claude-code/mcp
- **Claude Code Project-Scoped Servers:** https://docs.claude.com/en/docs/claude-code/mcp/project-scoped-servers
- **Serena Project:** https://github.com/oraios/serena
- **uv Installation:** https://github.com/astral-sh/uv
