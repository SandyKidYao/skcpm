---
description: Review code for bugs, security, performance, and best practices
---

Review the selected code or recent changes. Focus on:

1. **Bugs & Edge Cases** — Logic errors, off-by-one, null/undefined handling, race conditions
2. **Security** — Injection vulnerabilities, hardcoded secrets, unsafe deserialization, OWASP top 10
3. **Performance** — Unnecessary allocations, N+1 queries, missing memoization, blocking operations
4. **Readability** — Naming clarity, function length, dead code, unclear control flow

For each issue found:
- State the severity (critical / warning / suggestion)
- Quote the relevant code
- Explain *why* it's a problem
- Suggest a concrete fix

If $ARGUMENTS is provided, focus the review on that specific aspect (e.g., "security", "performance").

Be concise and actionable. Skip generic praise — only report findings.
