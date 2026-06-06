# General Development Lifecycle Framework

AI-agent-driven software development lifecycle framework. Orchestrates specialized agents through PRD→SE→Story→Test→Dev→Code Review→Validation workflow with built-in challenge/audit and plugin extension mechanisms. Supports a Dev-Story shortcut path.

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                       CLAUDE.md (Orchestrator)                    │
├──────────────────────────────────────────────────────────────────┤
│  Agents Layer       │ Scanner │ SE Agent │ Dev Agent │ Test Agent│
├──────────────────────────────────────────────────────────────────┤
│  Workflow Layer     │  Standard Lifecycle + Challenge + Plugin    │
│                     │  + Dev-Story Shortcut                       │
├──────────────────────────────────────────────────────────────────┤
│  Artifacts Layer    │  PRD │ SE Design │ Story │ Test Plan       │
├──────────────────────────────────────────────────────────────────┤
│  Standards Layer    │  Coding(JS/TS/Java/Python) │ Structure      │
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
User provides a pre-existing `docs/dev-story.md`. The framework bypasses upstream phases:

| Artifacts Present | Entry Phase |
|-------------------|-------------|
| Dev Story only | Phase 5 (Dev Coding): write code, then Test Agent creates test plan and validates |
| Dev Story + Test Plan | Phase 5 (Dev Coding): code against both story and test plan |
| Dev Story + Test Plan + Code written | Phase 6 (Code Review) |
| No dev story | Phase 1 (PRD) — full lifecycle |

**Shortcut constraints:** Without SE Design, Code Review uses Dev Story as design reference. Without PRD, Test Agent derives cases from Dev Story. Challenges still apply.

---

## Requirement Directory Model

Each requirement/feature gets an isolated directory under `docs/requirements/`. All lifecycle artifacts, reports, and challenges for that requirement live within it. Shared project-level docs (architecture, standards, module map) remain at `docs/` root.

### Directory Layout
```
docs/
├── architecture.md                   # Shared — project-wide
├── module-map.md                     # Shared — project-wide
├── coding-standards.md               # Shared — project-wide
├── project-structure.md              # Shared — project-wide
├── challenges/
│   └── index.md                      # Shared — global challenge index
└── requirements/
    ├── index.md                      # Requirement registry
    └── REQ-YYYY-NNN-{slug}/
        ├── prd.md                    # Phase 1 artifact
        ├── se-design.md              # Phase 2 artifact
        ├── dev-story.md              # Phase 3 artifact
        ├── test-plan.md              # Phase 4 artifact
        ├── reports/
        │   ├── phase-1-prd-report.md
        │   ├── phase-2-se-design-report.md
        │   ├── phase-3-story-design-report.md
        │   ├── phase-4-test-plan-report.md
        │   ├── phase-5-dev-coding-report.md
        │   ├── phase-6-code-review-report.md
        │   └── phase-7-validation-report.md
        └── challenges/
            └── CH-YYYY-NNN.md
```

### Requirement Index (`docs/requirements/index.md`)
```markdown
# Requirement Index
| REQ ID | Name | Status | Current Phase | Created | Completed |
|--------|------|--------|---------------|---------|-----------|
| REQ-2026-001 | user-auth | Phase 5 | 2026-06-06 | - |
| REQ-2026-002 | payment | Phase 2 | 2026-06-06 | - |
```
Status values: `ACTIVE`, `COMPLETED`, `ABANDONED`, `ON-HOLD`.

### Requirement Lifecycle
1. **Creation:** When a new PRD is provided (Entry A) or a Dev Story is provided (Entry B), the orchestrator assigns a REQ-ID and creates the requirement directory
2. **Execution:** All phases for this requirement write into its directory
3. **Completion:** When Phase 7 passes, the requirement is marked COMPLETED in the index
4. **Parallel requirements:** Multiple requirements can be ACTIVE simultaneously; each has its own directory and independent phase state

### REQ-ID Assignment
- Format: `REQ-YYYY-NNN` (year + 3-digit sequential counter)
- Counter resets per year, padded to 3 digits (001, 002, ...)
- Slug is derived from the PRD title or Dev Story name, lowercased with hyphens

