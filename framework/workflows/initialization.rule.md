# Phase 0: Framework Initialization Protocol (框架初始化协议)

## Overview

Phase 0 is a hard prerequisite. No lifecycle phase may execute before initialization completes. If `docs/.framework-init.lock` is absent, ALL lifecycle requests are REJECTED. To initialize:

```
@CLAUDE.md 初始化框架
```

## Enforcement Rules

1. Before ANY lifecycle operation, verify `docs/.framework-init.lock` exists. If absent, REJECT.
2. Re-initialization: only on explicit user request — delete lock, re-run from scratch.
3. Staleness: if lock >30 days old, suggest re-init but don't force.
4. Empty project: still execute Phase 0 (minimal scaffolding), still create lock.
5. Partial init forbidden: if any step fails, report, roll back docs, instruct user.

## Protocol Flow (8 sequential steps)

```
Step 0.1 → Step 0.2 → Step 0.3 → Step 0.4 → Step 0.5 → Step 0.6 → Step 0.7 → Step 0.8
Detect    Scan      Identify   Analyze    Generate   Validate   Classify   Create
Language  Files     Modules    Deps       Docs       & Report   Project    Lock
```

---

## Step 0.1: Detect Language Stack & Metadata

Scan project root for build manifests in priority order: `package.json` → `tsconfig.json` → `pom.xml` → `build.gradle` → `pyproject.toml` → `Cargo.toml` → `go.mod` → `*.csproj` → `Gemfile` → `CMakeLists.txt` → `pubspec.yaml`. Detect frameworks from dependencies (React/Vue/Angular/Express/Next/NestJS from `package.json`; Spring Boot/Quarkus from Java manifests; Django/Flask/FastAPI from Python manifests). Detect package manager from lock files. Extract project name, version, description, entry point, minimum runtime. Record complete language stack.

### Step 0.1.1: Detect UI Framework & UX Dependencies

When UI-related dependencies are detected, record all UX-relevant packages:

| Category | Common Packages to Detect |
|----------|--------------------------|
| **UI Framework** | `react`, `vue`, `@angular/core`, `svelte`, `solid-js`, `preact` |
| **Meta-Framework** | `next`, `nuxt`, `@nestjs/core`, `remix`, `sveltekit`, `astro` |
| **Component Library** | `@mui/material`, `antd`, `@shadcn/ui`, `element-plus`, `@chakra-ui`, `@mantine/core`, `radix-ui`, `daisyui`, `bootstrap` |
| **Styling** | `tailwindcss`, `styled-components`, `@emotion/react`, `sass`, `less`, `css-modules` (built-in), `vanilla-extract`, `stitches` |
| **State Management** | `redux`, `@reduxjs/toolkit`, `zustand`, `jotai`, `recoil`, `mobx`, `pinia`, `vuex`, `@ngrx/store` |
| **Routing** | `react-router-dom`, `@tanstack/react-router`, `vue-router`, `@angular/router`, `next/router` |
| **Animation** | `framer-motion`, `gsap`, `react-spring`, `@vueuse/motion`, `auto-animate` |
| **i18n/i10n** | `react-i18next`, `vue-i18n`, `@angular/localize`, `next-intl`, `formatjs` |
| **Accessibility** | `@axe-core/react`, `eslint-plugin-jsx-a11y`, `react-aria`, `radix-ui` (a11y focus) |
| **Design Tools** | `storybook`, `@ladle/react`, `histoire` |
| **Form Handling** | `react-hook-form`, `formik`, `vee-validate`, `@angular/forms`, `zod` |
| **UI Testing** | `@testing-library/react`, `@testing-library/vue`, `cypress`, `playwright`, `@playwright/test` |

**Output — UX Stack Record:**
```json
{
  "has_ui": true,
  "ui_framework": "React 18.x",
  "meta_framework": "Next.js 14.x",
  "component_library": "@mui/material 5.x",
  "styling": ["tailwindcss 3.x", "@emotion/react"],
  "state_management": "zustand",
  "routing": "next/router (App Router)",
  "i18n": "next-intl",
  "form_handling": "react-hook-form + zod",
  "ui_testing": ["@testing-library/react", "playwright"],
  "animation": "framer-motion",
  "design_tooling": "storybook"
}
```

**Multi-root/monorepo:** If multiple first-level subdirectories each have their own manifest, flag as monorepo — list each sub-project separately. Primary = repo root.

**No manifest:** List all file extensions, present top 3 to user for confirmation, then load/generate standards.

