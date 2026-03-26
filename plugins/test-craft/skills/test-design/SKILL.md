---
name: test-design
description: >-
  This skill should be used when the user asks to "design tests", "create test cases",
  "write tests for a module", "generate unit tests", "generate integration tests",
  "test this package", "design test plan", "add test coverage", "cover this module with tests",
  or wants to develop tests for a specific package or module following a requirement-driven approach.
argument-hint: "<package or module path>"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Edit", "Bash", "Agent", "AskUserQuestion"]
---

# Test Design — Requirement-Driven Test Development

Design and generate tests for a specified package or module by first understanding its business requirements and functional design, then crafting tests based on that understanding — not based on code implementation details.

## Scope Constraints

**CRITICAL — enforce these before starting:**

1. **Granularity**: The target MUST be a single package, module, or well-bounded component. If the user specifies an entire project or a very large scope, stop and ask them to narrow down to a specific package or module. Explain that overly broad scope degrades analysis quality and test effectiveness.
2. **Test type**: Focus on **functional correctness** and **performance** testing only. AI model output quality evaluation, prompt effectiveness assessment, or LLM behavior testing are explicitly out of scope. If the target module's primary purpose is AI inference/evaluation, inform the user and suggest they use specialized AI evaluation frameworks instead.

## Workflow

Execute the following phases in order. Each phase requires explicit user confirmation before proceeding to the next.

### Phase 1: Requirement Analysis

Use the `requirement-analyzer` agent to analyze the target module:

1. Provide the agent with the package/module path from `$ARGUMENTS`
2. The agent returns a structured requirements document containing:
   - Module responsibility summary
   - Core business entities and concepts
   - Functional capabilities (organized by business scenario)
   - External dependency inventory (in-project vs. out-of-project)
   - Data flow description

Present the requirements document to the user and ask for confirmation:
- Are the identified requirements complete and accurate?
- Any missing business scenarios or functional capabilities?
- Any corrections to the dependency classification?

Iterate until the user confirms the requirements are correct.

### Phase 2: Testability Assessment

After requirements are confirmed, perform a testability assessment of the module. Evaluate these dimensions:

| Dimension | What to Check |
|-----------|--------------|
| Dependency injection | Are dependencies injected via constructors/parameters, or hardcoded? |
| Interface abstraction | Are external dependencies accessed through interfaces/abstractions? |
| Function complexity | Are functions focused with single responsibilities, or monolithic? |
| Global state coupling | Does the module rely on global variables, singletons, or shared mutable state? |
| Side effect isolation | Are I/O operations and side effects isolated from business logic? |
| Configuration flexibility | Can behavior be configured for test environments? |

Produce a testability score (High / Medium / Low) for each dimension and an overall assessment. Include specific improvement suggestions for any dimension scored Low.

Present the assessment to the user for review.

### Phase 3: Test Plan Design

**IMPORTANT**: From this point forward, base all test design on the confirmed requirements and testability assessment — not on code implementation details. The goal is requirement-driven testing.

Design a comprehensive test plan with two sections:

#### 3a: Unit Test Plan

For each functional capability identified in Phase 1, design test scenarios:

- **Test objective**: What business requirement is being verified
- **Preconditions**: Required state or setup
- **Test cases**: Specific inputs, actions, and expected outcomes (include normal paths, edge cases, error paths)
- **Mock strategy**: Mock ALL external dependencies — I/O, component calls, API calls, database access, file system, network, etc.

#### 3b: Integration Test Plan

Design integration test scenarios with a narrower mock scope:

- **Mock scope**: ONLY mock other modules/packages within the same project. Do NOT mock:
  - Middleware (Redis, MQ, databases, etc.)
  - External APIs and services outside the project
  - Infrastructure components
- **Business flow identification**: Find complete business flows that can be exercised within the module boundary (with only in-project dependencies mocked). Each flow should represent a real user scenario that exercises the module's integration with its real external dependencies.
- **Test scenarios**: For each business flow:
  - Flow description and business value
  - Required infrastructure setup (database state, MQ topics, etc.)
  - Step-by-step execution plan
  - Verification points (assertions on state, output, side effects)

Present the complete test plan to the user:
- Review the full plan first
- Allow per-scenario modifications if the user has feedback
- Iterate until the plan is confirmed

### Phase 4: Output Decision

Ask the user to choose:

1. **Test document first**: Output a structured test specification document (Markdown) following the template in `references/test-plan-template.md`, then generate code upon request
2. **Generate code directly**: Proceed straight to test code generation

### Phase 5: Test Generation

#### Detect Test Framework

Before generating code, identify the project's existing test setup:
- Scan for existing test files and their patterns
- Check dependency files (go.mod, package.json, requirements.txt, pom.xml, etc.) for test framework dependencies
- Identify assertion libraries, mock frameworks, and test utilities already in use
- Follow the project's existing conventions (file naming, directory structure, test organization)

#### Generate Unit Tests

For each scenario in the unit test plan:
- Create test functions with descriptive names reflecting the business requirement
- Set up mocks for all external dependencies (consult `references/mock-strategies.md` for language-specific patterns)
- Implement test logic following the project's existing test patterns
- Include setup/teardown as needed
- Add clear comments linking each test to its business requirement

#### Generate Integration Tests

For each business flow in the integration test plan:
- Create test functions that exercise the full flow
- Set up only in-project mocks (other modules/packages in the same project)
- Include infrastructure setup/teardown helpers (database seeding, MQ setup, etc.)
- Add tags or build constraints to separate integration tests from unit tests (e.g., `//go:build integration`, `@pytest.mark.integration`, etc.)
- Document required infrastructure in test file comments

### Phase 6: Review and Iterate

After generating tests:
- Present a summary of all generated test files
- Ask the user to review and run the tests
- Offer to fix any issues, add missing scenarios, or adjust mock strategies

## Additional Resources

### Reference Files

For detailed guidance on specific aspects:
- **`references/mock-strategies.md`** — Detailed mock strategy patterns for unit vs. integration tests across languages
- **`references/test-plan-template.md`** — Complete test plan document template