---

## Phase 0: Initialization Protocol

### Enforcement Rules
1. Before ANY lifecycle operation, verify `docs/.framework-init.lock` exists. If absent, REJECT.
2. Re-initialization: only on explicit user request — delete lock, re-run from scratch.
3. Staleness: if lock >30 days old, suggest re-init but don't force.
4. Empty project: still execute Phase 0 (minimal scaffolding), still create lock.
5. Partial init forbidden: if any step fails, report, roll back docs, instruct user.

### Protocol Flow (8 sequential steps)

```
Step 0.1 → Step 0.2 → Step 0.3 → Step 0.4 → Step 0.5 → Step 0.6 → Step 0.7 → Step 0.8
Detect    Scan      Identify   Analyze    Generate   Validate   Classify   Create
Language  Files     Modules    Deps       Docs       & Report   Project    Lock
```

### Step 0.1: Detect Language Stack & Metadata
Scan project root for build manifests in priority order: `package.json` → `tsconfig.json` → `pom.xml` → `build.gradle` → `pyproject.toml` → `Cargo.toml` → `go.mod` → `*.csproj` → `Gemfile` → `CMakeLists.txt` → `pubspec.yaml`. Detect frameworks from dependencies (React/Vue/Angular/Express/Next/NestJS from `package.json`; Spring Boot/Quarkus from Java manifests; Django/Flask/FastAPI from Python manifests). Detect package manager from lock files. Extract project name, version, description, entry point, minimum runtime. Record complete language stack.

**Multi-root/monorepo:** If multiple first-level subdirectories each have their own manifest, flag as monorepo — list each sub-project separately. Primary = repo root.

**No manifest:** List all file extensions, present top 3 to user for confirmation, then load/generate standards.

### Step 0.2-0.4: Project Structure Scan → Delegate to Scanner Agent

Steps 0.2 (File Scanning), 0.3 (Module Identification), and 0.4 (Dependency Analysis) are executed by the **Project Scanner Agent** (`framework/agents/project-scanner.skill.md`).

The scanner performs six phases:
| Phase | Description | Key Output |
|-------|-------------|------------|
| **A. File Discovery** | Scan all source files, classify each by type, detect organizational pattern | File inventory with classifications |
| **B. Shared Resource Detection** | Identify shared components, methods, and infrastructure used across 2+ modules | Shared component/method/infra catalogs |
| **C. Module Boundaries** | Group files into modules, resolve path aliases, determine public API surfaces | Module inventory with import paths |
| **D. Dependency Analysis** | Map all import edges, classify weights, detect circular/bidirectional deps | Dependency matrix + violation report |
| **E. Generate Output** | Produce `docs/module-map.md`, `docs/.scanner-report.json`, architecture contributions | Machine-readable scan data |
| **F. Coding Style Detection** | Detect public method naming, file organization patterns, coding conventions from actual code | Convention report → merged into `docs/coding-standards.md` §0 |

**Key differentiators from generic file scanning:**
- **Shared vs. module-private:** Every component/utility is classified by its usage scope (shared if imported by 2+ modules)
- **Path alias resolution:** All imports are resolved through `tsconfig.json` paths or equivalent to produce canonical module paths
- **Reference relationship map:** Full directed graph showing which modules reference which, with edge weights
- **Architecture rules for shared code:** Shared components MUST NOT import from feature modules; features SHOULD NOT directly import from other features

The orchestrator invokes this skill during Phase 0 and on structural changes. See `framework/agents/project-scanner.skill.md` for the complete protocol.

### Step 0.5: Generate Documentation
Produce files in `docs/` (minimal set for empty projects):

| Document | Source |
|----------|--------|
| `docs/architecture.md` | Data from Steps 0.1 + Scanner Agent (Phases C/D) |
| `docs/module-map.md` | Scanner Agent (Phase E.1) |
| `docs/coding-standards.md` | Copy/adapt from `framework/standards/coding-standards.<lang>.md` |
| `docs/project-structure.md` | `framework/standards/project-structure.template.md` + detected pattern |
| `docs/.scanner-report.json` | Scanner Agent (Phase E.2) — machine-readable scan data |

