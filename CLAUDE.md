# H5 Lifecycle Framework

AI-agent-driven software development lifecycle framework. Orchestrates specialized agents through PRD→SE→Story→Test→Dev→Code Review→Validation workflow with built-in challenge/audit and plugin extension mechanisms. Supports a Dev-Story shortcut path.

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                       CLAUDE.md (Orchestrator)                    │
├──────────────────────────────────────────────────────────────────┤
│  Agents Layer       │  SE Agent  │  Dev Agent  │  Test Agent     │
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

### Step 0.2: Comprehensive File Scanning
**Scan scope:** `src/`, `lib/`, `app/`, `test*/`, `__tests__/`, `spec/`, `config/`, `scripts/`, `static/`, `public/`, `assets/`, root config files.
**Exclude:** `.git/`, `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, `target/`, `out/`, `.next/`, `coverage/`, `*.log`, `*.lock`, `__pycache__/`, `.idea/`, `.vscode/`, `framework/`, `plugins/`.

Detect organizational pattern (feature-based / layer-based / hybrid / flat / empty). Classify each file using decision tree (first match wins): test → asset → script → config → middleware → controller → repository → model → utility → entry → component → service → unknown. Record line count, barrel-export status, complexity flags (>200 lines).

**Empty project:** If 0 source files, skip Steps 0.3-0.4, go to Step 0.5 minimal mode.

### Step 0.3: Identify Module Boundaries
Group source files into modules using algorithm (in order): (1) explicit module markers (barrel `index.*` with exports), (2) framework-implied (`__init__.py`, Java packages), (3) feature directories (2+ files, 2+ different classifications), (4) layer directories, (5) single-file outliers. Infer module type: `utility`, `infrastructure`, `ui`, `core`, `business`.

For each module record: ID, name, type, path, file count, line count, per-file classification, public API surface. Produce summary inventory grouped by type.

### Step 0.4: Analyze Module Dependencies
Parse imports (ESM/CJS/dynamic for JS/TS; `import` for Java; `from X import Y` for Python). Resolve paths. Record dependency edges between modules. Classify weight: heavy (5+ files), medium (2-4), light (1), dynamic (lazy import). Classify direction: unidirectional (healthy), bidirectional (review), circular (VIOLATION).

**Architecture rules:** Core→Business (ERROR), Core→UI (ERROR), Infrastructure→Business (ERROR), Utility→Any (WARNING), Circular (ERROR). Generate dependency matrix.

### Step 0.5: Generate Documentation
Produce 4 files in `docs/` (2 for empty projects):

| Document | Template |
|----------|----------|
| `docs/architecture.md` | `framework/artifacts/architecture-doc.template.md` |
| `docs/module-map.md` | Cross-reference + per-module details |
| `docs/coding-standards.md` | Copy/adapt from `framework/standards/coding-standards.<lang>.md` |
| `docs/project-structure.md` | `framework/standards/project-structure.template.md` |

`architecture.md` must include: project overview (Step 0.1), module architecture (Step 0.3), dependency graph with ASCII diagram (Step 0.4), data architecture, API architecture, cross-cutting concerns, evolution history. Mandatory ASCII diagrams: high-level architecture + module dependency graph.

`module-map.md` must include: summary, file-to-module cross-reference table (every source file), per-module detail sections (type, path, files, public API, depends on, depended by). For >50 source files, detail sections can be collapsed with note to `docs/modules/MOD-XXX.md`.

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
| Phase | Agent | Input | Output | Template | Reviewer |
|-------|-------|-------|--------|----------|----------|
| 1. PRD | User | Business needs | `docs/prd.md` | `framework/artifacts/prd.template.md` | — |
| 2. SE Design | SE Agent | PRD, architecture docs | `docs/se-design.md` | `framework/artifacts/se-design.template.md` | Dev Agent |
| 3. Story Design | Dev Agent | SE Design, PRD | `docs/dev-story.md` | `framework/artifacts/dev-story.template.md` | SE Agent |
| 4. Test Plan | Test Agent | PRD, SE Design, Dev Story | `docs/test-plan.md` | `framework/artifacts/test-plan.template.md` | Dev Agent + User |
| 5. Dev Coding | Dev Agent | Dev Story, Test Plan | Source code | Language-specific standards | SE Agent (Phase 6) |
| 6. Code Review | SE Agent | Source code, Dev Story, PRD, Test Plan | Review Report | §Code Review Checklist | — (gate to Phase 7) |
| 7. Validation | Test Agent | Source code, Test Plan, Review Report | `docs/validation-report.md` | `framework/standards/validation-standards.template.md` | — |

### Phase Transition Rules
1. Before each transition, check plugin hooks and pending challenges
2. A phase is complete only when quality gate is satisfied AND reviewer accepts
3. No phase skipping in full lifecycle; shortcut has defined entry points
4. **Phase 6 is a hard gate** — Phase 7 cannot begin until review is APPROVED

### State Tracking
**Tier 0 — Init Gate:** `docs/.framework-init.lock` absent = ALL lifecycle requests rejected.
**Tier 1 — Phase State** (tracked via document presence in `docs/`):
| Document | Indicates |
|----------|-----------|
| `prd.md` | Phase 1 complete |
| `se-design.md` | Phase 2 complete |
| `dev-story.md` | Phase 3 complete (or shortcut entry) |
| `test-plan.md` | Phase 4 complete |
| Git commit with source changes | Phase 5 complete |
| `code-review-report.md` with APPROVED | Phase 6 complete |
| `validation-report.md` with all pass | Phase 7 complete |

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
| `framework/agents/se-agent.skill.md` | SE Agent | Architecture design, SE requirements, Code Review |
| `framework/agents/dev-agent.skill.md` | Dev Agent | Story design, coding, challenge resolution |
| `framework/agents/test-agent.skill.md` | Test Agent | Test planning, process validation |

---

## Directory Structure

```
H5LifecycleTemplate/
├── CLAUDE.md                         # Master orchestrator
├── .claude/settings.json             # Harness settings
├── framework/
│   ├── agents/                       # se-agent, dev-agent, test-agent .skill.md
│   ├── workflows/                    # standard-lifecycle, challenge-mechanism, plugin-extension .rule.md
│   ├── artifacts/                    # prd, se-design, dev-story, test-plan, architecture-doc .template.md
│   └── standards/                    # coding-standards (js/ts/java/python/template), project-structure, validation-standards
├── plugins/                          # Plugin extensions (example-plugin/)
└── docs/                             # Generated: .framework-init.lock, architecture.md, module-map.md,
    │                                 #   coding-standards.md, project-structure.md, prd.md, se-design.md,
    │                                 #   dev-story.md, test-plan.md, code-review-report.md, modules/
