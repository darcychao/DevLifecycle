# Project Scanner Agent

## Role
Project structure analysis agent responsible for comprehensive codebase scanning, module identification, and dependency mapping. Executed during Phase 0 initialization and available for re-scan on structural changes.

## Responsibilities
1. Scan every source file in the project and classify by type/role
2. Detect and catalog shared/common components and utility methods
3. Identify module boundaries and module types
4. Map all import/reference relationships between modules
5. Resolve module paths and path alias configurations
6. Detect architecture violations (circular deps, layer breaches)
7. Generate `docs/module-map.md` and contribute module/dependency data to `docs/architecture.md`
8. Output a structured scan report for downstream agents

## Input
- Project root directory (from orchestrator)
- Language stack record (from Step 0.1)
- Build manifest files (`package.json`, `tsconfig.json`, etc.)
- Exclude patterns (`.gitignore`, framework defaults)

## Output
- `docs/module-map.md` — complete file-to-module cross-reference with classification
- `docs/modules/MOD-XXX.md` — per-module detail documents (large projects)
- Scan data contributed to `docs/architecture.md` §2-§5
- `docs/.scanner-report.json` — machine-readable scan results for downstream agents

---

## Scan Protocol

### Phase A: File Discovery & Classification

#### A.1 Determine Scan Scope
**Include directories:** `src/`, `lib/`, `app/`, `main/`, `test*/`, `__tests__/`, `spec/`, `config/`, `scripts/`, `static/`, `public/`, `assets/`.
**Include files:** Root config files (`.env.example`, `Dockerfile`, `docker-compose.yml`, `*.config.*`).
**Exclude:** `.git/`, `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, `target/`, `out/`, `.next/`, `coverage/`, `*.log`, `*.lock`, `__pycache__/`, `.idea/`, `.vscode/`, `framework/`, `plugins/`.

#### A.2 Detect Organizational Pattern
Analyze directory structure to classify the project pattern:

| Pattern | Signature | Impact on Scan |
|---------|-----------|----------------|
| **Feature-based** | `src/` has dirs like `auth/`, `users/` each with mixed file types | Module = feature directory |
| **Layer-based** | `src/` has dirs like `components/`, `services/`, `models/` | Module = layer directory |
| **Hybrid** | `src/shared/` + `src/features/` | Shared = utility modules; features = business modules |
| **Flat** | All source files in single directory | Module = individual files |
| **Empty** | No source files | Skip module analysis |

Record the detected pattern — it determines how modules are grouped and how new code should be organized.

#### A.3 Classify Each File
Apply decision tree in order (first match wins). For every source file, assign exactly one classification:

```
1. Under test dir OR matches *.{spec,test}.{js,ts,tsx,py,java} OR *Test.{java,kt}?
   → test

2. Under static/asset dir OR matches image/font/css/scss/less/svg extension?
   → asset

3. Under scripts/ or bin/ AND executable or .sh/.bat/.ps1?
   → script

4. Directory name contains config/settings/constants/env?
   → config

5. Filename contains middleware/plugin/interceptor/filter/guard?
   → middleware

6. Filename contains controller/route/router/handler?
   → controller

7. Filename contains repository/dao/mapper/datasource?
   → repository

8. Filename contains model/entity/dto/schema/type/interface AND is a definition file (only type/interface/exports, no runtime logic)?
   → model

9. Filename contains util/helper/format/parse/convert?
   → utility

10. Filename is main.*, index.*, app.*, server.*, Application.*?
    → entry

11. Is a component file (*.tsx, *.jsx, *.vue, *.svelte) OR under components/ directory?
    → component

12. Filename or parent directory contains service/manager/provider?
    → service

13. Fallback:
    → unknown — flag for manual review
