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
| **code-review** | `/code-review:review` | Review code for bugs, security issues, performance, and best practices |
| **git-commit** | `/git-commit:commit` | Smart commit assistant: analyzes changes, generates style-matching commit messages, checks key docs, and optionally bumps version |
| **doc-generator** | `/doc-generator:doc` | Generate documentation comments for functions, classes, and modules |

### Install a plugin

```bash
/plugin install code-review@skcpm
```

### Use a plugin

```bash
# Review code
/code-review:review

# Review with focus on security
/code-review:review security

# Smart commit with message generation, doc check, and optional version bump
/git-commit:commit

# Generate docs for selected code
/doc-generator:doc
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
sk-claude-plugin-marketplace/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace catalog
├── plugins/
│   ├── code-review/          # Code review plugin
│   ├── git-commit/           # Smart git commit assistant
│   └── doc-generator/        # Documentation generator
├── .gitignore
└── README.md
```