---

## Step 0.2-0.4: Project Structure Scan → Delegate to Scanner Agent

Steps 0.2 (File Scanning), 0.3 (Module Identification), and 0.4 (Dependency Analysis) are executed by the **Project Scanner Agent** (`framework/agents/project-scanner.skill.md`).

The scanner performs six phases:

| Phase | Description | Key Output |
|-------|-------------|------------|
| **A. File Discovery** | Scan all source files, classify each by type, detect organizational pattern | File inventory with classifications |
| **B. Shared Resource Detection** | Identify shared components, methods, and infrastructure used across 2+ modules | Shared component/method/infra catalogs |
| **C. Module Boundaries** | Group files into modules, resolve path aliases, determine public API surfaces | Module inventory with import paths |
| **D. Dependency Analysis** | Map all import edges, classify weights, detect circular/bidirectional deps | Dependency matrix + violation report |
| **E. Generate Output** | Produce `docs/module-map.md`, `docs/.scanner-report.json`, architecture contributions | Machine-readable scan data |
| **F. Generate Project-Level Standard** | Detect public methods, constant definitions, file organization, coding style, UX conventions from actual code | **Authoritative `docs/coding-standards.md` §0** — all agents must prioritize this over generic standards |

**Key differentiators from generic file scanning:**
- **Shared vs. module-private:** Every component/utility is classified by its usage scope (shared if imported by 2+ modules)
- **Path alias resolution:** All imports are resolved through `tsconfig.json` paths or equivalent to produce canonical module paths
- **Reference relationship map:** Full directed graph showing which modules reference which, with edge weights
- **Architecture rules for shared code:** Shared components MUST NOT import from feature modules; features SHOULD NOT directly import from other features

The orchestrator invokes this skill during Phase 0 and on structural changes. See `framework/agents/project-scanner.skill.md` for the complete protocol.

---

## Step 0.5: Generate Documentation

Produce files in `docs/` (minimal set for empty projects):

| Document | Source |
|----------|--------|
| `docs/architecture.md` | Data from Steps 0.1 + Scanner Agent (Phases C/D) |
| `docs/module-map.md` | Scanner Agent (Phase E.1) |
| `docs/coding-standards.md` | Copy/adapt from `framework/standards/coding-standards.<lang>.md` + Scanner Phase F §0 |
| `docs/project-structure.md` | `framework/standards/project-structure.template.md` + detected pattern |
| `docs/ux-guidelines.md` | Scanner Phase F §0.6 (if UI detected) |
| `docs/.scanner-report.json` | Scanner Agent (Phase E.2) — machine-readable scan data |
| `docs/modules/MOD-XXX.md` | Scanner Agent (Phase E.1) — follows `framework/artifacts/module-detail.template.md` |
| `docs/public-method-catalog.md` | Scanner Agent (Phase E.3) — follows `framework/artifacts/public-method-catalog.template.md` |
| `docs/constant-catalog.md` | Scanner Agent (Phase E.4) — follows `framework/artifacts/constant-catalog.template.md` |
| `docs/terminology-glossary.md` | Scanner Agent (Phase E.5) — follows `framework/artifacts/terminology-glossary.template.md` |

**Document content requirements:** See `framework/workflows/coding-standards-hierarchy.rule.md` for coding-standards.md generation rules. See individual artifact templates in `framework/artifacts/` for document structure requirements.

`architecture.md` must include: project overview (Step 0.1), module architecture (Step 0.3), dependency graph with ASCII diagram (Step 0.4), data architecture, API architecture, cross-cutting concerns, evolution history. Mandatory ASCII diagrams: high-level architecture + module dependency graph.

`module-map.md` must include: summary, keyword-to-module index, file-to-module cross-reference table, shared resources summary, per-module detail sections. For >50 source files, detail sections can be collapsed with note to `docs/modules/MOD-XXX.md`.

`public-method-catalog.md` is the authoritative reference for method reuse checks — agents MUST consult it before creating any new exported function.

`constant-catalog.md` is the authoritative reference for constant registration and reuse — agents MUST consult it before defining any new constant.

`terminology-glossary.md` is the authoritative reference for domain terminology consistency — agents MUST consult it before naming new types, functions, or concepts.

### Step 0.5.1: Generate UX Guidelines Document

When UX dependencies are detected (Step 0.1.1), generate `docs/ux-guidelines.md`:

```markdown
# UX Guidelines (用户体验规范)

## 1. UI Technology Stack
[Detected framework, libraries, and versions from Step 0.1.1]

## 2. Component Architecture
[Detected component patterns — file structure, naming, props conventions]

## 3. Styling Conventions
[Detected styling approach — class naming, theming, responsive breakpoints]

## 4. Layout Patterns
[Detected layout conventions — container, grid, spacing]

## 5. Accessibility Standards
[Detected a11y patterns — ARIA, keyboard nav, screen reader support]

## 6. Design Tokens & Theming
[Detected design tokens — colors, typography, spacing]

## 7. UX Constraint Requirements
[When PRD or Dev Story involves UI design, users MUST provide:]
- Interaction flow descriptions
- Visual references or design files
- Responsive design requirements
- Form interaction specifications
- Feedback mechanism details (loading, empty, error states)
```

---

## Step 0.6: Validate Documentation

Cross-validation checks (any ERROR halts Phase 0):

### Core Validation (ERROR)
| ID | Check | Severity |
|----|-------|----------|
| V-01 | Every source file appears in module-map.md | ERROR |
| V-02 | Every module in module-map.md appears in architecture.md §2 | ERROR |
| V-03 | All MOD-XXX IDs unique | ERROR |
| V-04 | File count matches scan count | ERROR |
| V-05 | Module count matches inventory | ERROR |
| V-07 | No module claims 0 files (unless empty) | ERROR |
| V-09 | All referenced standards files exist | ERROR |
| V-13 | Keyword index in module-map.md covers all modules | ERROR |
| V-15 | Every exported method appears in public-method-catalog.md | ERROR |
| V-16 | Method count in public-method-catalog.md matches B.4 inventory count | ERROR |
| V-17 | Every constant appears in constant-catalog.md | ERROR |
| V-18 | Constant count in constant-catalog.md matches B.5 inventory count | ERROR |
| V-21 | All cross-references within catalogs are internally consistent | ERROR |

### UX-Specific Validation
| ID | Check | Severity |
|----|-------|----------|
| V-23 | If UI framework detected, ux-guidelines.md exists | ERROR |
| V-24 | If UI framework detected, coding-standards.md §0.6 populated | ERROR |
| V-25 | If UI framework detected, UX stack record in .scanner-report.json | WARNING |
| V-26 | UX conventions coverage ≥70% for detected UI patterns | WARNING |

### Quality Validation (WARNING)
| ID | Check | Severity |
|----|-------|----------|
| V-06 | Dependency count reasonable | WARNING |
| V-08 | No bidirectional edges | WARNING |
| V-10 | ASCII diagrams present and correct | WARNING |
| V-11 | Every module has at least 3 keyword tags | WARNING |
| V-12 | Every module has at least 1 functional capability label | WARNING |
| V-14 | Per-module detail docs follow `module-detail.template.md` | WARNING |
| V-19 | Terminology glossary has at least 10 terms for non-empty projects | WARNING |
| V-20 | Domain cluster map has at least 1 cluster for projects with >=3 modules | WARNING |
| V-22 | Shared constants in constant-catalog.md match shared_resources cross-check | WARNING |

Compile architecture violations from Step 0.4.3. Output standardized init summary.

---

## Step 0.7: Classify Project Complexity

| Category | Files | Modules | Behavior |
|----------|-------|---------|----------|
| Empty | 0 | 0 | Minimal docs, standards only |
| Small | 1-30 | 1-5 | All details inline |
| Medium | 31-100 | 6-15 | All inline, architecture may split sub-sections |
| Large | 101-500 | 16-50 | Split modules to `docs/modules/MOD-XXX.md` |
| Very Large | 500+ | 50+ | Architecture split into sub-documents |

---

## Step 0.8: Create Lock File

Write `docs/.framework-init.lock` as JSON with:

```json
{
  "initialized_at": "YYYY-MM-DD HH:MM",
  "framework_version": "1.0",
  "project_name": "...",
  "language_stack": { ... },
  "framework": "...",
  "ui_stack": { "has_ui": true, "ui_framework": "...", "component_library": "...", ... },
  "organizational_pattern": "feature-based",
  "complexity": "medium",
  "source_file_count": 127,
  "module_count": 8,
  "circular_dependencies": 0,
  "architecture_violations": [],
  "generated_docs": ["architecture.md", "module-map.md", "coding-standards.md", "ux-guidelines.md", ...]
}
```

The `ui_stack` field is included only when UI dependencies are detected. It serves as the marker that UX constraint rules apply to this project.