`architecture.md` must include: project overview (Step 0.1), module architecture (Step 0.3), dependency graph with ASCII diagram (Step 0.4), data architecture, API architecture, cross-cutting concerns, evolution history. Mandatory ASCII diagrams: high-level architecture + module dependency graph.

`module-map.md` must include: summary, file-to-module cross-reference table (every source file), per-module detail sections (type, path, files, public API, depends on, depended by). For >50 source files, detail sections can be collapsed with note to `docs/modules/MOD-XXX.md`.

`coding-standards.md` is generated by merging the language-specific standard with Scanner Phase F detections:

1. Copy the matched standard from `framework/standards/coding-standards.<lang>.md`
2. Prepend a **§0: Project-Detected Conventions** section containing scanner-detected: public method naming, file organization, coding style (indentation, quotes, semicolons, etc.)
3. For each generic rule, apply the merge action from the scanner:
   - **confirm:** Rule matches project practice → annotate "✓ confirmed by project scan"
   - **override:** Project differs → replace with detected convention, annotate "⚠ overridden by project detection"
   - **add:** Project has a convention not in the generic standard → add as new rule
4. Flag inconsistencies (<70% coverage) as remediation items with file paths

This ensures `docs/coding-standards.md` reflects the project's actual conventions, not just generic language rules.

### Step 0.6: Validate Documentation
Cross-validation checks (any ERROR halts Phase 0):
- V-01: Every source file appears in module-map.md (ERROR)
- V-02: Every module in module-map.md appears in architecture.md §2 (ERROR)
- V-03: All MOD-XXX IDs unique (ERROR)
- V-04: File count matches scan count (ERROR)
- V-05: Module count matches inventory (ERROR)
- V-07: No module claims 0 files (ERROR, unless empty)
- V-09: All referenced standards files exist (ERROR)
- V-06/V-08/V-10: Dependency count, bidirectional edges, ASCII diagrams (WARNING)

Compile architecture violations from Step 0.4.3. Output standardized init summary.

### Step 0.7: Classify Project Complexity
| Category | Files | Modules | Behavior |
|----------|-------|---------|----------|
| Empty | 0 | 0 | Minimal docs, standards only |
| Small | 1-30 | 1-5 | All details inline |
| Medium | 31-100 | 6-15 | All inline, architecture may split sub-sections |
| Large | 101-500 | 16-50 | Split modules to `docs/modules/MOD-XXX.md` |
| Very Large | 500+ | 50+ | Architecture split into sub-documents |

### Step 0.8: Create Lock File
Write `docs/.framework-init.lock` as JSON with: `initialized_at`, `framework_version`, `project_name`, `language_stack`, `framework`, `organizational_pattern`, `complexity`, `source_file_count`, `module_count`, `circular_dependencies`, `architecture_violations`, `generated_docs`. This serves as init marker, project metadata card, and staleness timestamp.

---

## Coding Standards Selection

| Language | Standards File |
|----------|----------------|
| JavaScript (ES2022+) | `framework/standards/coding-standards.javascript.md` |
| TypeScript (5.x+) | `framework/standards/coding-standards.typescript.md` |
| Java (17 LTS+) | `framework/standards/coding-standards.java.md` |
| Python (3.11+) | `framework/standards/coding-standards.python.md` |
| Other | `framework/standards/coding-standards.template.md` |

Each standard covers 15 chapters: Naming, Code Structure, Formatting, Types, Classes, Modules, Async, Error Handling, Comments, Comparison, Performance, Security, Toolchain Configuration, naming cheat sheet, prohibited-items checklist. For polyglot projects, agents load only their language's standard; SE Agent loads all when designing cross-language integration.

---

## Standard Lifecycle

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
                                                                      │
                                                       APPROVED → Phase 7
                                                       CHALLENGED → back to Phase 5
```

### Shortcut Path
```
User provides dev-story.md
         │
         ├── Test Plan missing ──▶ Phase 4 (Test Plan) → Phase 5 → Phase 6 → Phase 7
         ├── Test Plan exists  ──▶ Phase 5 (Dev Coding) → Phase 6 → Phase 7
         └── Code written      ──▶ Phase 6 (Code Review) → Phase 7
