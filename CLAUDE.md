# General Development Lifecycle Framework

AI-agent-driven software development lifecycle framework. Orchestrates specialized agents (Scanner, SE, UX, Dev, Test) through PRD→SE→UX Spec→Story→Test→Dev→Code Review→Validation workflow with built-in challenge/audit, plugin extension, and UX constraint enforcement. Supports a Dev-Story shortcut path.

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                       CLAUDE.md (Orchestrator)                    │
├──────────────────────────────────────────────────────────────────┤
│  Agents Layer       │ Scanner │ SE │ UX │ Dev │ Test Agents       │
├──────────────────────────────────────────────────────────────────┤
│  Workflow Layer     │  Standard Lifecycle + Challenge + Plugin    │
│                     │  + Dev-Story Shortcut + UX Spec + Review    │
├──────────────────────────────────────────────────────────────────┤
│  Artifacts Layer    │  PRD │ SE Design │ UX Spec │ Story │ Test  │
├──────────────────────────────────────────────────────────────────┤
│  Standards Layer    │  Coding(JS/TS/Java/Python) │ Structure      │
│                     │  + UX Guidelines (if UI detected)           │
├──────────────────────────────────────────────────────────────────┤
│  Plugin Layer       │  Custom hooks injected at each stage        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Quick Start

### Phase 0: Framework Initialization (MANDATORY)

**Phase 0 is a hard prerequisite.** No lifecycle phase may execute before initialization completes. If `docs/.framework-init.lock` is absent, the request is REJECTED. To initialize:

```
@CLAUDE.md 初始化框架
```

Full initialization protocol: `framework/workflows/initialization.rule.md`

---

## Entry Points

### Entry A: Full Lifecycle
```
@CLAUDE.md 基于PRD开始开发流程
```
Executes the complete 7-phase lifecycle: PRD → SE Design → Story → Test Plan → Dev → Code Review → Validation.

### Entry B: Dev-Story Shortcut
```
@CLAUDE.md 基于Dev Story开始开发
```
User provides a pre-existing Dev Story. The framework bypasses upstream phases:

| Artifacts Present | Entry Phase |
|-------------------|-------------|
| Dev Story only | Phase 5 (Dev Coding): write code, then Test Agent creates test plan and validates |
| Dev Story + Test Plan | Phase 5 (Dev Coding): code against both story and test plan |
| Dev Story + Test Plan + Code written | Phase 6 (Code Review) |
| No dev story | Phase 1 (PRD) — full lifecycle |

**Shortcut constraints:** Without SE Design, Code Review uses Dev Story as design reference. Without PRD, Test Agent derives cases from Dev Story. Challenges still apply.

---

## Framework Rules (priority order)

All detailed rules are in `framework/workflows/`. The orchestrator loads them in priority order:

| Priority | Rule File | Purpose |
|----------|-----------|---------|
| 0 | `framework/workflows/challenge-mechanism.rule.md` | Challenge/audit — highest priority |
| 1 | `framework/workflows/plugin-extension.rule.md` | Plugin hooks — checked before each step |
| 2 | `framework/workflows/ux-constraint.rule.md` | UX constraint enforcement — blocks phases when UI design involved |
| 3 | `framework/workflows/standard-lifecycle.rule.md` | 7-phase workflow + shortcut + Phase 5.5/6.5 catalog gates |
| 4 | `framework/workflows/execution-protocol.rule.md` | 9-step execution sequence for every lifecycle run |
| 5 | `framework/workflows/initialization.rule.md` | Phase 0 protocol — language detection, scanning, doc generation, lock |
| 6 | `framework/workflows/requirement-model.rule.md` | Requirement directory model, REQ-ID assignment, lifecycle tracking |
| 7 | `framework/workflows/coding-standards-hierarchy.rule.md` | Two-tier standards hierarchy + §0.6 UX规范约定 |
| 8 | `framework/standards/coding-standards.<lang>.md` | Language-specific standards (JS/TS/Java/Python) |
| 9 | `framework/standards/project-structure.template.md` | Project structure spec |
| 10 | `framework/standards/validation-standards.template.md` | Process validation spec |

---

## Agent Skills

