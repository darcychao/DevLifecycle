# Execution Protocol (执行协议)

## 1. Determine Operating Mode

- No lock file → REJECT, instruct `@CLAUDE.md 初始化框架`
- Lock + re-init command → confirm, delete lock, re-run Phase 0
- Lock >30 days → warn, suggest re-init
- Lock + Dev Story → Entry B (Shortcut)
- Lock + PRD/default → Entry A (Full Lifecycle)

## 2. Create Requirement Directory

When starting a new lifecycle (Entry A or B for a new requirement):

1. Assign the next REQ-ID from `docs/requirements/index.md` (or REQ-YYYY-001 if first)
2. Derive slug from PRD title or Dev Story name
3. Create `docs/requirements/REQ-YYYY-NNN-{slug}/` with `reports/`, `challenges/` subdirectories
4. Register the requirement in `docs/requirements/index.md` with status ACTIVE, current phase = 1
5. All subsequent phases for this requirement write into this directory

## 3. Load Environment

Read `docs/architecture.md`, `docs/module-map.md` (including the keyword-to-module index), `docs/public-method-catalog.md` (for method reuse checks), `docs/constant-catalog.md` (for constant reuse and registration), and `docs/terminology-glossary.md` (for domain term consistency). If the project has UI (`ui_stack` present in lock file), also load `docs/ux-guidelines.md`. Load coding standards, scan `plugins/` for hooks, verify framework rules accessible. Identify the current requirement directory. For code-level work:
- Consult the keyword index in module-map.md to locate relevant modules and their capabilities.
- Check public-method-catalog.md for existing methods before writing new functions.
- Check constant-catalog.md for existing constants before defining new ones.
- Check terminology-glossary.md to use consistent domain terminology.
- Check ux-guidelines.md for UX conventions before implementing UI components (if project has UI).

## 4. Pre-Flight Checks

Resolve pending challenges for this requirement first. Execute `*-pre` plugin hooks. Validate input artifacts against templates. If the project has UI and the current phase involves UI design (Phase 1 PRD, Phase 3 Story Design), execute UX constraint check per `framework/workflows/ux-constraint.rule.md`.

**Catalog compliance pre-check:** Verify `docs/public-method-catalog.md`, `docs/constant-catalog.md`, and `docs/terminology-glossary.md` exist and are readable. If any catalog is missing (first run or corrupted), treat as WARNING — agents proceed but MUST self-declare new symbols in the phase report for future catalog inclusion.

## 5. Execute Current Phase

Load agent skill file → provide inputs/standards/architecture → agent produces primary artifact in the requirement directory → **agent performs proactive challenge check** (scan for CAT-1 requirement gaps and CAT-2 standards violations) → if issues found, raise challenges immediately → agent generates phase completion report in `{req-dir}/reports/` → run `*-post` hooks → validate output against template.

**Phase report is MANDATORY.** A phase is not complete until its report is written. If the phase produces no report, the orchestrator MUST NOT advance.

**Proactive challenge rule:** Agents MUST scan for CAT-1 and CAT-2 issues before declaring their phase complete. Issues found during execution should be raised as challenges immediately — do not wait for a downstream review phase to catch them. This is the framework's primary self-correction mechanism.

**Challenge blocking rule:** If any proactive challenge (CAT-1, CAT-2, CAT-2a/2b/2c) is raised during this phase, the phase status becomes CHALLENGED. The phase report MUST list all challenges raised with their CH-YYYY-NNN IDs. Phase transition is BLOCKED until all challenges are RESOLVED. The orchestrator MUST NOT advance to the next phase while any OPEN challenge exists against the current requirement.

**UX constraint enforcement:** When the phase involves UI design and the project is marked as having UI, the orchestrator enforces UX constraint checks. See `framework/workflows/ux-constraint.rule.md` for the enforcement protocol.

## 6. Handoff or Halt

Reviewer accepts → advance phase, update `docs/requirements/index.md` current phase. Reviewer challenges → **create challenge record** in `{req-dir}/challenges/CH-YYYY-NNN.md`, update `docs/challenges/index.md`, pause lifecycle, route to challenged agent. Validation fails → return for revision.

**Catalog violation blocking:** If the phase produced new public methods, constants, or terminology symbols without declaring them in the phase report's "New Symbol Declaration" section → REJECT handoff → return to agent for declaration → re-execute phase. Phase 5.5 and Phase 6.5 catalog gates are mandatory — skipping them constitutes a CAT-2 violation and blocks phase transition.

## 7. Challenge Resolution

Challenged agent reads challenge record → implements fix → updates challenge Resolution section → sets status RESOLVED. Challenger verifies and closes or re-opens. Every round is appended to the challenge record. After resolution, the phase that triggered the challenge is re-executed and a new phase report is generated.

## 8. Code Review Gate (before Phase 7)

Invoke SE Agent in Code Review mode → execute 8-item checklist → APPROVED → Phase 7; CHALLENGED → create challenge record → fix → re-review; twice-failed → escalate.

## 9. Completion

When Phase 7 passes: mark requirement COMPLETED in `docs/requirements/index.md`. Report: phases completed, files changed/created, test results, review status. All artifacts, reports, and challenges are self-contained in the requirement directory.
