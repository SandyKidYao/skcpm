# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SKCPM — a monorepo-style Claude Code plugin marketplace. All plugins live under `plugins/` and are referenced via relative paths in `.claude-plugin/marketplace.json`.

## Validation

```bash
claude plugin validate .
```

## Architecture

- `.claude-plugin/marketplace.json` — marketplace catalog listing all plugins, their sources, categories, and tags
- `plugins/<name>/` — each plugin is a self-contained directory with its own manifest and skills

### Plugin structure

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json        # name, description, version, author
└── skills/
    └── <skill-name>/
        └── SKILL.md        # frontmatter (description) + instructions
```

### Conventions

- Plugin and skill names use **kebab-case**
- SKILL.md frontmatter requires a `description` field
- Use `$ARGUMENTS` in SKILL.md to accept user input
- When adding a new plugin, register it in `.claude-plugin/marketplace.json` under the `plugins` array with `name`, `source` (`./plugins/<name>`), `description`, `version`, `category`, and `tags`
- Marketplace version is in `metadata.version`; each plugin tracks its own version in `plugin.json`
