# Test Plan Document

## Document Metadata
- **Test Plan ID:** [TEST-YYYY-NNN]
- **Title:** [Test plan title]
- **Author:** Test Agent
- **Date:** [YYYY-MM-DD]
- **Version:** [1.0]
- **PRD Reference:** [PRD-YYYY-NNN]
- **Status:** [Draft | In Review | Approved]

---

## 1. Test Strategy

### 1.1 Scope
[What is being tested and what is explicitly not being tested.]

### 1.2 Test Levels
| Level | Description | Owner | Environment |
|-------|-------------|-------|-------------|
| Unit | Individual functions/components | Dev Agent | Local |
| Integration | Module interactions | Test Agent | Staging |
| E2E | Full user workflows | Test Agent | Staging |
| Regression | Existing functionality | Test Agent | Staging |

### 1.3 Test Environment
- **OS:** [Windows/Linux/macOS version]
- **Runtime:** [Node/Python/Java version]
- **Dependencies:** [Database, external services, mock services]

---

## 2. Acceptance Criteria

Map each PRD requirement to verifiable acceptance criteria.

| AC ID | PRD Reference | Criterion | Pass Condition | Priority |
|-------|--------------|-----------|---------------|----------|
| AC-001 | FR-001 | [Description] | [Measurable pass condition] | P0 |
| AC-002 | US-001 | [Description] | [Measurable pass condition] | P1 |

---

## 3. Test Cases

### TC-001: [Test Case Title]
- **AC Reference:** AC-001
- **Level:** [Unit / Integration / E2E]
- **Priority:** [P0 / P1 / P2]
- **Preconditions:**
  - [State that must exist before test runs]
  - [Data that must be in the system]
- **Test Data:**
  ```
  Input: { "field": "value" }
  ```
- **Test Steps:**
  1. [Step 1 — specific action]
  2. [Step 2 — specific action]
  3. [Step 3 — verify result]
- **Expected Result:**
  ```
  Output: { "result": "expected" }
  Status Code: 200
  ```
- **Cleanup:**
  - [Steps to restore state after test]

### TC-002: [Test Case Title — Error Path]
- **AC Reference:** AC-001
- **Level:** Unit
- **Priority:** P1
- **Preconditions:**
  - [Invalid state setup]
- **Test Data:**
  ```
  Input: null / empty / malformed
  ```
- **Test Steps:**
  1. Send invalid input
  2. Verify error response
- **Expected Result:**
  ```
  Error: { "code": "VALIDATION_ERROR", "message": "..." }
  Status Code: 400
  ```

### TC-003: [Test Case Title — Edge Case]
[Repeat structure for each test case]

---

## 4. Test Case Matrix

| Test Case | FR-001 | FR-002 | FR-003 | US-001 | US-002 | NFR-001 |
|-----------|--------|--------|--------|--------|--------|---------|
| TC-001 | ✓ | - | - | ✓ | - | - |
| TC-002 | ✓ | - | - | - | - | - |
| TC-003 | - | ✓ | ✓ | - | ✓ | - |
| TC-004 | - | - | - | - | - | ✓ |
| **Coverage** | 2/2 | 1/1 | 1/1 | 1/2 | 1/1 | 1/1 |

---

## 5. Validation Process

### 5.1 Validation Plugin
- **Plugin:** `plugins/validation-<lang>/`
- **Input Format:** [Standardized input per validation standards]
- **Output Format:** [Standardized output per validation standards]
- **Comparison Method:** [Exact match / Schema match / Tolerance range]

### 5.2 Validation Cases
| VC ID | Input File | Expected Output File | Tolerance |
|-------|-----------|---------------------|-----------|
| VC-001 | `test-data/input-001.json` | `test-data/expected-001.json` | Exact |
| VC-002 | `test-data/input-002.json` | `test-data/expected-002.json` | Schema |

---

## 6. Regression Test Suite

| Test Case | Covers | Last Run | Status |
|-----------|--------|---------|--------|
| REG-001 | [Existing feature] | N/A | Pending |
| REG-002 | [Existing feature] | N/A | Pending |

---

## 7. Test Execution Schedule

| Phase | Test Cases | Owner | Expected Duration | Dependencies |
|-------|-----------|-------|-------------------|--------------|
| 1. Unit | TC-001, TC-002 | Dev Agent | [Estimate] | Code complete |
| 2. Integration | TC-003, TC-004 | Test Agent | [Estimate] | Unit tests pass |
| 3. Validation | VC-001, VC-002 | Test Agent | [Estimate] | Integration pass |

---

## 8. Appendix
### 8.1 References
- PRD: `docs/prd.md`
- SE Design: `docs/se-design.md`
- Validation Standards: `framework/standards/validation-standards.template.md`
