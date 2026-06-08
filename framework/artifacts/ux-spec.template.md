# UX Requirements Specification (UX需求级规范)

## Document Metadata
- **UX Spec ID:** [UX-YYYY-NNN]
- **Title:** [UX specification title — matching requirement]
- **Author:** UX Agent (extracted from user-provided design artifacts)
- **Date:** [YYYY-MM-DD]
- **Version:** [1.0]
- **PRD Reference:** [PRD-YYYY-NNN]
- **SE Design Reference:** [SE-YYYY-NNN]
- **Design Source:** [Figma/Sketch link, wireframe, or style reference from PRD §5.4 UX-02]
- **Status:** [Draft | In Review | Approved]

> **Important:** This document contains UX requirements EXTRACTED from existing design artifacts. The UX Agent does NOT create or design — it only extracts, organizes, and formalizes UX requirements that already exist in PRD §5.4, SE Design §4.5, and user-provided design files. Every requirement in this document is traceable to a source. If a requirement cannot be traced to a source, it is marked as a GAP.

---

## 1. UX Requirements Inventory

### 1.1 Source Traceability Matrix

| Spec Item ID | Requirement Summary | Source Type | Source Reference | Status |
|-------------|--------------------|-------------|-----------------|--------|
| UX-S-001 | [Requirement] | PRD | §5.4 UX-01 | Extracted |
| UX-S-002 | [Requirement] | PRD | §5.4 UX-02 | Extracted |
| UX-S-003 | [Requirement] | Design File | [Figma frame / wireframe page] | Extracted |
| UX-S-004 | [Requirement] | SE Design | §4.5.1 | Extracted |
| UX-S-005 | [Requirement] | — | — | **GAP — needs user input** |

### 1.2 PRD UX Constraint Coverage

| PRD Constraint | Extracted Spec Items | Coverage |
|---------------|---------------------|----------|
| UX-01: Interaction Flow | [UX-S-IDs] | Complete / Partial / GAP |
| UX-02: Visual Reference | [UX-S-IDs] | Complete / Partial / GAP |
| UX-03: Responsive Design | [UX-S-IDs] | Complete / Partial / GAP |
| UX-04: Form Interaction | [UX-S-IDs] | Complete / Partial / GAP |
| UX-05: Feedback States | [UX-S-IDs] | Complete / Partial / GAP |
| UX-06: Accessibility | [UX-S-IDs] | Complete / Partial / GAP |
| UX-07: Animation | [UX-S-IDs] | Complete / Partial / GAP |

---

## 2. Component UX Requirements (extracted from design sources)

### 2.1 Component Inventory (from SE Design §4.5.1)

| Component | Type | Page/Context | UX Requirements Extracted | Primary Source |
|-----------|------|-------------|--------------------------|---------------|
| [Component] | [Page/Layout/UI] | [Location] | [N items] | [Source ref] |

### 2.2 [Component Name]

- **SE Design reference:** §[X.Y]
- **Design source:** [Figma frame / wireframe / PRD description]

#### Visual Requirements (as observed in design source)

| State | Appearance (extracted from source) | Source Reference | Specified? |
|-------|-----------------------------------|-----------------|------------|
| **Default** | [Colors, borders, shadow, typography, icon, spacing] | [Source] | Yes / GAP |
| **Hover** | [Visual change] | [Source] | Yes / GAP |
| **Active/Pressed** | [Visual change] | [Source] | Yes / GAP |
| **Focus** | [Focus style] | [Source] | Yes / GAP |
| **Disabled** | [Muted treatment] | [Source] | Yes / GAP |
| **Loading** | [Spinner/skeleton] | [Source] | Yes / GAP |
| **Error** | [Error treatment] | [Source] | Yes / GAP |

*States not shown in design source → marked GAP → flag to user*

#### Size Variants (as observed in design source)

| Variant | Dimensions (from design) | Source |
|---------|--------------------------|--------|
| — | — | — |

*Only list variants present in design source*

#### Design Token Mapping

| Visual Property | Value in Design | Mapped Project Token | Match Quality |
|----------------|-----------------|---------------------|---------------|
| [Property] | [Design value] | [Token = value] | Exact / Δ[X] / No token |

### 2.3 [Next Component]
[Repeat from SE Design §4.5.1 component inventory]

---

## 3. Interaction Requirements (extracted from PRD UX-01 and design sources)

### 3.1 Page Navigation Flow (as described in PRD UX-01)

```
[Extracted from PRD §5.4 UX-01 — do not invent]
```

