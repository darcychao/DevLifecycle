# Standard Development Lifecycle

## Overview
The standard lifecycle defines the sequential phases of AI-agent-driven software development. Each phase has defined inputs, outputs, responsible agent, and quality gates. The lifecycle supports two entry paths: full lifecycle (PRD → 7 phases) and Dev-Story shortcut (Dev Story → development-focused phases).

## Entry Paths

### Full Lifecycle Entry
User provides a PRD. All 7 phases execute in sequence.

### Dev-Story Shortcut Entry
User provides a pre-existing Dev Story document. Upstream phases (PRD, SE Design, Story Design) are bypassed. The lifecycle enters at Phase 4 (Test Plan) or Phase 5 (Dev Coding), depending on which artifacts already exist.

**Shortcut detection:**
The orchestrator detects `docs/dev-story.md` exists and was user-provided (not framework-generated).

**Artifact evaluation:**

| Artifacts Available | Entry Phase | Next Steps |
|--------------------|-------------|------------|
| Dev Story only | Phase 4 (Test Plan) | Test Agent generates test plan from Dev Story → Phase 5 → 6 → 7 |
| Dev Story + Test Plan | Phase 5 (Dev Coding) | Dev Agent codes → Phase 6 → 7 |
| Dev Story + Test Plan + Code | Phase 6 (Code Review) | SE Agent reviews → Phase 7 |

**Shortcut rules:**
- When PRD is absent, Test Agent derives test cases from Dev Story implementation details
- When SE Design is absent, SE Agent uses Dev Story as the design reference for code review
- The Dev Story itself can be challenged by any agent if it contains contradictions or impossibilities
- Missing upstream artifacts are replaced with placeholder docs that reference the Dev Story as source of truth

## Lifecycle Phases (Full Path)

```
Phase 1       Phase 2      Phase 3       Phase 4      Phase 5      Phase 6        Phase 7
┌──────┐     ┌──────┐     ┌───────┐     ┌──────┐     ┌──────┐     ┌────────┐     ┌──────────┐
│ PRD  │ ──▶ │  SE  │ ──▶ │ Story │ ──▶ │ Test │ ──▶ │ Dev  │ ──▶ │ Code   │ ──▶ │Validation│
│Input │     │Design│     │Design │     │ Plan │     │Coding│     │Review  │     │ (Plugin) │
└──────┘     └──────┘     └───────┘     └──────┘     └──────┘     └────────┘     └──────────┘
    │            │            │             │            │             │                │
    ▼            ▼            ▼             ▼            ▼             ▼                ▼
  PRD.md     SE Design   Dev Story     Test Plan    Source Code   Review Report   Pass/Fail
  (User)     (SE Agent)  (Dev Agent)   (Test Agent) (Dev Agent)   (SE Agent)     (Plugin)
```

## Phase Details

### Phase 1: PRD (Product Requirements Document)
- **Provider:** User / Requirements provider
- **Output:** `docs/prd.md`
- **Content:** Core requirements, user stories, business logic, constraints
- **Format:** Follow `framework/artifacts/prd.template.md`
- **Gate:** PRD must be complete enough for SE Agent to design architecture

### Phase 2: SE Design (System Engineering Design)
- **Agent:** SE Agent (`framework/agents/se-agent.skill.md`)
- **Input:** `docs/prd.md`, `docs/architecture.md`, `docs/module-map.md`
- **Output:** `docs/se-design.md`
- **Format:** Follow `framework/artifacts/se-design.template.md`
- **Reviewer:** Dev Agent
- **Gate:** Dev Agent confirms design is complete and implementable

### Phase 2.6: UX Specification Extraction (UX需求规范提取) — CONDITIONAL

**Trigger:** This phase executes ONLY when ALL of the following are true:
1. `docs/.framework-init.lock` → `ui_stack.has_ui: true`
2. The current requirement involves UI design (PRD §5.4 is populated)

**Skip condition:** If the requirement has no UI component (backend-only, API-only), this phase is skipped entirely. Phase transition goes from Phase 2 directly to Phase 3.

