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

If there are unstaged changes, ask the user which files to stage:
- If the user says "all" or the intent is clear from $ARGUMENTS, stage all relevant files (prefer `git add <specific-files>` over `git add -A`)
- Otherwise, list the changed files and let the user choose
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
5. Present the proposed message to the user and ask for confirmation or edits

## Step 5: Check Key Documentation

Before committing, inspect whether these documentation files exist and need updates based on the staged changes:

- **README.md** / **README** — Does the change affect installation steps, usage examples, API surface, or feature lists?
- **CHANGELOG.md** / **CHANGELOG** — Should a new entry be added for this change?
- **CLAUDE.md** — Do the changes affect project architecture, build commands, or conventions documented here?

For each file that needs updating:
1. Read the current content
2. Explain what should change and why
3. **Ask the user for confirmation before making any edits**
4. Apply the approved edits
5. Stage the updated documentation files

If none of these files need changes, skip this step silently.

## Step 6: Version Bump (Optional)

Ask the user: "Do you want to bump the version number?"

If yes:
1. Detect the project's version management approach:
   - `package.json` → `version` field
   - `pyproject.toml` → `version` field
   - `Cargo.toml` → `version` field
   - `plugin.json` / `marketplace.json` → `version` field
   - Other version files as appropriate
2. Show the current version and suggest the next version based on the change type:
   - Breaking changes → major bump
   - New features → minor bump
   - Bug fixes / patches → patch bump
3. Let the user confirm or specify the desired version
4. Update the version file(s) and stage them
5. Include the version bump in the commit, or create a separate version bump commit if the user prefers

If the user declines, skip this step.

## Step 7: Commit

1. Create the commit with the confirmed message
2. Show the result with `git log --oneline -1` to confirm
3. Do NOT push to remote unless the user explicitly asks

## Important Rules

- Never use `git add -A` or `git add .` without user awareness of what's being staged
- Never commit files containing secrets or credentials
- Never amend existing commits unless the user explicitly asks
- Never push automatically — only commit locally
- Never skip pre-commit hooks (no `--no-verify`)
- If a pre-commit hook fails, help fix the issue and create a NEW commit
- Always get user confirmation on the commit message before committing
