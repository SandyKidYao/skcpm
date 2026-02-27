---
description: Generate documentation comments for functions, classes, and modules
---

Generate documentation comments for the selected code. Automatically detect the programming language and use its idiomatic doc comment style:

- **TypeScript/JavaScript**: JSDoc (`/** ... */`)
- **Python**: Docstring (`"""..."""`) following Google style
- **Go**: Godoc comments (`// FunctionName ...`)
- **Rust**: Rustdoc (`/// ...`)
- **Java/Kotlin**: Javadoc (`/** ... */`)
- **Other languages**: use the most common doc comment convention

Include:
- Brief description of what the function/class/module does
- `@param` / parameter descriptions with types (if not already typed)
- `@returns` / return value description
- `@throws` / exceptions that may be raised (if applicable)
- `@example` with a short usage example (if the function signature isn't self-explanatory)

If $ARGUMENTS is provided, treat it as additional context about the code's purpose.

Output only the doc comment, ready to paste above the code.
