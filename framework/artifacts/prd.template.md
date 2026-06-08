# PRD (Product Requirements Document)

## Document Metadata
- **PRD ID:** [PRD-YYYY-NNN]
- **Title:** [Brief descriptive title]
- **Author:** [Requirements provider]
- **Date:** [YYYY-MM-DD]
- **Version:** [1.0]
- **Status:** [Draft | Review | Approved]

---

## 1. Overview
### 1.1 Background
[Why this feature/change is needed. Business context, user pain points, market opportunity.]

### 1.2 Scope
[What is in scope and explicitly out of scope. High-level boundaries.]

### 1.3 Goals
- [Measurable goal 1]
- [Measurable goal 2]

### 1.4 Non-Goals
- [Explicitly excluded goal 1]
- [Explicitly excluded goal 2]

---

## 2. User Stories / Use Cases

### US-001: [Story Title]
**As a** [role]
**I want** [action/feature]
**So that** [benefit/value]

**Acceptance Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2]

**Priority:** [P0-Critical | P1-High | P2-Medium | P3-Low]

### US-002: [Story Title]
[Repeat for each user story]

---

## 3. Functional Requirements

### FR-001: [Requirement Name]
- **Description:** [Detailed description]
- **Input:** [What data/events trigger this]
- **Output:** [What result/state change occurs]
- **Dependencies:** [Other FRs or systems this depends on]
- **Priority:** [P0-P3]

### FR-002: [Requirement Name]
[Repeat for each functional requirement]

---

## 4. Non-Functional Requirements

### NFR-001: Performance
- [Response time, throughput, concurrency targets]

### NFR-002: Security
- [Authentication, authorization, data protection requirements]

### NFR-003: Reliability
- [Availability, fault tolerance, recovery requirements]

### NFR-004: Compatibility
- [Browser, OS, API version compatibility]

---

## 5. Constraints & Assumptions

### 5.1 Technical Constraints
- [Technology stack limitations, infrastructure constraints]

### 5.2 Business Constraints
- [Timeline, budget, regulatory constraints]

### 5.3 Assumptions
- [Assumption 1 — what we believe to be true that affects this design]

### 5.4 UX Constraints (MANDATORY when requirement involves UI design)

> **This section is MANDATORY when the PRD involves any user-facing interface (pages, forms, dashboards, visual components, interaction flows).** If this section is absent and UI design is involved, phase transition to Phase 2 is BLOCKED per `framework/workflows/ux-constraint.rule.md`.

#### UX-01: Interaction Flow (交互流程)
- [ ] **Page navigation map:** [Describe the page navigation structure — how users move between pages/screens]
- [ ] **User operation path:** [Step-by-step user journey for key tasks]
- [ ] **Key interaction points:** [Where users click, type, scroll, drag — what triggers what]

#### UX-02: Visual Reference (视觉参考)
- [ ] **Design file link:** [Figma/Sketch/Adobe XD link, OR wireframe sketch, OR reference to existing similar UI]
- [ ] **Visual style reference:** [Color palette, typography preferences, icon style, overall visual tone]
- [ ] **If no formal design exists:** [Screenshot of similar product, hand-drawn wireframe, or written visual description]

#### UX-03: Responsive Design (响应式设计)
- [ ] **Target devices:** [Desktop / Tablet / Mobile — specify supported screen widths]
- [ ] **Breakpoints:** [e.g., Mobile: <768px, Tablet: 768-1024px, Desktop: >1024px]
- [ ] **Layout differences per device:** [How the layout adapts — stacked vs side-by-side, hidden elements, condensed navigation]

#### UX-04: Form Interaction (表单交互规范) — if forms exist
- [ ] **Validation rules per field:** [Required/optional, format constraints, character limits]
- [ ] **Error messages:** [Error text for each validation failure]
- [ ] **Submit behavior:** [What happens on submit — navigation, confirmation, inline feedback]
- [ ] **Success/failure feedback:** [How user knows operation succeeded or failed]

#### UX-05: Feedback States (状态反馈)
- [ ] **Loading state:** [What the user sees while data loads — skeleton, spinner, progress bar]
- [ ] **Empty state:** [What displays when there is no data — illustration, message, CTA]
- [ ] **Error state:** [How errors are presented — toast, inline error, error page]
- [ ] **Edge cases:** [Long content handling, concurrent operations, offline behavior]

#### UX-06: Accessibility (可访问性) — WARNING if missing
- [ ] **Keyboard navigation:** [Tab order, keyboard shortcuts, focus management]
- [ ] **Screen reader support:** [ARIA labels, alt text, semantic HTML requirements]
- [ ] **Color contrast:** [WCAG level target — AA or AAA]
- [ ] **i18n/i10n:** [Multi-language requirements, RTL support if applicable]

#### UX-07: Animation & Transitions (动效与过渡) — WARNING if missing
- [ ] **Page transitions:** [Fade, slide, none — what transition between pages]
- [ ] **Interaction feedback:** [Button press, hover effects, expand/collapse animations]
- [ ] **Reduced motion:** [Whether to respect `prefers-reduced-motion`]

---

## 6. Dependencies
| Dependency | Type | Impact | Status |
|-----------|------|--------|--------|
| [Other system/team] | [Blocking/Non-blocking] | [Description] | [Ready/Pending] |

---

## 7. Acceptance Criteria Summary
| ID | Criterion | Related Stories | Priority |
|----|-----------|----------------|----------|
| AC-001 | [Criterion] | US-001, US-002 | P0 |
| AC-002 | [Criterion] | US-003 | P1 |

---

## 8. Appendix
### 8.1 Glossary
| Term | Definition |
|------|-----------|
| [Term] | [Definition] |

### 8.2 References
- [Related documents, external specs, design mockups]