**Principle:** The UX Agent does NOT design — it EXTRACTS and ORGANIZES UX requirements from existing design artifacts (PRD §5.4, SE Design §4.5, user-provided design files). Every requirement in the UX Specification is traceable to a source. Gaps are flagged as CAT-1 challenges, not filled by the UX Agent.

- **Agent:** UX Agent (`framework/agents/ux-agent.skill.md`)
- **Input:** `docs/prd.md` (specifically §5.4 UX Constraints), `docs/se-design.md` (specifically §4.5 UI Architecture), user-provided design files (cited in UX-02), `docs/ux-guidelines.md`, `docs/coding-standards.md` §0.6
- **Output:** `docs/requirements/REQ-YYYY-NNN-{slug}/ux-spec.md`
- **Report:** `docs/requirements/REQ-YYYY-NNN-{slug}/reports/phase-2.6-ux-spec-report.md`
- **Format:** Follow `framework/artifacts/ux-spec.template.md`
- **Reviewer:** Dev Agent (implementability check)
- **Gate:** Dev Agent confirms extracted requirements are specific enough to implement; all PRD UX constraints (UX-01 through UX-05) have corresponding extracted spec items; SE Design §4.5 component tree is covered; gaps are explicitly documented

#### UX Specification Extraction Flow

```
Phase 2 (SE Design) Complete
       │
       ├── UI requirement? ──NO──▶ Phase 3 (Story Design)
       │
       └── YES
            │
            ▼
       UX Agent loads PRD §5.4 + SE Design §4.5 + design files + ux-guidelines.md
            │
            ▼
       UX Agent EXTRACTS UX requirements from sources (does NOT design)
            │
            ├── Trace every requirement to a source
            ├── Extract per-component visual/interaction/state requirements
            ├── Map design values to project design tokens
            ├── Document gaps (missing/ambiguous requirements)
            └── Flag gaps as CAT-1 challenges → request user input
            │
            ▼
       UX Agent produces UX Requirements Specification (ux-spec.md)
            │
            ▼
       Dev Agent reviews: are extracted requirements implementable?
            │
            ├── Sufficiently specified → ACCEPT → Phase 3
            ├── Gaps block implementation → UX Agent escalates to user → revise → re-review
            └── UX Agent invented requirements? → REJECT (violates extraction principle)
```

#### UX Specification Phase Report

The UX Agent generates `phase-2.6-ux-spec-report.md` containing:
- Sources analyzed (PRD, SE Design, design files, guidelines)
- Requirements extracted per source (count)
- Specification items traceable to sources (count)
- Gaps documented and escalated to user (list)
- Design token mapping coverage
- CAT-1 challenges raised for missing requirements
- Dev Agent implementability sign-off
- Confirmation: no requirements invented — all traceable

### Phase 3: Story Design (Development Story)
- **Agent:** Dev Agent (`framework/agents/dev-agent.skill.md`)
- **Input:** `docs/se-design.md`, `docs/prd.md`, `docs/ux-spec.md` (if UI requirement)
- **Output:** `docs/dev-story.md`
- **Format:** Follow `framework/artifacts/dev-story.template.md`
- **Reviewer:** SE Agent + UX Agent (if UI requirement)
- **Gate:** SE Agent confirms story aligns with design; **UX Agent confirms UI tasks have UX specification coverage (UX-S-01~UX-S-04)**

### Phase 4: Test Plan
- **Agent:** Test Agent (`framework/agents/test-agent.skill.md`)
- **Input:** `docs/prd.md`, `docs/se-design.md`, `docs/dev-story.md`
- **Output:** `docs/test-plan.md`
- **Format:** Follow `framework/artifacts/test-plan.template.md`
- **Reviewer:** Dev Agent + Requirements Provider (User)
- **Gate:** All PRD requirements (or Dev Story tasks, in shortcut path) have corresponding test cases
- **Shortcut note:** When PRD is absent, acceptance criteria are derived from Dev Story tasks and implementation details