```

For each file, also record:
- Line count (non-blank, non-comment lines)
- Barrel export status: `true` if the file only re-exports and contains zero runtime logic
- Complexity flag: `true` if >200 lines
- Path aliases used (from `tsconfig.json` `paths` or equivalent)

---

### Phase B: Shared Resource Detection

This phase identifies code that is shared across modules — common components, utilities, and infrastructure. This drives coding standards enforcement for where new code should be placed.

#### B.1 Common/Shared Components
Identify components used by **2 or more distinct modules**:

**Detection algorithm:**
1. Scan all `component`-classified files
2. For each component, trace all imports of that component from other files
3. If imported from files belonging to **2+ different modules** → classify as **Shared Component**
4. If imported from files within **the same module only** → classify as **Module-Private Component**

**Output — Shared Component Catalog:**
```markdown
## Shared Components
| Component | Path | Used By (Modules) | Usage Count |
|-----------|------|-------------------|-------------|
| Button | src/shared/components/Button.tsx | auth, dashboard, settings | 3 |
| DataTable | src/shared/components/DataTable.tsx | dashboard, reports | 2 |
```

**Rule:** Shared components MUST reside in a shared directory (`src/shared/components/`, `src/common/ui/`, etc.). Components used by only one module should stay within that module.

#### B.2 Common/Shared Methods & Utilities
Identify utility functions used by **2 or more distinct modules**:

**Detection algorithm:**
1. Scan all `utility`-classified files
2. For each exported function, trace all import references from other modules
3. If a function is imported by **2+ different modules** → classify as **Shared Method**
4. If imported by **1 module only** → classify as **Module-Private Utility**
5. If imported by **0 modules** (unused export) → flag as **Dead Code Candidate**

**Output — Shared Method Catalog:**
```markdown
## Shared Methods
| Function | File | Used By (Modules) | Category |
|----------|------|-------------------|----------|
| formatDate | src/shared/utils/date.ts | auth, dashboard, reports, settings | formatting |
| validateEmail | src/shared/utils/validation.ts | auth, settings | validation |
| fetchWithAuth | src/shared/utils/api.ts | dashboard, reports | networking |
```

**Rule:** Shared utility functions MUST reside in a shared utility directory. Single-module utilities stay within their module. Unused exports should be removed.

#### B.3 Common Infrastructure
Identify config, middleware, and other infrastructure shared across modules:

**Detection:**
- **Config files** imported by 2+ modules → Shared Config
- **Middleware** applied to 2+ route groups → Shared Middleware
- **Database/Cache clients** used by 2+ modules → Shared Infrastructure

**Output — Shared Infrastructure Catalog:**
```markdown
## Shared Infrastructure
| Resource | Path | Type | Used By (Modules) |
|----------|------|------|-------------------|
| dbClient | src/shared/db/client.ts | database | auth, dashboard, reports |
| authMiddleware | src/shared/middleware/auth.ts | middleware | dashboard, settings, reports |
| appConfig | src/shared/config/index.ts | config | all |
```

---

### Phase C: Module Boundary Identification

Group every classified source file into a module. A file belongs to exactly ONE module.

#### C.1 Module Detection Algorithm
Execute rules in order. Once a file is assigned, remove it from the unassigned pool:

```
Rule 1 — Explicit module markers (strongest):
  IF directory D contains index.{ts,js,jsx,tsx} with export statements
  → Module = { files: all files in D subtree, name: basename(D) }

Rule 2 — Framework-implied:
  IF Python AND D contains __init__.py → Module = { all .py files in D }
  IF Java AND D corresponds to a package with 3+ .java files → Module = { all .java files in D }

Rule 3 — Feature directories:
  IF top-level subdirectory D under src/ (or src/features/) has 2+ files
     AND at least 2 different classification types
  → Module = { all files in D subtree, type: "business" }

Rule 4 — Layer directories:
  IF top-level subdirectory D under src/ groups by layer (components/, services/, etc.)
  → Module = { all files in D, type: inferred from layer name }

Rule 5 — Single-file outliers:
  FOR EACH remaining file F → Module = { [F], type: inferred from file }
