---
description: Trace a specific function, class, or symbol — follow its call chain, data flow, and usages across the codebase
---

# Code Explorer: Trace

Given a symbol name (function, class, method, constant, or type) provided in $ARGUMENTS, trace it through the codebase: find its definition, understand its behavior, follow every call site and usage, and map the full call chain.

If $ARGUMENTS is empty, ask the user: "What symbol would you like to trace? (e.g. 'AuthService.validateToken', 'processPayment', 'UserSchema')"

If $ARGUMENTS contains a dot (e.g. `AuthService.validateToken`), treat the part before the dot as the class/module name and the part after as the method name, and search accordingly.

---

## Step 1: Find the Definition

Search the codebase for the definition of $ARGUMENTS:
- For a function: look for `function <name>`, `const <name> =`, `def <name>`, `func <name>`, arrow functions assigned to `<name>`, class methods named `<name>`
- For a class: look for `class <name>`
- For a type/interface: look for `type <name>`, `interface <name>`, `struct <name>`
- For a constant: look for `const <name>`, `let <name>`, `var <name>`

If multiple definitions are found in different modules, list all candidates and let the user confirm which to trace. If the intent is clear from context, proceed with the most relevant one.

Read the file containing the definition. Note:
- Exact file path and line number of the definition
- The full signature (parameters, return type)
- What the function/class does
- What it imports and calls internally

---

## Step 2: Trace Internal Calls (Depth-First, Max 3 Levels)

Starting from the definition, trace the functions and methods it calls internally:

**Level 1**: List every function/method called directly inside $ARGUMENTS. For each:
- Note the call site (file and line number)
- If the called function is internal to the project (not an external library), read its definition
- Summarize what it does in one sentence

**Level 2**: For each Level 1 internal call, list the functions it in turn calls. Read definitions of internal ones. Summarize in one sentence each.

**Level 3**: For each Level 2 internal call, note what it calls but do not read further unless the user asks.

If a call goes to an external library, note the package name and method being called, but do not trace further into it.

---

## Step 3: Find All Call Sites (Who Calls This Symbol?)

Search the codebase for every place that references $ARGUMENTS:

1. Search for the symbol used in call position: `<name>(`, `.<name>(`, `new <name>(`
2. Search for import statements that import $ARGUMENTS
3. Group call sites by file
4. For each file that calls $ARGUMENTS, note:
   - The file path and approximate line numbers
   - The context: why is it being called here? (read 5–10 lines around each call site)

If there are more than 15 call sites, group them by pattern (e.g. "12 route handlers," "3 middleware files," "2 test files") and read only the most structurally important ones.

---

## Step 4: Trace Data Flow Through the Symbol

Describe how data flows through $ARGUMENTS:

1. **Inputs**: parameters, closed-over variables, globals read, context injected
2. **Transformations**: what does the symbol do to the data? (compute, validate, persist, fetch, format)
3. **Outputs**: return value, side effects (database writes, API calls, events emitted, file writes, mutations)
4. **Error paths**: what happens when something goes wrong? Are errors thrown, returned, logged, or silently swallowed?

---

## Step 5: Identify Related Tests

Search for test files that:
- Import or reference $ARGUMENTS directly
- Test the behavior of $ARGUMENTS indirectly (e.g. integration tests for routes that use it)

For the most relevant test file, read it and note:
- What scenarios are tested?
- What is mocked at the boundary?
- What is NOT tested (visible gaps)?

---

## Step 6: Produce the Trace Report

Output a structured markdown report with this structure:

```markdown
# Code Explorer Trace: <symbol>

## Definition
- **File**: `path/to/file.ts`
- **Line**: 42
- **Signature**: `function validateToken(token: string, secret: string): DecodedToken | null`
- **Summary**: One-sentence description of what this symbol does.

## Internal Call Chain

### Level 1 — Direct calls made by `<symbol>`
- `jwt.verify()` (external: `jsonwebtoken`) — verifies token signature and expiry
- `normalizePayload()` (`src/utils/jwt.ts:18`) — strips internal fields from decoded payload

### Level 2 — Calls made by Level 1 internals
- `normalizePayload` calls:
  - `omit()` (external: `lodash`) — removes specified keys from an object

### Level 3 — Summary only
(No further internal calls beyond Level 2)

## Call Sites (Who Uses This Symbol?)

| File | Lines | Context |
|------|-------|---------|
| `src/middleware/requireAuth.ts` | 34 | Validates bearer token on every protected route |
| `src/__tests__/auth.test.ts` | 12, 45, 67 | Unit tests for valid, expired, and malformed tokens |

## Data Flow

```
Input: raw JWT string (from Authorization header)
  → jwt.verify() — cryptographic signature check + expiry check
  → normalizePayload() — strips `iat`, `exp`, `iss` fields
Output: DecodedToken { userId, roles } or null

Error paths:
  - Invalid signature → caught, returns null
  - Expired token → caught, returns null
```

## Testing Coverage
- Directly tested in: `src/__tests__/auth.test.ts`
- Scenarios covered: valid token, expired token, wrong secret, malformed string
- Gaps: no test for token with valid signature but missing required payload fields

## Observations
- Notable patterns, potential issues, or design notes (3–6 bullets)

## Suggested Next Steps
- Concrete suggestions for the user's next task
- Use `/code-explorer:explore <module>` to see the broader module context
```

---

## Important Rules

- Never modify any files — this skill is read-only
- Only report actual file paths and line numbers you have verified by reading the file
- Do not hallucinate code contents — if you cannot find a definition, say so clearly
- If the symbol is very widely used (>50 call sites), report the most architecturally significant ones and note the total count
- Stop recursive tracing at Level 3 or when all further calls are external library calls