| Skill File | Agent | Role |
|-------------|-------|------|
| `framework/agents/project-scanner.skill.md` | Scanner Agent | Project scanning, module mapping, shared resource detection, dependency analysis, UX convention detection |
| `framework/agents/se-agent.skill.md` | SE Agent | Architecture design, SE requirements, Code Review (CR-1~CR-8), UX architecture validation |
| `framework/agents/ux-agent.skill.md` | UX Agent | UX Specification Extraction (Phase 2.6), UX Compliance Review (CR-9), UX constraint validation, CAT-3 challenges |
| `framework/agents/dev-agent.skill.md` | Dev Agent | Story design, coding, challenge resolution, UX implementation per §0.6 and UX Spec |
| `framework/agents/test-agent.skill.md` | Test Agent | Test planning, process validation |

---

## Phase Reference

| Phase | Agent | Input | Artifact | Report | Template | Reviewer |
|-------|-------|-------|----------|--------|----------|----------|
| 1. PRD | User | Business needs | `prd.md` | `reports/phase-1-prd-report.md` | `framework/artifacts/prd.template.md` | — |
| 2. SE Design | SE Agent | PRD, architecture docs | `se-design.md` | `reports/phase-2-se-design-report.md` | `framework/artifacts/se-design.template.md` | Dev Agent |
| **2.6 UX Spec** | **UX Agent** | **PRD §5.4, SE Design §4.5, UX guidelines** | **`ux-spec.md`** | **`reports/phase-2.6-ux-spec-report.md`** | **`framework/artifacts/ux-spec.template.md`** | **Dev Agent** |
| 3. Story Design | Dev Agent | SE Design, PRD, UX Spec (if UI) | `dev-story.md` | `reports/phase-3-story-design-report.md` | `framework/artifacts/dev-story.template.md` | SE Agent + UX Agent |
| 4. Test Plan | Test Agent | PRD, SE Design, Dev Story | `test-plan.md` | `reports/phase-4-test-plan-report.md` | `framework/artifacts/test-plan.template.md` | Dev Agent + User |
| 5. Dev Coding | Dev Agent | Dev Story, Test Plan, UX Spec (if UI) | Source code | `reports/phase-5-dev-coding-report.md` | Language-specific standards + §0.6 | SE Agent (CR 1~8), UX Agent (CR-9) |
| 5.5 Catalog Self-Check | Dev Agent | Code diff, catalogs | Self-check in Phase 5 Report | (embedded) | `standard-lifecycle.rule.md` §5.5 | — |
| 6. Code Review | SE Agent + UX Agent | Code, Dev Story, PRD, Test Plan, UX Spec | Review Report | `reports/phase-6-code-review-report.md` | 9-item checklist (CR-1~CR-9) | — (gate to Phase 7) |
| 6.5 Catalog Verification | SE Agent | Code diff, catalogs, Phase 5 Report | Verification in Phase 6 Report | (embedded) | `standard-lifecycle.rule.md` §6.5 | — |
| 7. Validation | Test Agent | Code, Test Plan, Review Report | `validation-report.md` | `reports/phase-7-validation-report.md` | `framework/standards/validation-standards.template.md` | — |

**Key gates:** Phase 0 lock → Phase 2.6 UX Spec Extraction (if UI) → Phase 5.5 catalog self-check → Phase 6 (CR-1~CR-9 checklist) → Phase 6.5 catalog consistency → Phase 7 validation. See `framework/workflows/standard-lifecycle.rule.md` for full phase details.

### UX Specification & Review Gates (Conditional — only when project has UI)

When the project has UI (`docs/.framework-init.lock` → `ui_stack.has_ui: true`), UX-specific phases and gates activate. **The UX Agent extracts requirements from existing design artifacts — it does NOT create designs.**

| Gate | Phase | What | Reviewer | Rule |
|------|-------|------|----------|------|
| UX-C01 | 1→2 | PRD must include §5.4 UX Constraints (UX-01 to UX-05) | SE Agent | `ux-constraint.rule.md` |
| UX-C02 | 2→2.6 | SE Design must include §4.5 UI Architecture | UX Agent | `se-design.template.md` |
| UX-S01 | **2.6** | **UX Requirements Specification extracted** (from PRD, design files, SE Design — no invention) | **Dev Agent** | **`ux-agent.skill.md`** |
| UX-C03 | 3→4 | Dev Story UI tasks have UX spec coverage (UX-S-01~UX-S-04) | UX Agent | `ux-constraint.rule.md` |
| UX-R01 | **6 (CR-9)** | **UX Spec Compliance Review: checks code against extracted spec only** | **UX Agent → Dev Agent** | **`standard-lifecycle.rule.md` §CR-9** |