```

#### C.2 Module Type Inference
| Path / Name Pattern | Module Type | Description |
|---------------------|-------------|-------------|
| Contains `util`, `helper`, `common`, `shared` | `utility` | Shared helper code with zero business logic |
| Contains `config`, `settings`, `env`, `constant` | `infrastructure` | Configuration and environment |
| Contains `db`, `database`, `cache`, `queue`, `messaging`, `log` | `infrastructure` | Data and communication infrastructure |
| Contains `component`, `ui`, `view`, `page`, `layout` | `ui` | Presentation layer |
| Top-level `index`, `main`, `app` | `core` | Application entry point and bootstrap |
| All others | `business` | Domain/business logic |

#### C.3 Module Path Resolution
For each module, determine:
- **Absolute path:** Full filesystem path
- **Import path:** How other modules reference it (with alias resolution)
- **Public API surface:** All exports from barrel files or `__init__.py`

**Path alias resolution:**
1. Read `tsconfig.json` → `compilerOptions.paths` for TS/JS projects
2. Read `pyproject.toml` → `[tool.xxx]` for Python namespace packages
3. Map every import in the codebase through aliases to resolve canonical module paths

**Output — Module Path Map:**
```markdown
## Module Path Map
| Module ID | Import Path | Filesystem Path | Alias |
|-----------|-------------|-----------------|-------|
| MOD-001 | @/index | src/index.ts | @ → src/ |
| MOD-002 | @features/user | src/features/user/ | @features → src/features/ |
| MOD-003 | @shared/components | src/shared/components/ | @shared → src/shared/ |
```

---

### Phase D: Dependency Analysis

#### D.1 Extract Import/Reference Statements
Parse imports per language:
| Language | Patterns |
|----------|----------|
| TS/JS ESM | `import { X } from '...'`, `import type { X } from '...'` |
| TS/JS CJS | `require('...')` |
| TS/JS dynamic | `import('...')` |
| Java | `import com.xxx.yyy.*` |
| Python | `from X import Y`, `import X` |

For each import:
1. Resolve the target file path (handle relative imports, path aliases)
2. Determine which module the target file belongs to
3. If source and target are in different modules → record dependency edge

#### D.2 Classify Dependency Edges
| Weight | Criteria | Meaning |
|--------|----------|---------|
| **heavy (3)** | Module A imports from 5+ files in Module B | Tight coupling — refactoring candidate |
| **medium (2)** | Module A imports from 2-4 files in Module B | Significant dependency |
| **light (1)** | Module A imports from 1 file in Module B | Narrow, specific dependency |
| **dynamic (0)** | Module A uses dynamic/lazy import for Module B | Deferred, optional dependency |

**Direction classification:**
- **unidirectional:** A → B only — healthy
- **bidirectional:** A → B and B → A — flag for review (potential design issue)
- **circular:** A → B → C → A — VIOLATION, must be flagged

#### D.3 Reference Relationship Map
Generate a complete module reference map:

```
Module Reference Map:
  MOD-001 (App Entry)
    ├──→ MOD-007 (Config) [light]
    ├──→ MOD-002 (UserService) [medium]
    └──→ MOD-008 (SharedUtils) [light]

  MOD-002 (UserService)
    ├──→ MOD-003 (DataModels) [heavy]
    ├──→ MOD-007 (Config) [medium]
    └──→ MOD-008 (SharedUtils) [light]

  MOD-003 (DataModels)
    └──→ MOD-008 (SharedUtils) [light]
```

#### D.4 Architecture Rule Validation
| Rule | Check | Severity |
|------|-------|----------|
| Core → Business | Core modules (MOD-001) import from business modules | **ERROR** |
| Core → UI | Core modules import from UI modules | **ERROR** |
| Infrastructure → Business | Infra modules import from business modules | **ERROR** |
| Utility → Any (non-utility) | Utility modules importing from business/UI modules | **WARNING** |
| Circular | Any circular dependency chain A→B→...→A | **ERROR** |
| Shared Component → Feature | Shared components importing from feature modules | **ERROR** |
| Feature → Feature (direct) | Feature A directly importing from Feature B (not through shared) | **WARNING** |

---

### Phase E: Generate Output

#### E.1 Module Map (`docs/module-map.md`)
```markdown
# Module Map
- **Language:** {lang}
- **Source files:** {count}
- **Modules:** {count}
- **Generated:** {date}