```

### Phase Details

All paths are relative to the requirement directory `docs/requirements/REQ-YYYY-NNN-{slug}/`.

| Phase | Agent | Input | Artifact | Report | Template | Reviewer |
|-------|-------|-------|----------|--------|----------|----------|
| 1. PRD | User | Business needs | `prd.md` | `reports/phase-1-prd-report.md` | `framework/artifacts/prd.template.md` | — |
| 2. SE Design | SE Agent | PRD, architecture docs | `se-design.md` | `reports/phase-2-se-design-report.md` | `framework/artifacts/se-design.template.md` | Dev Agent |
| 3. Story Design | Dev Agent | SE Design, PRD | `dev-story.md` | `reports/phase-3-story-design-report.md` | `framework/artifacts/dev-story.template.md` | SE Agent |
| 4. Test Plan | Test Agent | PRD, SE Design, Dev Story | `test-plan.md` | `reports/phase-4-test-plan-report.md` | `framework/artifacts/test-plan.template.md` | Dev Agent + User |
| 5. Dev Coding | Dev Agent | Dev Story, Test Plan | Source code | `reports/phase-5-dev-coding-report.md` | Language-specific standards | SE Agent (Phase 6) |
| 6. Code Review | SE Agent | Source code, Dev Story, PRD, Test Plan | Review Report | `reports/phase-6-code-review-report.md` | §Code Review Checklist | — (gate to Phase 7) |
| 7. Validation | Test Agent | Source code, Test Plan, Review Report | `validation-report.md` | `reports/phase-7-validation-report.md` | `framework/standards/validation-standards.template.md` | — |

Each phase produces TWO output files within the requirement directory: the primary artifact and a phase completion report.

### Phase Transition Rules
1. Before each transition, check plugin hooks and pending challenges
2. A phase is complete only when quality gate is satisfied AND reviewer accepts
3. No phase skipping in full lifecycle; shortcut has defined entry points
4. **Phase 6 is a hard gate** — Phase 7 cannot begin until review is APPROVED

### State Tracking
**Tier 0 — Init Gate:** `docs/.framework-init.lock` absent = ALL lifecycle requests rejected.

**Tier 1 — Requirement State:** Tracked per-requirement via `docs/requirements/index.md`. Each REQ row records its current phase and status.

**Tier 2 — Phase State** (tracked via artifact + report within the requirement directory):
| Artifact | Report | Indicates |
|----------|--------|-----------|
| `prd.md` | `reports/phase-1-prd-report.md` | Phase 1 complete |
| `se-design.md` | `reports/phase-2-se-design-report.md` | Phase 2 complete |
| `dev-story.md` | `reports/phase-3-story-design-report.md` | Phase 3 complete (or shortcut entry) |
| `test-plan.md` | `reports/phase-4-test-plan-report.md` | Phase 4 complete |
| Git commit with source changes | `reports/phase-5-dev-coding-report.md` | Phase 5 complete |
| `reports/phase-6-code-review-report.md` with APPROVED | — | Phase 6 complete |
| `validation-report.md` | `reports/phase-7-validation-report.md` | Phase 7 complete |

**Tier 3 — Challenge State:** Tracked via `docs/challenges/index.md` — any OPEN challenge blocks phase transition for its requirement.

### Phase Completion Reports (MANDATORY)

Every phase MUST generate a phase report within the requirement's `reports/` directory. The report documents what happened during execution and serves as the quality gate record for phase transition.

**Report path:** `docs/requirements/REQ-YYYY-NNN-{slug}/reports/phase-{N}-{phase-name}-report.md`

**Report format:**
```markdown
# Phase {N} Report: {Phase Name}
- **Report ID:** PH{N}-YYYY-NNN | **Date:** YYYY-MM-DD HH:MM
- **Agent:** {Agent Name} | **Status:** COMPLETED | CHALLENGED | FAILED

## Input Artifacts
| Artifact | Path | Status |
|----------|------|--------|
| PRD | docs/prd.md | ✓ Present |
| ... | ... | ... |

## Execution Summary
- **Steps executed:** N
- **Decisions made:** (list key design/implementation decisions)
- **Deviations:** (any deviations from template or upstream spec, with justification)