---

## Directory Structure

```
project/
├── CLAUDE.md                              # Master orchestrator (this file)
├── .claude/settings.json                  # Harness settings
├── framework/
│   ├── agents/                            # project-scanner, se-agent, ux-agent, dev-agent, test-agent .skill.md
│   ├── workflows/                         # 8 rule files (challenge, plugin, ux-constraint, lifecycle, execution, init, requirement-model, coding-standards-hierarchy)
│   ├── artifacts/                         # prd, se-design, ux-spec, dev-story, test-plan, architecture-doc, module-detail .template.md + catalog templates
│   └── standards/                         # coding-standards (js/ts/java/python/template), project-structure, validation-standards
├── plugins/                               # Plugin extensions
└── docs/                                  # Generated documentation
    ├── .framework-init.lock               # Init marker + project metadata (JSON)
    ├── architecture.md                    # Shared — project architecture
    ├── module-map.md                      # Shared — file-to-module map + keyword index
    ├── coding-standards.md                # Shared — §0 project-level + language standards
    ├── project-structure.md               # Shared — directory conventions
    ├── ux-guidelines.md                   # Shared — UX guidelines (if UI detected)
    ├── public-method-catalog.md           # Shared — exported method inventory
    ├── constant-catalog.md                # Shared — constants inventory
    ├── terminology-glossary.md            # Shared — domain terminology
    ├── .scanner-report.json               # Shared — machine-readable scan data
    ├── challenges/
    │   └── index.md                       # Global challenge index
    ├── modules/                           # Per-module docs (large projects)
    └── requirements/                      # Requirement directory root
        ├── index.md                       # Requirement registry
        └── REQ-YYYY-NNN-{slug}/           # Isolated requirement directory
            ├── prd.md                     # Phase 1 artifact
            ├── se-design.md               # Phase 2 artifact
            ├── ux-spec.md                 # Phase 2.6 artifact (if UI) — extracted UX requirements
            ├── dev-story.md               # Phase 3 artifact
            ├── test-plan.md               # Phase 4 artifact
            ├── validation-report.md       # Phase 7 artifact
            ├── reports/                   # Phase completion reports
            └── challenges/                # Challenge records
```

---

## Execution Summary

1. **Determine mode:** Check lock → Entry A (full) or Entry B (shortcut) or reject
2. **Create requirement dir:** Assign REQ-ID, create directory, register in index
3. **Load environment:** Architecture, module map, catalogs, UX guidelines (if UI), standards, plugins
4. **Pre-flight:** Resolve pending challenges, run pre-hooks, validate inputs, **check UX constraints if UI**
5. **Execute phase:** Agent produces artifact + phase report → proactive challenge scan → post-hooks
6. **UX Spec Extraction (conditional):** If project has UI, UX Agent extracts UX requirements from existing design artifacts into UX Specification (does not design)
7. **Handoff/Halt:** Accept → advance phase; Challenge → create challenge record, block transition
8. **Challenge resolution:** Fix → re-execute phase → re-review
9. **Code review gate:** CR-1~CR-8 (SE Agent) + CR-9 UX Review (UX Agent if UI) → APPROVED → Phase 7
10. **Completion:** Phase 7 passes → mark requirement COMPLETED

---

## Key Principles

- **Challenges are the primary iteration mechanism** — agents must proactively raise CAT-1 (requirement gaps), CAT-2 (standards violations), and CAT-3 (UX violations) challenges during their work, not wait for review phases
- **Catalog integrity is mandatory** — agents must consult `public-method-catalog.md`, `constant-catalog.md`, and `terminology-glossary.md` before creating new symbols; Phase 5.5 and 6.5 gates enforce this
- **Project standards take precedence** — `docs/coding-standards.md` §0 overrides all generic language standards when they conflict; §0.6 (UX规范约定) applies to all UI work
- **UX is a first-class quality gate** — when `ui_stack.has_ui` is true, UX Specification (Phase 2.6) extracts requirements from existing designs, and UX Spec Compliance Review (CR-9) validates implementation before Phase 7. UX Agent does NOT design — it extracts and enforces
- **UX constraints are mandatory for UI projects** — PRD and Dev Story phases involving UI design are blocked until UX constraints are fulfilled
- **Phase reports are MANDATORY** — no phase is complete without its report; no phase transition without report
- **The lock file is the gatekeeper** — no lifecycle execution without `docs/.framework-init.lock`
