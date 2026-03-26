# Test Plan Document Template

Use this template when the user chooses "test document first" in Phase 4.

---

```markdown
# Test Plan: [Module Name]

## 1. Overview

- **Module**: [module path]
- **Date**: [date]
- **Scope**: Functional and performance testing for [module name]
- **Out of Scope**: [anything explicitly excluded]

## 2. Requirements Summary

[Paste the confirmed requirements from Phase 1 — module responsibility, core entities, functional capabilities]

## 3. Testability Assessment

| Dimension | Score | Notes |
|-----------|-------|-------|
| Dependency Injection | | |
| Interface Abstraction | | |
| Function Complexity | | |
| Global State Coupling | | |
| Side Effect Isolation | | |
| Configuration Flexibility | | |

**Overall**: [High / Medium / Low]

## 4. Unit Test Plan

### 4.1 [Business Scenario Name]

#### Test Case: [TC-U-001] [Test Name]
- **Objective**: [Which requirement this verifies]
- **Preconditions**: [Required state]
- **Input**: [Test input data]
- **Action**: [What to do]
- **Expected Result**: [What should happen]
- **Mock Setup**:
  - [Dependency 1]: returns [value]
  - [Dependency 2]: returns [value]

#### Test Case: [TC-U-002] [Test Name]
...

### 4.2 [Another Business Scenario]
...

### 4.x Edge Cases and Error Paths

#### Test Case: [TC-U-E01] [Error Scenario Name]
- **Objective**: Verify error handling when [condition]
- **Input**: [Invalid or edge-case input]
- **Expected Result**: [Error response, fallback behavior, etc.]

## 5. Integration Test Plan

### 5.1 Infrastructure Requirements

| Component | Version | Setup Method |
|-----------|---------|-------------|
| [Database] | [version] | [Testcontainers / Docker / local] |
| [Cache] | [version] | [method] |
| [MQ] | [version] | [method] |

### 5.2 Business Flow Tests

#### Flow: [TC-I-001] [Business Flow Name]
- **Description**: [What business process this tests]
- **Business Value**: [Why this flow matters]
- **In-Project Mocks**:
  - [Module A]: stub [method] to return [value]
- **Real Dependencies**: [DB, Redis, MQ, etc.]
- **Steps**:
  1. [Setup] Seed database with [data]
  2. [Action] Call [entry point] with [input]
  3. [Verify] Assert [database state / MQ message / API response]
  4. [Cleanup] [cleanup actions]
- **Expected Results**:
  - [Assertion 1]
  - [Assertion 2]

#### Flow: [TC-I-002] [Another Business Flow]
...

## 6. Performance Test Considerations

### Scenarios
- [Scenario 1]: [What to measure, target metrics]
- [Scenario 2]: ...

### Benchmarks
- [Function/endpoint]: target [X] ops/sec or [Y] ms latency

## 7. Test Execution

### Unit Tests
- **Command**: [how to run unit tests]
- **Expected Duration**: [approximate]

### Integration Tests
- **Prerequisites**: [infrastructure that must be running]
- **Command**: [how to run integration tests]
- **Expected Duration**: [approximate]

## 8. Appendix

### A. Test Data
[Describe test fixtures and seed data]

### B. Environment Variables
[Required env vars for integration tests]
```