## Output Artifacts
| Artifact | Path | Lines/Size |
|----------|------|------------|
| ... | ... | ... |

## Quality Gate
- [ ] Output conforms to template
- [ ] All mandatory sections populated
- [ ] Reviewer (if applicable) has accepted
- **Gate result:** PASS | FAIL (with reason)

## Plugin Hooks Executed
- `{phase}-pre`: executed / skipped (no hooks registered)
- `{phase}-post`: executed / skipped

## Reviewer Sign-off
- **Reviewer:** {Name} | **Decision:** ACCEPT | CHALLENGE
- **Challenge ID:** CH-YYYY-NNN (if challenged)
```

**Report generation rule:** The executing agent generates the report immediately after producing the primary artifact and before handoff. If a phase is re-entered (e.g., after challenge resolution), a new report is generated superseding the previous one.

### Challenge Documentation (MANDATORY)

Every challenge MUST be recorded as a standalone document. Per-requirement challenges live in the requirement's `challenges/` directory. The global index at `docs/challenges/index.md` aggregates all challenges across requirements.

**Challenge path:** `docs/requirements/REQ-YYYY-NNN-{slug}/challenges/CH-YYYY-NNN.md` (sequential counter, padded to 3 digits)

**Challenge record format:**
```markdown
# Challenge CH-YYYY-NNN
- **Filed:** YYYY-MM-DD HH:MM | **Phase:** {N} {Phase Name}
- **Challenger:** {Agent Name} | **Challenged:** {Agent Name}
- **Status:** OPEN | RESOLVED | ESCALATED | REJECTED
- **Resolution Date:** YYYY-MM-DD HH:MM (if resolved)

## Basis
Specific standard/clause/rule citation (e.g., "CR-3: Standards Compliance — coding-standards.typescript.md §4.2")

## Evidence
[File path:line — concrete example of the violation]

## Impact
[What breaks, what risks are introduced, what downstream phases are affected]

## Suggested Fix
[Specific remediation steps]

## Resolution
- **Resolved by:** {Agent Name} | **Date:** YYYY-MM-DD HH:MM
- **Changes made:** [Summary of what changed]
- **Verification:** [How the fix was verified]
- **Re-review result:** PASS | FAIL
```

**Challenge lifecycle:**
1. Challenger creates the challenge record in the requirement's `challenges/` directory with status OPEN
2. Challenged agent reviews, implements fix, updates Resolution section → status RESOLVED
3. Challenger verifies fix → if accepted, close; if rejected, status returns to OPEN with explanation
4. If deadlocked after 2 rounds → status ESCALATED, user decides
5. Invalid challenges (no cited basis) → status REJECTED with explanation
6. On every status change, update both the challenge record AND `docs/challenges/index.md`

**Challenge index:** Maintain `docs/challenges/index.md` as a global table across all requirements:
```markdown
# Challenge Index
| ID | REQ ID | Phase | Filed | Challenger | Challenged | Basis | Status | Resolution Date |
|----|--------|-------|-------|------------|------------|-------|--------|-----------------|
| CH-2026-001 | REQ-2026-001 | 6 | 2026-06-06 | SE Agent | Dev Agent | CR-2 | RESOLVED | 2026-06-06 |
```

---

## Phase 6: Code Review (代码审核)

Mandatory hard gate. SE Agent is reviewer. Any FAIL triggers a challenge against Dev Agent.

### Checklist (8 items)

**CR-1: Requirement Completeness** — Every FR in PRD/Dev Story implemented; every AC in test plan addressed; no TODO/FIXME/stubs.
  Challenge basis: cite FR-XXX or AC-XXX missing/incomplete.

**CR-2: Dev Story Alignment** — Every task implemented; file paths, signatures, structures match Dev Story; deviations documented.
  Challenge basis: cite TASK-XXX misaligned.

**CR-3: Standards Compliance** — Follows language-specific standards; consistent naming; correct file organization; no prohibited patterns.
  Challenge basis: cite standards chapter/section violated.

**CR-4: Architectural Integrity** — No circular dependencies; module rules respected; proper encapsulation; aligns with `docs/architecture.md`.
  Challenge basis: cite architecture.md section or dependency rule violated.

**CR-5: Correctness** — All code paths reachable; error handling complete; edge cases handled; no race conditions.
  Challenge basis: cite unhandled error scenario from Dev Story §5 or SE Design §3.1.6.

**CR-6: Test Coverage** — All new functions tested; Arrange-Act-Assert pattern; happy/error/boundary paths covered; no regressions.
  Challenge basis: cite missing/failing test case from test plan.

**CR-7: Security** — No hardcoded secrets; input validated at boundary; injection vectors closed; no `eval()`.
  Challenge basis: cite security standard from coding standards §Security.

**CR-8: No Omissions** — No commented-out code; no empty catch; no unreachable code; no debug leftovers; no dead imports.
  Challenge basis: cite file + line with omission.

### Review Flow
```
Phase 5 → SE Agent reviews → ALL PASS → APPROVED → Phase 7
                           → ANY FAIL → Challenge → Dev fixes → re-review → repeat
                           → Same item fails twice → Escalate to User
