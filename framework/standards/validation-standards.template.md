# Process Validation Standards

> Defines how validation plugins interface with the framework. Language-specific plugins are generated during Phase 0 initialization.

## Document Metadata
- **Language:** [Programming Language]
- **Validation Plugin:** `plugins/validation-<lang>/`

---

## 1. Overview

Process validation (Phase 6) verifies that implemented code produces correct output for a given input. Validation is language-specific and implemented as a framework plugin.

## 2. Validation Interface Contract

### 2.1 Input Format
Validation input is provided as structured files. Each language plugin defines its exact format.

**Generic structure:**
```
test-data/
├── case-001/
│   ├── input.json             # Input data (standardized)
│   └── expected.json          # Expected output (standardized)
├── case-002/
│   ├── input.json
│   └── expected.json
└── manifest.json              # Validation suite metadata
```

### 2.2 Output Format
The validation plugin produces a standardized report.

```json
{
  "suite": "validation-suite-name",
  "timestamp": "ISO-8601",
  "language": "language-name",
  "results": [
    {
      "caseId": "case-001",
      "status": "pass|fail|error",
      "input": { ... },
      "expected": { ... },
      "actual": { ... },
      "diff": "description of difference (if failed)",
      "durationMs": 123
    }
  ],
  "summary": {
    "total": 10,
    "passed": 9,
    "failed": 1,
    "errors": 0,
    "durationMs": 1234
  }
}
```

### 2.3 Comparison Methods
| Method | Description | Use Case |
|--------|-------------|----------|
| Exact | Byte-for-byte match | Deterministic outputs, serialization |
| Schema | Validate structure matches schema | APIs, dynamic data |
| Tolerance | Numeric comparison within threshold | Floating point, timing |
| Predicate | Custom comparison function | Complex business logic |

---

## 3. Plugin Implementation Requirements

### 3.1 Required Files
```
plugins/validation-<lang>/
├── plugin.json                 # Manifest
├── skill.md                    # How to invoke this plugin
├── hooks/
│   └── dev-post.md             # Post-development validation hook
├── scripts/
│   ├── validate.sh             # Run all validations
│   └── generate-report.sh      # Generate report from results
└── test-data/                  # Validation test cases
    └── manifest.json
```

### 3.2 Plugin Manifest
```json
{
  "name": "validation-<lang>",
  "version": "1.0.0",
  "description": "Standardized I/O validation for <language>",
  "language": "<language>",
  "hooks": {
    "dev-post": "hooks/dev-post.md"
  },
  "scripts": {
    "validate": "scripts/validate.sh",
    "report": "scripts/generate-report.sh"
  }
}
```

### 3.3 Validation Script Contract
The `validate.sh` script:
1. Reads `test-data/manifest.json` for test case list
2. For each test case, reads `input.json`
3. Executes the code under test with the input
4. Captures actual output
5. Compares actual vs expected using the specified comparison method
6. Outputs the standardized JSON report to stdout
7. Exits 0 if all pass, non-zero if any fail

---

## 4. Test Data Authoring

### 4.1 Manifest Format
```json
{
  "suite": "feature-name-validation",
  "language": "typescript",
  "comparisonMethod": "exact",
  "cases": [
    {
      "id": "case-001",
      "description": "Valid input produces expected output",
      "inputFile": "case-001/input.json",
      "expectedFile": "case-001/expected.json",
      "comparisonMethod": "exact",
      "targetFunction": "src/services/user-service.ts:createUser"
    }
  ]
}
```

### 4.2 Input/Expected File Guidelines
- Use standard JSON format for structured data
- Input contains all parameters the function needs
- Expected contains the complete expected output (including error states)
- For stateful operations, expected may include both return value and side effects

---

## 5. Integration with Test Plan

The Test Agent maps test plan validation cases (VC-XXX) to validation plugin test cases:

| VC ID | Plugin Case | Plugin Input | Plugin Expected |
|-------|------------|-------------|----------------|
| VC-001 | case-001 | `test-data/case-001/input.json` | `test-data/case-001/expected.json` |

---

## 6. Error Handling

### 6.1 Plugin Errors
If the validation plugin cannot execute:
- Report as `status: "error"` with error message in `actual` field
- This counts as a validation failure
- Test Agent investigates and either fixes plugin or raises challenge

### 6.2 Invalid Test Data
If test data is malformed:
- Plugin reports specific parsing error
- Test Agent fixes test data (no challenge needed)
- Re-run validation

---

## 7. Language-Specific Examples

### TypeScript/JavaScript
```json
// input.json
{
  "function": "createUser",
  "args": [{ "name": "Alice", "email": "alice@example.com" }]
}
// expected.json
{
  "id": "string",  // Schema match: any string
  "name": "Alice",
  "email": "alice@example.com",
  "createdAt": "string"
}
```

### Python
```json
// input.json
{
  "module": "services.user_service",
  "function": "create_user",
  "kwargs": { "name": "Alice", "email": "alice@example.com" }
}
```

### Go
```json
// input.json
{
  "package": "userservice",
  "function": "CreateUser",
  "args": [{ "Name": "Alice", "Email": "alice@example.com" }]
}
```

---

## Appendix A: Validation Plugin Template
The framework generates a language-specific validation plugin during Phase 0 initialization based on this standard.