### Phase 5: Development Coding
- **Agent:** Dev Agent (`framework/agents/dev-agent.skill.md`)
- **Input:** `docs/dev-story.md`, `docs/test-plan.md`
- **Output:** Source code changes
- **Format:** Follow language-specific coding standards and `framework/standards/project-structure.template.md`
- **Reviewer:** SE Agent (in Phase 6 Code Review)
- **Gate:** Dev Agent self-validates: tests pass, code compiles, standards check passes

### Phase 5.5: Catalog Compliance Self-Check (目录合规自检)

**Mandatory gate between Dev Coding (Phase 5) and Phase 5 Report generation.**

Before generating the Phase 5 Report, the Dev Agent MUST execute the following self-check:

1. **List new exported methods:** All exported functions/classes/types created or modified in this phase
2. **Check method catalog:** Verify each new exported method against `docs/public-method-catalog.md` Method-to-Module Index — no duplicate exists
3. **List new constants:** All constants/enums/as-const objects created or modified in this phase
4. **Check constant catalog:** Verify each new constant against `docs/constant-catalog.md` Constant-to-Module Index — no duplicate; shared constants (used by 2+ modules) are registered in the Shared Constants table
5. **Check terminology:** Verify each new type/interface/function name against `docs/terminology-glossary.md` Term Index — naming is consistent with existing domain terminology
6. **Write symbol declaration:** The above checklist results MUST be written into the Phase 5 Report under a "New Symbol Declaration" (新增符号声明) section

**Gate rule:** If any check FAILS → self-correct → re-run self-check → ALL PASS before Phase 5 Report generation.

**Challenge rule:** If the self-check discovers a catalog conflict (duplicate method, unregistered constant, terminology clash), the Dev Agent MUST file a **self-challenge** (CAT-2a/2b/2c) with a CH-YYYY-NNN record in the requirement's `challenges/` directory. The self-challenge must be RESOLVED before the Phase 5 Report can be generated.

### Phase 6: Code Review (代码审核)
- **Agent:** SE Agent (`framework/agents/se-agent.skill.md`) — acting as Code Reviewer (CR-1~CR-8)
- **Co-Reviewer (if UI):** UX Agent (`framework/agents/ux-agent.skill.md`) — UX Specification Compliance Reviewer (CR-9)
- **Input:** Source code changes, `docs/dev-story.md`, `docs/prd.md` (or Dev Story if shortcut), `docs/test-plan.md`, language-specific coding standards, `docs/architecture.md`, `docs/ux-spec.md` (if UI requirement), `docs/ux-guidelines.md` (if UI)
- **Output:** `docs/code-review-report.md` (with CR-9 section if UI)
- **Format:** Follow the Code Review Checklist (9 items, defined below)
- **Reviewer:** — (this phase IS the review; its output gates Phase 7)
- **Gate:** ALL applicable checklist items must pass. Any failure triggers a formal challenge. **For UI requirements, CR-9 must pass in addition to CR-1~CR-8.**

#### Code Review Checklist

| ID | Item | Reviewer | What to Verify | Challenge Basis |
|----|------|----------|---------------|-----------------|
| CR-1 | Requirement Completeness | SE Agent | Every PRD requirement / Dev Story task has corresponding implementation. No TODOs, FIXMEs, stubs. | Cite specific PRD FR-XXX or Dev Story TASK-XXX |
| CR-2 | Dev Story Alignment | SE Agent | Code structure matches Dev Story: file paths, function signatures, class structures, data flow. Deviations must be documented. | Cite specific Dev Story §section and line |
| CR-3 | Standards Compliance | SE Agent | Naming, formatting, file organization follow coding standards + §0.6 UX规范约定 (if UI). No prohibited patterns. | Cite specific standards chapter/clause |
| CR-4 | Architectural Integrity | SE Agent | No circular dependencies. Module dependency rules respected. Public API encapsulation correct. | Cite `docs/architecture.md` section or dependency rule |
| CR-5 | Correctness | SE Agent | All code paths reachable. Error handling covers all documented scenarios. Edge cases handled. No race conditions. | Cite specific error scenario from Dev Story §5 or SE Design |
| CR-6 | Test Coverage | SE Agent | All new functions have tests. AAA pattern followed. Happy path, error path, boundary conditions covered. No regressions. | Cite specific function or test case from test plan |
| CR-7 | Security | SE Agent | No hardcoded secrets. External input validated. Injection vectors closed. No eval() or equivalent. | Cite coding standards §Security section |
| CR-8 | No Omissions | SE Agent | No commented-out code. No empty catch blocks. No unreachable code. No debug/log temp code. All imports used. | Cite specific file:line |
| **CR-9** | **UX Specification Compliance** | **UX Agent** | **UI implementation conforms to extracted UX Specification: visual requirements, interaction requirements, responsive requirements, accessibility requirements, design system compliance, UX state completeness. (Conditional — only when requirement has UI.)** | **Cite UX Spec §section, §0.6 clause, or `docs/ux-guidelines.md`** |