```

### Report Format
```markdown
# Code Review Report
- **Review ID:** CR-YYYY-NNN | **Reviewer:** SE Agent | **Date:** YYYY-MM-DD

## Summary
- Passed: N / Failed: N / Overall: APPROVED | CHALLENGED

## Checklist Results
### CR-X: Name — PASS/FAIL
- Issue / Basis / Impact / Suggested Fix (if FAIL)

## Resolution
- [ ] All FAIL items resolved → Proceed to Phase 7
```

---

## Framework Rules (priority order)

| Priority | Rule File | Purpose |
|----------|-----------|---------|
| 0 | `framework/workflows/challenge-mechanism.rule.md` | Challenge/audit — highest priority |
| 1 | `framework/workflows/plugin-extension.rule.md` | Plugin hooks — checked before each step |
| 2 | `framework/workflows/standard-lifecycle.rule.md` | 7-phase workflow + shortcut |
| 3 | `framework/standards/coding-standards.<lang>.md` | Language-specific standards |
| 4 | `framework/standards/project-structure.template.md` | Project structure spec |
| 5 | `framework/standards/validation-standards.template.md` | Process validation spec |

---

## Agent Skills

| Skill File | Agent | Role |
|-------------|-------|------|
| `framework/agents/project-scanner.skill.md` | Scanner Agent | Project structure scanning, module mapping, shared resource detection, dependency analysis |
| `framework/agents/se-agent.skill.md` | SE Agent | Architecture design, SE requirements, Code Review |
| `framework/agents/dev-agent.skill.md` | Dev Agent | Story design, coding, challenge resolution |
| `framework/agents/test-agent.skill.md` | Test Agent | Test planning, process validation |

---

## Directory Structure

```
H5LifecycleTemplate/
├── CLAUDE.md                              # Master orchestrator
├── .claude/settings.json                  # Harness settings
├── framework/
│   ├── agents/                            # project-scanner, se-agent, dev-agent, test-agent .skill.md
│   ├── workflows/                         # standard-lifecycle, challenge-mechanism, plugin-extension .rule.md
│   ├── artifacts/                         # prd, se-design, dev-story, test-plan, architecture-doc .template.md
│   └── standards/                         # coding-standards (js/ts/java/python/template), project-structure, validation-standards
├── plugins/                               # Plugin extensions (example-plugin/)
└── docs/                                  # Generated documentation
    ├── .framework-init.lock               # Init marker + project metadata (JSON)
    ├── architecture.md                    # Shared — project architecture (Phase 0)
    ├── module-map.md                      # Shared — file-to-module map (Phase 0)
    ├── coding-standards.md                # Shared — language standards (Phase 0)
    ├── project-structure.md               # Shared — directory conventions (Phase 0)
    ├── .scanner-report.json               # Shared — machine-readable scan data (Phase 0)
    ├── challenges/
    │   └── index.md                       # Global challenge index (all requirements)
    ├── modules/                           # Per-module docs (large projects, Phase 0)
    │   └── MOD-XXX.md
    └── requirements/                      # Requirement directory root
        ├── index.md                       # Requirement registry
        └── REQ-YYYY-NNN-{slug}/           # Isolated requirement directory
            ├── prd.md                     # Phase 1 artifact
            ├── se-design.md               # Phase 2 artifact
            ├── dev-story.md               # Phase 3 artifact
            ├── test-plan.md               # Phase 4 artifact
            ├── validation-report.md       # Phase 7 artifact
            ├── reports/                   # Phase completion reports (this req only)
            │   ├── phase-1-prd-report.md
            │   ├── phase-2-se-design-report.md
            │   ├── phase-3-story-design-report.md
            │   ├── phase-4-test-plan-report.md
            │   ├── phase-5-dev-coding-report.md
            │   ├── phase-6-code-review-report.md
            │   └── phase-7-validation-report.md
            └── challenges/                # Challenge records (this req only)
                └── CH-YYYY-NNN.md