*If no flow is provided in PRD, mark: "GAP — user has not provided interaction flow"*

### 3.2 Per-Component Interactions (as observed in design sources)

| Component | Interaction Type | Expected Behavior | Source |
|-----------|-----------------|-------------------|--------|
| [Component] | Click/Tap | [What happens] | [Source] |
| [Component] | Hover | [Visual feedback] | [Source] |

### 3.3 Keyboard Requirements (from PRD UX-06 if specified)

| Shortcut | Scope | Action | Source |
|----------|-------|--------|--------|
| — | — | — | — |

*If no keyboard requirements specified: "Not specified by user — project defaults apply"*

---

## 4. Responsive Requirements (extracted from PRD UX-03 and project §0.6)

### 4.1 Breakpoint Definitions (from project §0.6)

| Breakpoint | Width Range | Source |
|-----------|------------|--------|
| [Mobile] | [range] | §0.6 |

### 4.2 Per-Page/Component Responsive Behavior (from PRD UX-03)

| Page/Component | Mobile | Tablet | Desktop | Source |
|---------------|--------|--------|---------|--------|
| [Element] | [Behavior] | [Behavior] | [Behavior] | PRD §5.4 UX-03 |

*If no responsive behavior specified: "GAP — user has not specified responsive requirements"*

---

## 5. Accessibility Requirements (extracted from PRD UX-06 and §0.6)

| Requirement | Specified? | Detail | Source |
|------------|-----------|--------|--------|
| WCAG target | Yes / GAP | [Level] | [§0.6 / PRD UX-06] |
| Keyboard navigation | Yes / GAP | [Details] | [Source] |
| Screen reader support | Yes / GAP | [Details] | [Source] |
| Color contrast | Yes / GAP | [Details] | [Source] |
| Focus management | Yes / GAP | [Details] | [Source] |
| Semantic HTML | Yes / GAP | [Details] | [Source] |
| i18n/i10n | Yes / GAP | [Languages] | [Source] |

---

## 6. Design System Mapping (extracted from design source + §0.6)

### 6.1 Color Mapping

| Semantic Color | Design Hex | Project Token | Token Value | Δ |
|---------------|-----------|---------------|-------------|---|
| [Name] | [#hex] | [token] | [value] | [diff] |

### 6.2 Typography Mapping

| Text Role | Design Spec | Project Token | Token Spec | Δ |
|----------|------------|---------------|------------|---|
| [Role] | [size/weight/line] | [token] | [spec] | [diff] |

### 6.3 Spacing Mapping

| Element | Design Value | Project Token | Token Value | Δ |
|---------|-------------|---------------|-------------|---|
| [Element:property] | [px] | [token] | [px] | [diff] |

---

## 7. UX Specification Gaps

### 7.1 Blocking Gaps (must be resolved before Phase 3 → Story Design)

| Gap ID | Missing Item | Affected Components | What User Must Provide |
|--------|-------------|-------------------|-----------------------|
| GAP-001 | [Description] | [Components] | [Specific input needed] |

### 7.2 Non-Blocking Gaps (can proceed with project defaults from §0.6)

| Gap ID | Missing Item | Default Applied (from §0.6) |
|--------|-------------|---------------------------|
| GAP-002 | [Description] | [Default rule] |

---

## 8. UX Specification Traceability Summary

| Source | Items Extracted | Items Gapped | Coverage % |
|--------|----------------|-------------|------------|
| PRD §5.4 UX Constraints | [N] | [N] | [%] |
| SE Design §4.5 UI Architecture | [N] | [N] | [%] |
| Design files (UX-02 reference) | [N] | [N] | [%] |
| `docs/ux-guidelines.md` / §0.6 | [N] | [N] | [%] |
| **Total** | **[N]** | **[N]** | **[%]** |

---

## 9. Appendix

### 9.1 Design Source References
- [Figma/Sketch link from PRD §5.4 UX-02]
- [Wireframe/screenshot references]
- [Style guide references]

### 9.2 Extraction Notes
- [Any assumptions made during extraction from design sources]
- [Ambiguities encountered and how they were resolved / escalated]

### 9.3 References
- PRD: `docs/requirements/REQ-YYYY-NNN-{slug}/prd.md` §5.4
- SE Design: `docs/requirements/REQ-YYYY-NNN-{slug}/se-design.md` §4.5
- UX Guidelines: `docs/ux-guidelines.md`
- Coding Standards §0.6: `docs/coding-standards.md`