**CR-9 Sub-Items:**

| Sub-ID | Item | Challenge Basis |
|--------|------|----------------|
| CR-9.1 | Visual Requirements Compliance: Component states match UX Spec §2 extracted requirements | UX Spec §2.X per-component visual requirements table |
| CR-9.2 | Interaction Requirements Compliance: Behavior matches UX Spec §3 | UX Spec §3 interaction requirements |
| CR-9.3 | Responsive Requirements Compliance: Layout breaks at correct breakpoints per UX Spec §4 | UX Spec §4 responsive requirements |
| CR-9.4 | Accessibility Requirements Compliance: ARIA, focus, semantics per UX Spec §5 | UX Spec §5 accessibility requirements |
| CR-9.5 | Design System Compliance: Design tokens used per UX Spec §6, no hardcoded styles | `docs/coding-standards.md` §0.6 + UX Spec §6 |
| CR-9.6 | UX State Completeness: Loading, empty, error states implemented per spec | UX Spec §2 per-component state requirements |

#### Code Review Flow Control

```
Phase 5 Complete
      │
      ├── UI requirement? ──NO──▶ SE Agent reviews CR-1~CR-8
      │                                │
      │                                └── ALL 8 PASS → APPROVED → Phase 7
      │
      └── YES
           │
           ├── SE Agent reviews CR-1~CR-8 (in parallel)
           └── UX Agent reviews CR-9
                 │
                 ├── ALL CR-1~CR-9 PASS → Report APPROVED → Phase 7
                 │
                 ├── CR-1~CR-8 FAIL → SE Agent challenges Dev Agent
                 └── CR-9 FAIL → UX Agent files CAT-3 challenge against Dev Agent
                            │
                            ▼
                     Dev Agent fixes code
                            │
                            ├── CR-1~CR-8 fixes → SE Agent re-reviews (failed items only)
                            └── CR-9 fixes → UX Agent re-reviews (failed sub-items only)
                                    │
                                    ├── ALL PASS → Phase 7
                                    └── STILL FAILING → Escalate to User
```

- **Phase 7 CANNOT begin** until the Code Review Report status is APPROVED (all applicable CR-1~CR-9 pass)
- Re-review cycle: same item fails twice → escalate to user for a final decision
- Each failed checklist item generates a separate formal challenge entry in the review report
- CR-1~CR-8 challenges: SE Agent vs Dev Agent (standard); **CR-9 challenges: UX Agent vs Dev Agent (CAT-3)**
- Challenges follow all rules in `framework/workflows/challenge-mechanism.rule.md`

### Phase 6.5: Catalog Consistency Verification (目录一致性核查)

**Mandatory gate between Code Review (Phase 6) APPROVAL and Phase 7 (Validation).**

After all applicable Code Review checklist items pass (CR-1~CR-8 by SE Agent, CR-9 by UX Agent if UI; status: APPROVED), the SE Agent MUST perform an additional catalog consistency check before the orchestrator advances to Phase 7:

1. **Method declaration check:** Compare the "New Symbol Declaration" section in the Phase 5 Report against the actual code diff. Every new exported method in the code MUST be listed in the declaration. Every declared method MUST exist in the code diff.
2. **Constant declaration check:** Compare the "New Symbol Declaration" section in the Phase 5 Report against the actual code diff. Every new constant in the code MUST be listed in the declaration. Every declared constant MUST exist in the code diff.
3. **Cross-catalog verification:** Verify that the new symbols in the code diff are consistent with all three catalog documents:
   - `docs/public-method-catalog.md` — no undeclared new methods
   - `docs/constant-catalog.md` — no unregistered shared constants
   - `docs/terminology-glossary.md` — no terminology inconsistency

**Gate rule:** Any inconsistency → **immediately raise a CAT-2 challenge** (CAT-2a for method issues, CAT-2b for constant issues, CAT-2c for terminology issues) → challenge record created → Phase 7 is BLOCKED until the challenge is RESOLVED.

**Challenge record requirement:** Each Phase 6.5 violation generates a separate CH-YYYY-NNN record citing the specific catalog document, the symbol in question, and the nature of the inconsistency.

**Re-verification:** After Dev Agent resolves the challenge, the SE Agent re-executes Phase 6.5 only (not the full 8-item checklist, unless the fix introduced new code changes requiring re-review).

### Phase 7: Process Validation
- **Agent:** Test Agent (via plugin invocation)
- **Input:** Source code, `docs/test-plan.md`, `docs/code-review-report.md`
- **Output:** Validation report
- **Format:** Follow `framework/standards/validation-standards.template.md`
- **Gate:** All validation criteria pass

## Phase Transition Rules

1. **Before each phase transition**, check `framework/workflows/plugin-extension.rule.md` for plugin hooks
2. **Before each phase transition**, check `framework/workflows/challenge-mechanism.rule.md` for pending challenges
3. **A phase is complete** only when its quality gate is satisfied AND its reviewer has accepted the output
4. **Phase 2.6 (UX Specification Extraction) is a conditional gate** — executes only when project has UI AND requirement involves UI design; UX Agent extracts requirements from existing design artifacts (does not design); when skipped, transition goes from Phase 2 → Phase 3 directly
5. **Phase 5.5 is a hard gate** — Phase 5 Report cannot be generated until catalog compliance self-check passes
6. **Phase 6 is a hard gate** — Phase 7 is unreachable without an APPROVED code review report (CR-1~CR-8, plus CR-9 if UI)
7. **Phase 6.5 is a hard gate** — Phase 7 is unreachable if catalog consistency verification (or UX consistency) raises unresolved challenges
7. **Full lifecycle: no phase skipping** — each phase (including sub-phases) must complete in order
8. **Shortcut path: defined entry point** — enter at Phase 4 or Phase 5 with user-provided Dev Story
9. **Phase re-entry** — a challenge may force re-entry to a prior phase

## State Tracking

The orchestrator tracks lifecycle state implicitly through document presence:

| Document | Indicates |
|----------|-----------|
| `docs/prd.md` | Phase 1 complete |
| `docs/se-design.md` | Phase 2 complete |
| `docs/dev-story.md` | Phase 3 complete (or shortcut entry) |
| `docs/test-plan.md` | Phase 4 complete |
| Git commit with source changes | Phase 5 complete |
| `docs/code-review-report.md` with status `APPROVED` | Phase 6 complete |
| `docs/requirements/REQ-*/ux-spec.md` | Phase 2.6 complete (if UI requirement) |
| `docs/validation-report.md` with all pass | Phase 7 complete (lifecycle done) |

## Agent Handoff Protocol

When Agent A hands off to Agent B:
1. Agent A outputs document following the relevant artifact template
2. Agent A announces completion with a summary of key decisions
3. Agent B reads the document fully before responding
4. Agent B either accepts (proceeds to next phase) or challenges (enters challenge workflow)
5. If accepted, Agent B begins its phase; if challenged, Agent A must resolve first

**Special handoff — Phase 5 → Phase 6:**
The Dev Agent does NOT hand off directly to the SE Agent. Instead, the orchestrator invokes the SE Agent in Code Review mode. The SE Agent reads the Dev Story, code diff, test plan, and standards, then executes the 8-item checklist. This is a pull-based review, not a push-based handoff.
