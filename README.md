# SKCPM: SandyKid's Claude Plugin Marketplace

A collection of claude code plugins used by SandyKid

## Installation

Add this marketplace to Claude Code:

```bash
# From GitHub
/plugin marketplace add SandyKidYao/skcpm

# Or from a local clone
/plugin marketplace add ./path/to/skcpm
```

## Available Plugins

| Plugin | Command | Description |
|--------|---------|-------------|
| **git-commit** | `/git-commit:commit` | Smart commit assistant: analyzes changes, generates style-matching commit messages, checks key docs, and optionally bumps version |
| **code-explorer** | `/code-explorer:explore` | Breadth-first scan of a module or feature — files, entry points, dependencies, data flow, and test coverage |
| **code-explorer** | `/code-explorer:trace` | Depth-first call-chain trace for a specific function, class, or symbol |
| **doc-sync** | `/doc-sync:check` | Check documentation against code and output a diff report (read-only) |
| **doc-sync** | `/doc-sync:sync` | Check documentation against code and update outdated docs automatically |
| **test-craft** | `/test-craft:test-design` | Requirement-driven test design and generation for a package or module (unit tests + integration tests) |
| **test-craft** | `/test-craft:testability` | Assess a module's testability and provide improvement recommendations |

### Install a plugin

```bash
/plugin install code-review@skcpm
```

### Use a plugin

```bash
# Smart commit with message generation, doc check, and optional version bump
/git-commit:commit

# Explore a module or feature (breadth-first)
/code-explorer:explore authentication

# Trace a specific symbol (depth-first)
/code-explorer:trace AuthService.validateToken

# Check docs against code (read-only report)
/doc-sync:check

# Update outdated docs based on code
/doc-sync:sync README.md

# Design and generate tests for a module (requirement-driven)
/test-craft:test-design pkg/order

# Assess module testability
/test-craft:testability pkg/auth
```

## Adding a New Plugin

1. Create a directory under `plugins/`:

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```

2. Add `plugin.json`:

```json
{
  "name": "my-plugin",
  "description": "What your plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

3. Create your `SKILL.md` with frontmatter:

```markdown
---
description: Brief description of what this skill does
---

Instructions for Claude when this skill is invoked.
```

4. Register in `.claude-plugin/marketplace.json`:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "description": "What your plugin does",
  "version": "1.0.0",
  "category": "your-category",
  "tags": ["relevant", "tags"]
}
```

5. Validate:

```bash
claude plugin validate .
```

## Project Structure

```
skcpm/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace catalog
├── plugins/
│   ├── git-commit/           # Smart git commit assistant
│   ├── code-explorer/        # Codebase exploration and symbol tracing
│   ├── doc-sync/             # Documentation-code sync checker and updater
│   └── test-craft/           # Requirement-driven test design and generation
├── CLAUDE.md
├── .gitignore
└── README.md
```