```

---

## Execution Protocol

### 1. Determine Operating Mode
- No lock file → REJECT, instruct `@CLAUDE.md 初始化框架`
- Lock + re-init command → confirm, delete lock, re-run Phase 0
- Lock >30 days → warn, suggest re-init
- Lock + Dev Story → Entry B (Shortcut)
- Lock + PRD/default → Entry A (Full Lifecycle)

### 2. Create Requirement Directory
When starting a new lifecycle (Entry A or B for a new requirement):
1. Assign the next REQ-ID from `docs/requirements/index.md` (or REQ-YYYY-001 if first)
2. Derive slug from PRD title or Dev Story name
3. Create `docs/requirements/REQ-YYYY-NNN-{slug}/` with `reports/`, `challenges/` subdirectories
4. Register the requirement in `docs/requirements/index.md` with status ACTIVE, current phase = 1
5. All subsequent phases for this requirement write into this directory

### 3. Load Environment
Read `docs/architecture.md`, load coding standards, scan `plugins/` for hooks, verify framework rules accessible. Identify the current requirement directory.

### 4. Pre-Flight Checks
Resolve pending challenges for this requirement first. Execute `*-pre` plugin hooks. Validate input artifacts against templates.

### 5. Execute Current Phase
Load agent skill file → provide inputs/standards/architecture → agent produces primary artifact in the requirement directory → **agent performs proactive challenge check** (scan for CAT-1 requirement gaps and CAT-2 standards violations) → if issues found, raise challenges immediately → agent generates phase completion report in `{req-dir}/reports/` → run `*-post` hooks → validate output against template.

**Phase report is MANDATORY.** A phase is not complete until its report is written. If the phase produces no report, the orchestrator MUST NOT advance.

**Proactive challenge rule:** Agents MUST scan for CAT-1 and CAT-2 issues before declaring their phase complete. Issues found during execution should be raised as challenges immediately — do not wait for a downstream review phase to catch them. This is the framework's primary self-correction mechanism.

### 6. Handoff or Halt
Reviewer accepts → advance phase, update `docs/requirements/index.md` current phase. Reviewer challenges → **create challenge record** in `{req-dir}/challenges/CH-YYYY-NNN.md`, update `docs/challenges/index.md`, pause lifecycle, route to challenged agent. Validation fails → return for revision.

### 7. Challenge Resolution
Challenged agent reads challenge record → implements fix → updates challenge Resolution section → sets status RESOLVED. Challenger verifies and closes or re-opens. Every round is appended to the challenge record. After resolution, the phase that triggered the challenge is re-executed and a new phase report is generated.

### 8. Code Review Gate (before Phase 7)
Invoke SE Agent in Code Review mode → execute 8-item checklist → APPROVED → Phase 7; CHALLENGED → create challenge record → fix → re-review; twice-failed → escalate.

### 9. Completion
When Phase 7 passes: mark requirement COMPLETED in `docs/requirements/index.md`. Report: phases completed, files changed/created, test results, review status. All artifacts, reports, and challenges are self-contained in the requirement directory.

---

## Challenge Mechanism Summary

Detailed rules in `framework/workflows/challenge-mechanism.rule.md`. Highest priority. **Challenges are the framework's primary iteration mechanism** — agents are expected to raise challenges proactively, not just react to them.

### Challenge as Iteration Driver

The framework treats challenges as a positive, expected behavior — not a failure signal. Every agent MUST actively look for issues during their work and raise challenges when found. This creates a self-correcting development loop.

```
Agent executes phase → discovers issue → raises challenge → challenged agent fixes → re-execute → advances
                                              ↑
                               Proactive detection (REQ gaps, standards violations)
