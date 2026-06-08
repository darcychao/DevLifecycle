# SE Design Document (System Engineering Design)

## Document Metadata
- **Design ID:** [SE-YYYY-NNN]
- **Title:** [Technical design title]
- **Author:** SE Agent
- **Date:** [YYYY-MM-DD]
- **Version:** [1.0]
- **PRD Reference:** [PRD-YYYY-NNN]
- **Status:** [Draft | In Review | Approved]

---

## 1. Design Summary
### 1.1 Approach
[High-level technical approach. One paragraph summary of how the PRD requirements will be implemented technically.]

### 1.2 Key Decisions
| Decision | Alternatives Considered | Rationale |
|----------|------------------------|-----------|
| [Decision 1] | [Alt A, Alt B] | [Why this choice] |

---

## 2. Requirement Mapping

Map every PRD requirement to a technical design element.

| PRD Requirement | Technical Module | Design Section | Implementation Complexity |
|----------------|-----------------|----------------|--------------------------|
| FR-001 | [Module name] | §3.1 | Low / Medium / High |
| FR-002 | [Module name] | §3.2 | Low / Medium / High |
| NFR-001 | [Module name] | §4.1 | Low / Medium / High |

---

## 3. Module Design

### 3.1 [Module Name]

#### 3.1.1 Purpose
[What this module does, which requirements it fulfills]

#### 3.1.2 Architecture
```
[ASCII diagram or description of internal structure]
```

#### 3.1.3 Data Model
```
Field definitions, types, constraints, relationships
```

#### 3.1.4 Interface Definition
```
Function/API signatures:
- Input parameters (name, type, required, default, validation)
- Output (type, structure, error states)
- Side effects (database writes, external calls, events)
```

#### 3.1.5 Dependencies
- **Depends on:** [List of modules/services this module needs]
- **Depended by:** [List of modules/services that need this module]

#### 3.1.6 Error Handling
| Error Condition | Handling Strategy | User-Visible? |
|----------------|-------------------|---------------|
| [Condition] | [How to handle] | Yes/No |

### 3.2 [Next Module]
[Repeat for each module]

---

## 4. Cross-Cutting Concerns

### 4.1 Performance
[How the design addresses NFR performance requirements. Caching strategy, query optimization, async processing.]

### 4.2 Security
[How the design addresses NFR security requirements. Auth flow, data encryption, input validation.]

### 4.3 Error Handling Strategy
[Global error handling approach. Logging, monitoring, user feedback.]

### 4.4 State Management
[How state flows through the system. Stateless vs stateful, session management, persistence.]

### 4.5 UI Architecture (if project has UI)

#### 4.5.1 Component Tree
```
[ASCII diagram of the component hierarchy]
App
├── Layout
│   ├── Header
│   │   ├── Navigation
│   │   └── UserMenu
│   ├── Sidebar
│   └── Content
│       └── <PageContent />
└── Footer
```

#### 4.5.2 UI State Management
[How UI state is managed — local component state, global store, URL state]
- Local state: [useState, useReducer, etc.]
- Global state: [Redux store slice, Zustand store, Context]
- URL state: [Query parameters, path parameters]

#### 4.5.3 UX Constraint Mapping
Map PRD UX constraints to specific design decisions:

| UX Constraint | Design Decision | Module |
|--------------|-----------------|--------|
| UX-01: Interaction Flow | [How the flow maps to routes/components] | [Module] |
| UX-03: Responsive Design | [Breakpoint implementation strategy] | [Module] |
| UX-04: Form Interaction | [Form validation architecture] | [Module] |
| UX-05: Feedback States | [Loading/error/empty state handling] | [Module] |

---

## 5. Data Flow

### 5.1 Primary Flow
```
[Step-by-step data flow diagram for the main happy path]
User → Component A → Service B → Database → Response
```

### 5.2 Alternative Flows
[Error flows, edge case flows, async flows]

---

## 6. Integration Points

| Integration | Type | Protocol | Data Format | Error Handling |
|------------|------|---------|-------------|----------------|
| [System/API] | [Sync/Async] | [HTTP/gRPC/MQ] | [JSON/Protobuf] | [Retry/Fallback] |

---

## 7. Impact Analysis

### 7.1 Modules Modified
| Module | Change Type | Impact Level | Risk |
|--------|------------|-------------|------|
| [Module name] | New / Modified / Deprecated | High/Med/Low | [Risk description] |

### 7.2 Migration Required
- [ ] [Migration step 1]
- [ ] [Migration step 2]

### 7.3 Backward Compatibility
[Assessment of backward compatibility. Breaking changes and mitigation.]

---

## 8. Assumptions & Open Questions

### 8.1 Assumptions
- [Assumption 1]
- [Assumption 2]

### 8.2 Open Questions
| # | Question | Asked To | Status |
|---|---------|----------|--------|
| 1 | [Question] | [Who can answer] | Open/Resolved |

---

## 9. Appendix
### 9.1 Diagrams
[Reference to architecture diagrams, sequence diagrams, ERDs]

### 9.2 References
- PRD: `docs/prd.md`
- Architecture: `docs/architecture.md`
- Module Map: `docs/module-map.md`
