---
description: Smart git commit — analyze changes, generate commit message matching user's style, check key docs, optionally bump version, and commit
---

# Git Commit Assistant

Help the user commit their current changes with an elegant, well-crafted commit message. Follow these steps in order:

## Step 1: Gather Context

Run these commands to understand the current state:

1. `git status` — identify all modified, added, and deleted files (never use `-uall` flag)
2. `git diff` — review unstaged changes
3. `git diff --cached` — review staged changes
4. `git log --oneline -20` — read recent commit messages to learn the user's commit style

If there are no changes at all (working tree clean), inform the user and stop.

## Step 2: Stage Changes

If there are unstaged changes, decide how to stage:
- If the user's intent is clear from $ARGUMENTS or all changes are clearly related, stage the relevant files directly (prefer `git add <specific-files>` over `git add -A`)
- If changes span multiple unrelated concerns and it's ambiguous what to include, present a numbered list of changed files and let the user pick by number or range (e.g., "1-3, 5")
- Never stage files that likely contain secrets (`.env`, credentials, tokens, private keys)

## Step 3: Analyze Commit Style

Based on the `git log` output from Step 1, determine the user's commit message style:

- **If a clear pattern exists** (e.g., Conventional Commits, emoji-prefixed, ticket-number prefixed, etc.), follow that style
- **If no clear pattern**, use this default style:
  ```
  <type>(<scope>): <short description>

  <optional body explaining why, not what>
  ```
  Where `type` is one of: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`

## Step 4: Generate Commit Message

Based on the staged changes:

1. Summarize the nature of the changes
2. Draft a commit message in the detected (or default) style
3. The subject line must be under 72 characters, imperative mood, lowercase (unless the user's style differs)
4. Include a body only for non-trivial changes — explain *why*, not *what*
5. **Do NOT ask for confirmation** — proceed directly to the next step with the generated message. Only pause to ask if you are genuinely uncertain about the intent of the changes (e.g., changes span multiple unrelated areas and the purpose is ambiguous)

## Step 5: Check Key Documentation

Before committing, inspect whether these documentation files exist and need updates based on the staged changes:

- **README.md** / **README** — Does the change affect installation steps, usage examples, API surface, or feature lists?
- **CHANGELOG.md** / **CHANGELOG** — Should a new entry be added for this change?
- **CLAUDE.md** — Do the changes affect project architecture, build commands, or conventions documented here?

For each file that needs updating:
1. Read the current content
2. Propose the specific edits and present options to the user:
   - `[1] Apply suggested edits`
   - `[2] Skip documentation update`
   - `[3] Edit manually (I'll describe what to change)`
3. Apply the approved edits and stage the updated files

If none of these files need changes, skip this step silently.

## Step 6: Version Check

Automatically detect whether the project uses version management by checking for the presence of:
- `package.json` → `version` field
- `pyproject.toml` → `version` field
- `Cargo.toml` → `version` field
- `plugin.json` / `marketplace.json` → `version` field
- Other common version files

**If no version file is found**, skip this step silently.

**If version files are found**, evaluate whether a version bump is warranted based on the nature of the staged changes:
- Documentation-only, comments, or internal refactors with no behavior change → **skip** (no bump needed)
- Bug fixes, new features, breaking changes, or public API modifications → **suggest a bump**

When suggesting a bump:
1. Show the current version and present options:
   - `[1] Patch (x.y.Z)` — for bug fixes and minor improvements
   - `[2] Minor (x.Y.0)` — for new features
   - `[3] Major (X.0.0)` — for breaking changes
   - `[4] Skip version bump`
2. Update the version file(s) based on the user's choice and stage them
3. Include the version bump in the same commit

## Step 7: Commit

1. Create the commit with the generated (or confirmed) message
2. Show the result with `git log --oneline -1` to confirm
3. Do NOT push to remote unless the user explicitly asks

## Important Rules

- Never use `git add -A` or `git add .` without user awareness of what's being staged
- Never commit files containing secrets or credentials
- Never amend existing commits unless the user explicitly asks
- Never push automatically — only commit locally
- Never skip pre-commit hooks (no `--no-verify`)
- If a pre-commit hook fails, help fix the issue and create a NEW commit
- When interacting with the user, prefer numbered options over open-ended questions