```

### Two Proactive Challenge Categories

#### CAT-1: Requirement Gap Challenge (需求遗漏)
Raised when an agent discovers a functional requirement is missing, incomplete, or contradictory.

**Trigger conditions:**
- PRD references a feature without specifying behavior → challenge against PRD author
- SE Design omits a module needed to fulfill a PRD requirement → SE Agent vs PRD
- Dev Story task list misses a necessary implementation step → Dev Agent vs SE Agent
- Test Plan has no test case for a documented requirement → Test Agent vs Dev Agent
- Code implements a feature not documented in any upstream artifact → SE Agent vs Dev Agent (Phase 6)

**Challenge format for REQ gaps:**
```
Basis: "FR-XXX specifies {feature} but no corresponding {code/design/test} exists"
Evidence: Missing artifact reference + impacted downstream phases
Impact: {what breaks if this gap is not closed}
Suggested Fix: "Add {artifact} covering {requirement}"
```

#### CAT-2: Standards Violation Challenge (编码规范违反)
Raised when an agent discovers code or design that violates the project's coding standards.

**Trigger conditions:**
- Scanner detects file naming/organization inconsistent with `docs/coding-standards.md` §0 → Scanner vs Dev Agent
- SE Design specifies a module structure violating `docs/project-structure.md` → Dev Agent vs SE Agent
- Code uses prohibited patterns from standards "禁止事项" → any agent vs Dev Agent
- Code organization doesn't follow detected conventions (Phase F) → Scanner/SE Agent vs Dev Agent
- New file placed in wrong directory per module organization rules (§7.4) → any agent vs Dev Agent

**Challenge format for standards violations:**
```
Basis: "coding-standards.<lang>.md §X.Y: {rule} — {violation description}"
Evidence: file:line with actual vs expected
Impact: {maintainability risk, inconsistency with rest of codebase}
Suggested Fix: "Change {X} to {Y} per standard §X.Y"
```

### Challenge Rules
1. Any agent can challenge another's output by citing specific standard/clause violation
2. Challenged agent MUST stop and resolve challenge first
3. Valid challenge requires: target, basis (specific citation), evidence, impact, suggested fix
4. Invalid challenges (no cited basis, personal preference) are rejected
5. Deadlocked challenges escalate to user
6. Code Review challenges (Phase 6) are system-generated from the 8-item checklist
7. **Proactive challenges (CAT-1, CAT-2) are expected and encouraged** — agents should raise them immediately upon discovery, not wait for a review phase

**Challenge documentation (MANDATORY):**
- Every challenge is recorded in the requirement's `challenges/CH-YYYY-NNN.md` using the format defined in §Challenge Documentation
- `docs/challenges/index.md` (global) is updated on every challenge state change
- No phase transition is allowed while an OPEN challenge exists against that requirement
- Challenge records are permanent artifacts — never deleted, only status-updated
- Challenge category (CAT-1/CAT-2) must be recorded in the challenge document

## Plugin Extension Summary

Detailed rules in `framework/workflows/plugin-extension.rule.md`. 12 hook points:

| Hook | Trigger | Type |
|------|---------|------|
| `prd-post` | After PRD provided | Post |
| `se-pre` / `se-post` | Before/after SE Design | Pre/Post |
| `story-pre` / `story-post` | Before/after Story Design | Pre/Post |
| `test-pre` / `test-post` | Before/after Test Plan | Pre/Post |
| `dev-pre` / `dev-post` | Before/after Development | Pre/Post |
| `review-pre` / `review-post` | Before/after Code Review | Pre/Post |
| `validation-post` | After Validation passes | Post |

---

The framework is language-agnostic at its core. Language-specific standards (JS, TS, Java, Python) are pre-built; other languages are generated during Phase 0 from the generic template.
