---
description: Explore a module or feature — find all related files, map entry points, trace dependencies and data flow, and produce a structured context summary
---

# Code Explorer: Explore

Given a module name, feature keyword, file path, or rough description provided in $ARGUMENTS, systematically explore the codebase and produce a structured context summary.

If $ARGUMENTS is empty, ask the user: "What module, feature, or topic would you like to explore? (e.g. 'authentication', 'src/payments', 'UserProfile component')"

---

## Step 1: Understand the Project Layout

Before searching for anything, orient yourself in the project:

1. List the top-level directory contents to understand the high-level structure.
2. Look for and read any of these project-definition files if they exist (read only the ones present — do not error on missing files):
   - `CLAUDE.md` — project conventions, architecture notes
   - `README.md` — project overview
   - `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` — project type, dependencies, scripts
3. Identify the language(s) and primary framework(s) in use.
4. Note the directory layout convention: is source code under `src/`, `lib/`, `app/`, `packages/`, or at the root?

Carry this understanding silently into the next steps — do not summarize Step 1 to the user yet.

---

## Step 2: Locate the Topic

Using $ARGUMENTS as the search topic, find all files and symbols related to it. Work in this order:

### 2a. Direct path check
If $ARGUMENTS looks like a file path or directory path (contains `/` or a known extension), check if that path exists directly. If it does, add it to your findings and note it as the "primary entry point."

### 2b. Filename search
Search for files whose names contain the topic keyword (case-insensitive):
- Exact name matches (e.g. `auth.ts`, `auth/index.ts`, `AuthService.py`)
- Directory matches (e.g. `auth/`, `authentication/`)
- Test files that reference the topic (e.g. `auth.test.ts`, `test_auth.py`)

### 2c. Content search
Search for the topic keyword inside file contents. Look for:
- Class, function, or variable definitions that contain the keyword
- Import/export statements referencing the topic
- Configuration entries, route registrations, or dependency injection bindings
- Comments and documentation strings

### 2d. Collect and group
Collect all file paths found. Remove duplicates. Group them by category:
- **Core implementation files** — the main source files that define the feature
- **Types / interfaces / schemas** — type definitions, data models, database schemas
- **Tests** — unit tests, integration tests, end-to-end tests
- **Configuration / wiring** — route files, DI config, module registration, environment config
- **Documentation** — README sections, inline docs, API specs

If you find more than 20 files, focus on the most structurally central ones and note that the list has been truncated.

---

## Step 3: Read and Understand Core Files

For each file in the "Core implementation files" group (up to 8 files), read it fully. While reading, extract:

1. **Entry points**: exported functions, classes, HTTP route handlers, CLI commands, event listeners
2. **Key abstractions**: main classes, interfaces, types, or data structures defined here
3. **Internal logic summary**: what each major function or method does (one sentence per function)
4. **Dependencies imported**: external packages and internal modules this file imports
5. **Side effects / integrations**: database access, external API calls, event emissions, file system writes

For "Types / interfaces / schemas" files, note each exported type and its purpose.

For "Configuration / wiring" files, note how the feature is registered (routes, injected services, required env vars).

---

## Step 4: Map Dependencies

After reading the core files, build a dependency picture:

### 4a. Upstream dependencies (what this module depends on)
List all internal modules and external packages that the core files import. If a critical internal dependency has not been read yet, read it (up to 3 additional files).

### 4b. Downstream dependents (what depends on this module)
Search for files that import or reference the core files or their exported symbols. List the top consumers — you do not need to read them all.

### 4c. Data flow
Describe the high-level data flow in 3–6 steps using `→` notation:
```
Input → [Component A] → [Component B] → [Component C] → Output
```

---

## Step 5: Identify Testing Coverage

Look at the test files found in Step 2:

1. List which test files exist and which core files they cover.
2. Note any obvious gaps: core files with no corresponding test.
3. For the most important test file, read it and note:
   - What behaviors are tested?
   - What is mocked or stubbed?
   - What edge cases are covered?

---

## Step 6: Produce the Context Summary

Output a structured markdown report with this structure:

```markdown
# Code Explorer Report: <topic>

## Project Snapshot
- **Language / Framework**: ...
- **Project type**: ...
- **Source layout**: ...

## Files Found

### Core Implementation
- `path/to/file.ts` — one-line description

### Types / Interfaces / Schemas
- `path/to/types.ts` — one-line description

### Tests
- `path/to/file.test.ts` — one-line description

### Configuration / Wiring
- `path/to/router.ts` — one-line description

### Documentation
- `README.md#section` — one-line description

## Entry Points

| Entry Point | Location | Description |
|-------------|----------|-------------|
| `POST /auth/login` | `src/routes/auth.ts:23` | Authenticates user credentials |

## Key Abstractions
- **`ClassName`** (`path/to/file.ts`) — brief description

## Data Flow
```
Input → [Component A] → [Component B] → Output
```

## Dependencies

### Depends On (upstream)
| Module | Type | Purpose |
|--------|------|---------|
| `jsonwebtoken` | external | JWT signing and verification |
| `src/db/UserRepo` | internal | user persistence |

### Depended On By (downstream)
- `src/middleware/requireAuth.ts` — uses this module to validate tokens

## Testing Coverage
- Covered by: `src/__tests__/auth.test.ts`
- Gaps identified: ...
- Key behaviors tested: ...

## Observations
- Notable patterns, potential issues, or things to be aware of (3–7 bullets)

## Suggested Next Steps
- Concrete follow-up actions
- Use `/code-explorer:trace <symbol>` to dive deeper into a specific function
```

---

## Important Rules

- Never modify any files — this skill is read-only
- If a file is too large to read fully, read the first 200 lines and the last 50 lines, then note that it was partially read
- If the topic matches nothing in the codebase, say so clearly and ask the user to clarify
- Do not invent or hallucinate file paths — only report files you have actually found or read
- Do not ask clarifying questions mid-exploration — explore first, report findings, then invite questions at the end