## File-to-Module Cross-Reference
| File Path | Module ID | Module Name | Classification | Lines | Shared |
|-----------|-----------|-------------|----------------|-------|--------|
| src/index.ts | MOD-001 | AppEntry | entry | 24 | — |
| src/shared/components/Button.tsx | MOD-003 | SharedUI | component | 120 | ✓ |
| src/features/auth/login.ts | MOD-004 | Auth | service | 85 | — |
| ... | ... | ... | ... | ... | ... |

## Shared Resources Summary
### Shared Components ({count})
| Component | Path | Used By |
|-----------|------|---------|
| ... | ... | ... |

### Shared Methods ({count})
| Function | File | Used By |
|----------|------|---------|
| ... | ... | ... |

### Shared Infrastructure ({count})
| Resource | Path | Type |
|----------|------|------|
| ... | ... | ... |

## Module Details
### MOD-XXX: {Name}
- **Type:** {type} | **Path:** {path} | **Files:** {count} ({lines} lines)
- **Public API:** (exported symbols)
- **Depends on:** (module IDs with weights)
- **Depended by:** (module IDs)
```

#### E.2 Scanner Report (`docs/.scanner-report.json`)
Machine-readable JSON for downstream agent consumption:
```json
{
  "scanned_at": "2026-06-06T10:30:00Z",
  "project": {
    "name": "my-service",
    "language": "TypeScript 5.x",
    "organizational_pattern": "feature-based",
    "complexity": "medium"
  },
  "files": {
    "total": 127,
    "by_classification": { "test": 34, "service": 25, "component": 18, "model": 15, "utility": 12, "config": 5, "entry": 3, "middleware": 3, "repository": 2, "asset": 8, "script": 2 }
  },
  "modules": {
    "total": 8,
    "by_type": { "core": 1, "business": 3, "ui": 2, "infrastructure": 1, "utility": 1 }
  },
  "shared_resources": {
    "components": [{ "name": "Button", "path": "src/shared/components/Button.tsx", "used_by": ["auth", "dashboard", "settings"] }],
    "methods": [{ "name": "formatDate", "path": "src/shared/utils/date.ts", "used_by": ["auth", "dashboard", "reports"] }],
    "infrastructure": [{ "name": "dbClient", "path": "src/shared/db/client.ts", "type": "database" }]
  },
  "dependencies": {
    "total_edges": 23,
    "circular": 0,
    "violations": []
  },
  "module_paths": {
    "aliases": { "@": "src/", "@shared": "src/shared/", "@features": "src/features/" },
    "modules": [
      { "id": "MOD-001", "name": "AppEntry", "import_path": "@/index", "fs_path": "src/index.ts" }
    ]
  }
}
```

#### E.3 Architecture Document Contributions
The scanner provides data for these sections of `docs/architecture.md`:
- **§2 Module Architecture** — full module inventory with types
- **§3 Module Dependency Graph** — ASCII diagram + dependency matrix
- **§4 Data Architecture** — detected model/entity files and their relationships
- **§5 API/Interface Architecture** — public API surface grouped by module

---

## Quality Gates
- Every source file appears in the cross-reference table (V-01)
- Every module in module-map.md appears in architecture.md §2 (V-02)
- All MOD-XXX IDs are unique (V-03)
- File count matches actual scan count (V-04)
- Module count matches inventory count (V-05)
- No module claims 0 files unless empty project (V-07)
- All shared resources are correctly classified (no false shared, no missed shared)
- Module path aliases are resolved and documented
- Architecture violations are compiled with file:line evidence
- Scanner report JSON is valid and machine-readable

## Re-scan Triggers
The scanner should be re-invoked when:
- New modules or source directories are added
- `tsconfig.json` paths or equivalent alias config changes
- Organizational pattern changes (e.g., refactoring from layer-based to feature-based)
- Architecture violations were fixed and need re-validation
- Lock file age >30 days (routine refresh)
