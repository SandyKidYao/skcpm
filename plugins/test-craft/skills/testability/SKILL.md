---
name: testability
description: >-
  This skill should be used when the user asks to "assess testability", "check testability",
  "evaluate code testability", "is this module testable", "testability review",
  "how testable is this code", or wants to understand how easy a module or package is to test
  and what improvements would make it more testable.
argument-hint: "<package or module path>"
allowed-tools: ["Read", "Grep", "Glob", "Agent"]
---

# Testability Assessment

Evaluate the testability of a specified package or module and provide actionable improvement recommendations.

## Scope Constraints

The target MUST be a single package, module, or well-bounded component. If the user specifies an overly broad scope (e.g., an entire project), ask them to narrow it down. Testability assessment is most useful at the package/module level.

AI-related modules focused on model inference or prompt engineering are out of scope for this assessment — their "testability" involves different concerns (evaluation frameworks, benchmark suites) that this skill does not address.

## Assessment Process

### Step 1: Analyze Module Structure

Read the target module to understand:
- Public API surface (exported functions, classes, methods)
- Constructor patterns and dependency injection
- Use of interfaces/abstractions vs. concrete types
- Internal vs. external dependency boundaries

### Step 2: Evaluate Dimensions

Assess each dimension on a three-level scale: **High** (easy to test), **Medium** (testable with effort), **Low** (significant barriers).

| Dimension | High | Medium | Low |
|-----------|------|--------|-----|
| **Dependency Injection** | All dependencies injected via constructors or parameters | Some injected, some created internally | Dependencies hardcoded or created inline |
| **Interface Abstraction** | External dependencies accessed through interfaces | Partial use of interfaces | Direct dependency on concrete implementations |
| **Function Complexity** | Functions are focused, single-responsibility | Mixed — some focused, some doing too much | Monolithic functions with many responsibilities |
| **Global State Coupling** | No global state; all state is local or injected | Limited global state usage | Heavy reliance on globals, singletons, or shared mutable state |
| **Side Effect Isolation** | I/O and side effects are clearly separated from business logic | Partial separation | Business logic and side effects are intertwined |
| **Configuration Flexibility** | Behavior easily configurable for test environments | Some configurability | Hardcoded configuration, environment-dependent behavior |

### Step 3: Identify Specific Issues

For each dimension scored Medium or Low, identify concrete code locations and patterns causing the issue. Reference specific files and line numbers.

### Step 4: Provide Recommendations

For each issue, suggest a specific refactoring approach:
- What to change and why
- Expected effort (small / medium / large refactor)
- Priority (how much it would improve testability)

### Step 5: Generate Report

Present findings in this format:

```
## Testability Assessment: [Module Name]

### Overall Score: [High / Medium / Low]

### Dimension Scores

| Dimension | Score | Key Finding |
|-----------|-------|-------------|
| Dependency Injection | ... | ... |
| Interface Abstraction | ... | ... |
| Function Complexity | ... | ... |
| Global State Coupling | ... | ... |
| Side Effect Isolation | ... | ... |
| Configuration Flexibility | ... | ... |

### Issues and Recommendations

#### Issue 1: [Description]
- **Location**: [file:line]
- **Impact**: [How it affects testability]
- **Recommendation**: [Specific refactoring suggestion]
- **Effort**: [small / medium / large]

[... more issues ...]

### Summary
[2-3 sentence summary of overall testability and top-priority improvements]
```
