# Coding Standards

> This template should be instantiated with language-specific rules during Phase 0 initialization.

## Document Metadata
- **Language:** [Programming Language]
- **Version:** [Language version]
- **Based On:** [Industry standard reference, e.g., Google Style Guide, PEP 8, Airbnb]

---

## 1. Naming Conventions

### 1.1 Files
- **Pattern:** [kebab-case / PascalCase / snake_case]
- **Example:** `user-service.ts`, `UserService.ts`, `user_service.py`
- **Test files:** [pattern, e.g., `*.spec.ts`, `*_test.py`]
- **Config files:** [pattern]

### 1.2 Variables
- **Local variables:** [camelCase / snake_case]
- **Constants:** [UPPER_SNAKE_CASE]
- **Module-level:** [convention]
- **Boolean variables:** [is/has/should prefix convention]

### 1.3 Functions / Methods
- **Pattern:** [camelCase / snake_case]
- **Verb-led:** Functions start with a verb (get, set, create, delete, handle, validate, format, parse)
- **Examples:** `getUserById()`, `validateInput()`, `formatDate()`

### 1.4 Classes / Types / Interfaces
- **Pattern:** [PascalCase]
- **Interfaces prefix:** [I-prefix or no prefix convention]
- **Type aliases:** [convention]
- **Enums:** [PascalCase, values convention]

### 1.5 Packages / Modules / Namespaces
- **Pattern:** [convention]
- **Nesting:** [rules about nesting depth]

---

## 2. Code Structure

### 2.1 File Organization
```
1. Import statements (grouped: stdlib, third-party, internal)
2. Type/interface definitions
3. Constants
4. Module-level variables
5. Function/class definitions
6. Exports
```

### 2.2 Function Guidelines
- **Maximum length:** [N] lines (soft limit)
- **Maximum parameters:** [N] parameters (consider object parameter beyond this)
- **Single responsibility:** One function, one clear purpose
- **Pure functions preferred:** Avoid side effects where possible
- **Early returns:** Return early on error/boundary conditions

### 2.3 Class Guidelines
- **Maximum methods:** [N] (consider splitting beyond this)
- **Inheritance depth:** Maximum [N] levels
- **Composition over inheritance:** Prefer composition patterns

---

## 3. Formatting

### 3.1 Indentation
- **Style:** [Spaces / Tabs]
- **Width:** [N] spaces per level

### 3.2 Line Length
- **Maximum:** [N] characters
- **Exceptions:** Imports, URLs, generated code

### 3.3 Braces / Blocks
- **Style:** [K&R / Allman / etc.]
- **Single-statement blocks:** [always/often/sometimes/never use braces]

### 3.4 Spacing
- **Around operators:** [convention]
- **After commas:** [convention]
- **Between functions:** [convention]
- **Trailing whitespace:** [Strip / Allow]

### 3.5 Quotes
- **Strings:** [Single / Double]
- **Template literals:** [When to use]

---

## 4. Types & Type Safety

### 4.1 Type Annotations
- **Explicit vs inferred:** [When to use explicit types]
- **Return types:** [Always / Only public API]
- **Generics:** [Naming convention and constraints]

### 4.2 Null/Undefined Handling
- **Nullable types:** [Convention]
- **Optional chaining:** [When to use]
- **Null checks:** [Where required]

### 4.3 Type Casting
- **When allowed:** [Only when necessary, with justification]
- **Preferred method:** [Language-specific casting approach]

---

## 5. Error Handling

### 5.1 Error Types
- **Custom errors:** [When and how to define]
- **Error hierarchy:** [Base error class, specific subclasses]

### 5.2 Try/Catch
- **Granularity:** [How fine-grained — per operation vs per function]
- **Re-throwing:** [When to re-throw vs wrap]
- **Empty catch:** [Never — always handle or log]

### 5.3 Error Propagation
- **Async errors:** [Promise rejection, error callbacks]
- **Error logging:** [What to log, at what level]

---

## 6. Comments & Documentation

### 6.1 When to Comment
- **DO NOT comment what code does** — well-named identifiers already do that
- **DO comment why** — non-obvious constraints, bug workarounds, algorithmic choices
- **DO NOT leave commented-out code** — remove it; git history preserves it
- **DO NOT write docstrings for obvious getters/setters**

### 6.2 Comment Format
- **Single-line:** `// Brief explanation of why`
- **Multi-line:** Only for complex algorithm documentation

---

## 7. Imports & Dependencies

### 7.1 Import Order
1. Standard library imports
2. Third-party library imports
3. Internal/relative imports
4. (Blank line between each group)

### 7.2 Import Style
- **Named vs default:** [Preference]
- **Wildcard imports:** [Never / Only for specific modules]
- **Unused imports:** [Remove before commit]

---

## 8. Testing Standards

### 8.1 Test Organization
- **Location:** [Next to source / separate test directory]
- **Naming:** [describe what is tested and under what conditions]

### 8.2 Test Structure
- **Arrange-Act-Assert pattern**
- **One assertion concept per test**
- **Descriptive test names:** should[ExpectedBehavior]When[Condition]

### 8.3 Coverage Requirements
- **Critical paths:** Must be tested
- **Error handling:** Must be tested
- **Edge cases:** Should be tested
- **Coverage threshold:** [N%] minimum

---

## 9. Security Standards

### 9.1 Input Validation
- All external input must be validated at the boundary
- Use allowlists, not denylists
- Validate: type, length, format, range

### 9.2 Output Encoding
- Encode output based on context (HTML, URL, JS, SQL)
- Use framework-provided escaping mechanisms

### 9.3 Secrets Management
- Never hardcode secrets
- Use environment variables or secrets manager
- Never log secrets or sensitive data

### 9.4 Dependency Security
- Keep dependencies updated
- Review new dependencies before adding
- Run security audits regularly

---

## 10. Performance Guidelines

- Avoid premature optimization
- Profile before optimizing
- Cache expensive operations
- Be mindful of N+1 queries
- Use appropriate data structures

---

## Appendix A: Language-Specific Rules
[To be filled during Phase 0 initialization based on the detected language]

## Appendix B: Tooling Configuration
- **Linter:** [tool + config file]
- **Formatter:** [tool + config file]
- **Type checker:** [tool + config file]