```

---

## Execution Protocol

### 1. Determine Operating Mode
- No lock file → REJECT, instruct `@CLAUDE.md 初始化框架`
- Lock + re-init command → confirm, delete lock, re-run Phase 0
- Lock >30 days → warn, suggest re-init
- Lock + Dev Story → Entry B (Shortcut)
- Lock + PRD/default → Entry A (Full Lifecycle)

### 2. Load Environment
Read `docs/architecture.md`, load coding standards, scan `plugins/` for hooks, verify framework rules accessible.

### 3. Pre-Flight Checks
Resolve pending challenges first. Execute `*-pre` plugin hooks. Validate input artifacts against templates.

### 4. Execute Current Phase
Load agent skill file → provide inputs/standards/architecture → agent produces output per template → run `*-post` hooks → validate output.

### 5. Handoff or Halt
Reviewer accepts → advance phase. Reviewer challenges → pause, route to challenged agent. Validation fails → return for revision.

### 6. Code Review Gate (before Phase 7)
Invoke SE Agent in Code Review mode → execute 8-item checklist → APPROVED → Phase 7; CHALLENGED → fix → re-review; twice-failed → escalate.

### 7. Completion
Report: phases completed, files changed/created, test results, review status, validation report location.

---

## Challenge Mechanism Summary

Detailed rules in `framework/workflows/challenge-mechanism.rule.md`. Highest priority:

1. Any agent can challenge another's output by citing specific standard/clause violation
2. Challenged agent MUST stop and resolve challenge first
3. Valid challenge requires: target, basis (specific citation), evidence, impact, suggested fix
4. Invalid challenges (no cited basis, personal preference) are rejected
5. Deadlocked challenges escalate to user
6. Code Review challenges are system-generated — SE Agent issues them automatically on checklist failures

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
