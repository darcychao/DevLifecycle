# Requirement Directory Model (需求目录模型)

## Overview

Each requirement/feature gets an isolated directory under `docs/requirements/`. All lifecycle artifacts, reports, and challenges for that requirement live within it. Shared project-level docs (architecture, standards, module map) remain at `docs/` root.

## Directory Layout

```
docs/
├── architecture.md                   # Shared — project-wide
├── module-map.md                     # Shared — project-wide
├── coding-standards.md               # Shared — project-wide
├── project-structure.md              # Shared — project-wide
├── public-method-catalog.md          # Shared — exported method inventory
├── constant-catalog.md               # Shared — constants inventory
├── terminology-glossary.md           # Shared — domain terminology
├── ux-guidelines.md                  # Shared — UX guidelines (if project has UI)
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

## Requirement Index (`docs/requirements/index.md`)

```markdown
# Requirement Index
| REQ ID | Name | Status | Current Phase | Created | Completed |
|--------|------|--------|---------------|---------|-----------|
| REQ-2026-001 | user-auth | Phase 5 | 2026-06-06 | - |
| REQ-2026-002 | payment | Phase 2 | 2026-06-06 | - |
```

**Status values:** `ACTIVE`, `COMPLETED`, `ABANDONED`, `ON-HOLD`.

## Requirement Lifecycle

1. **Creation:** When a new PRD is provided (Entry A) or a Dev Story is provided (Entry B), the orchestrator assigns a REQ-ID and creates the requirement directory
2. **Execution:** All phases for this requirement write into its directory
3. **Completion:** When Phase 7 passes, the requirement is marked COMPLETED in the index
4. **Parallel requirements:** Multiple requirements can be ACTIVE simultaneously; each has its own directory and independent phase state

## REQ-ID Assignment

- Format: `REQ-YYYY-NNN` (year + 3-digit sequential counter)
- Counter resets per year, padded to 3 digits (001, 002, ...)
- Slug is derived from the PRD title or Dev Story name, lowercased with hyphens

## UX Guidelines (`docs/ux-guidelines.md`)

When the project has a UI component (detected during Phase 0), `docs/ux-guidelines.md` is generated. This document captures:
- Detected UI framework conventions (component naming, styling approach, layout patterns)
- Mandatory UX constraints that PRD/Story authors must provide when UI design is involved

See `framework/workflows/ux-constraint.rule.md` for the enforcement protocol.
