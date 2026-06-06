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

#### B.4 Public Method Inventory

This phase catalogs ALL exported public methods across the project, not just shared ones (B.2). Unlike B.2 which filters to "imported by 2+ modules", this is a comprehensive inventory for downstream reuse checks.

**Detection algorithm:**
1. Scan all files classified as `service`, `utility`, `controller`, `middleware`, `repository`, or `entry`
2. For each file, extract all exported function/method declarations:
   - `export function name(...)` — named function export
   - `export const name = (...) =>` — arrow function assigned to exported const
   - `export async function name(...)` — async function export
   - `export default function name(...)` — default function export
   - Class methods with `public` modifier (TypeScript)
3. For each exported method, record:
   - Name (the exported symbol name)
   - Full signature (parameters with types, return type)
   - Classification (derived from verb prefix — see table below)
   - Defining module (resolved via Phase C module grouping)
   - Source file and line number
   - Async/sync indicator
4. Determine consumers:
   - Scan all import statements across the codebase (using Phase D import graph data)
   - Record each consumer file path and its parent module ID
   - If imported by 2+ modules → `is_shared: true`
   - If imported by 0 files → `is_dead_code: true`
5. Flag methods with zero consumers as "Dead Code Candidates"

**Classification rules** (applied per-method, based on verb prefix):
| Verb Pattern | Classification |
|-------------|----------------|
| `fetch*`, `send*`, `request*`, `upload*`, `download*`, `api*` | API Communication |
| `create*`, `update*`, `delete*`, `save*`, `insert*`, `remove*` | CRUD Operations |
| `validate*`, `sanitize*`, `check*`, `assert*`, `verify*` | Validation |
| `format*`, `parse*`, `transform*`, `convert*`, `serialize*` | Data Transformation |
| `init*`, `setup*`, `bootstrap*`, `start*`, `stop*`, `destroy*` | Lifecycle Management |
| `subscribe*`, `emit*`, `dispatch*`, `notify*`, `publish*`, `handle*` | Event Handling |
| `calculate*`, `compute*`, `aggregate*`, `sum*`, `average*` | Computation |
| `render*`, `display*`, `show*`, `draw*`, `mount*` | UI Rendering |
| `get*`, `find*`, `query*`, `list*`, `search*`, `lookup*` | Data Retrieval |
| All others | Other |

**Output structure** feeds into `docs/public-method-catalog.md` and `.scanner-report.json` → `public_methods`.

---

#### B.5 Constant Inventory

This phase catalogs ALL constants, enums, and as-const objects across the project for downstream reuse checks and unified naming enforcement.

**Detection algorithm:**
1. Scan all source files (excluding test files) for:
   - Top-level `const` declarations (module-scope and namespace-scope)
   - `enum` declarations (numeric and string enums)
   - `as const` object declarations
   - `readonly` array/tuple declarations
2. For each constant, record:
   - Name (fully qualified: `EnumName.MemberName` for enum members)
   - Kind (`const`, `enum`, `enum-member`, or `as-const`)
   - Value (literal value for primitives; type description for complex; member count for enums)
   - Location (file:line)
   - Defining module (resolved via Phase C)
   - Category (inferred from naming and location — see table below)
   - Naming case (`UPPER_SNAKE_CASE`, `PascalCase`, `camelCase`, `mixed`)
   - Export status (named export, default export, or not exported)
   - String prefix pattern (for string constants: extract the prefix before the first underscore)
3. Determine consumers:
   - Scan all import statements across the codebase (using Phase D import graph data)
   - Record each consumer file path and parent module ID
   - If imported by 2+ modules → `is_shared: true`
4. Count inline literals in business logic files (non-config, non-constant files):
   - Calculate percentage of statements that are inline literals
   - Flag files where >5% of statements are inline literals → Magic Value Report

**Category inference:**
| Condition | Category |
|-----------|----------|
| Name starts with `API_`, `ENDPOINT_`, `ROUTE_`, `URL_`, `PATH_` | API path |
| Name starts with `ERR_`, `ERROR_`, `ERRCODE_` | Error code |
| Name starts with `CFG_`, `CONFIG_`, `SETTING_`, `OPTION_` | Config key |
| `enum` keyword or `as const` with string members and noun name | Enum |
| Name starts with `MSG_`, `MESSAGE_` | Message template |
| Name starts with `FEATURE_`, `FLAG_`, `TOGGLE_` or boolean const | Feature flag |
| Name starts with `REGEX_`, `PATTERN_`, `RE_` | Regex pattern |
| Name starts with `DEFAULT_`, `FALLBACK_` | Default value |
| Numeric literal, no descriptive prefix or unit context | Magic number |
| Fallback | Other |

**Output structure** feeds into `docs/constant-catalog.md` and `.scanner-report.json` → `constants`.

---

#### B.6 Terminology Extraction

This phase extracts domain terminology from the codebase and builds a glossary with cross-references to modules, methods, and constants.

**Detection algorithm:**

**Step 1 — Extract terms from type/interface/class/enum names (strongest signal):**
Extract all exported type, interface, class, and enum names. Split PascalCase names into constituent words. These are domain nouns.
- `UserProfile` → `User`, `Profile`
- `OrderRepository` → `Order` (filter `Repository` as generic)
- `PaymentStatus` → `Payment`, `Status`

**Step 2 — Extract terms from function/method names:**
From the B.4 inventory, extract noun components by removing known verb prefixes.
- `createUser` → verb `create`, noun `User`
- `validateEmailAddress` → verb `validate`, noun `EmailAddress` → split to `Email`, `Address`

**Step 3 — Extract terms from directory names:**
Module directory basenames (after removing `src/`, `features/`, `shared/`).
- `src/features/auth/` → `auth`
- `src/shared/components/` → filter as too generic

**Step 4 — Filter and normalize:**
- Convert all to lowercase for the term index
- Remove English stop words (same list as keyword extraction)
- Remove generic programming terms: `service`, `controller`, `handler`, `repository`, `middleware`, `util`, `helper`, `manager`, `provider`, `factory`, `builder`, `adapter`, `wrapper`, `proxy`, `decorator`, `plugin`, `module`, `component`, `interface`, `type`, `class`, `function`, `method`, `data`, `input`, `output`, `result`, `response`, `request`, `error`, `event`, `callback`, `config`, `option`, `setting`, `param`, `context`

**Step 5 — Derive definitions:**
For each term, analyze symbols containing that term:
- If mapped to an `interface`/`type` → extract property names to understand the shape
  - `User { id, name, email, role }` → "A user entity with identity, contact, and role attributes"
- If appearing in function names with consistent verb patterns → derive purpose
  - `createUser`, `getUserById`, `deleteUser` → "A user entity supporting CRUD operations"
- If mapped to an `enum` → values inform the definition
  - `OrderStatus { Pending, Processing, Completed }` → "The lifecycle states of an order"
- Fallback: "Domain entity: {term}" if insufficient context

**Step 6 — Build relationships:**
- **Term → Related modules:** All modules where the term appears (in symbol names, directory names, or file names)
- **Term → Related terms:** Terms co-occurring with this term in the same module's keyword list or in compound symbol names across 2+ modules
- **Term → Related methods:** From B.4 inventory, methods whose names contain this term
- **Term → Related constants:** From B.5 inventory, constants whose names contain this term

**Step 7 — Build domain clusters:**
- Create a co-occurrence matrix: term A and term B co-occur if found in the same module OR in the same compound symbol name
- Group using connected-components: two terms are in the same cluster if they co-occur in >=2 modules, or appear together in >=3 compound symbol names
- Label each cluster with the most frequent capability label from the module where cluster terms are most concentrated

**Step 8 — Flag undefined terms:**
Terms found in only one symbol or one file with no structural context → list in "Terms with No Definition" section.

**Output structure** feeds into `docs/terminology-glossary.md` and `.scanner-report.json` → `terminology`.

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

## Keyword-to-Module Index

| Keyword | Module | Capability |
|---------|--------|------------|
| auth, login, token | MOD-003 (Auth) | Authentication & Session Management |
| user, profile, account | MOD-004 (User) | User CRUD Operations |
| format, date, time | MOD-008 (SharedUtils) | Date/Time Formatting Utilities |
| validate, sanitize, check | MOD-005 (Validators) | Input Validation & Sanitization |
| ... | ... | ... |

*Keywords extracted from exported symbol names (split on camelCase/PascalCase/snake_case), directory basenames, and file basenames. Stop words filtered. Sorted alphabetically.*

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
- **Keywords:** {comma-separated keyword list}
- **Functional Capabilities:** {capability label 1}, {capability label 2}, ...
- **Public API:** (exported symbols)
- **Depends on:** MOD-XXX [weight, N files], MOD-YYY [weight, N files], ...
- **Depended by:** MOD-ZZZ [weight, N files], ...
```

#### E.1.5 Keyword Extraction & Functional Capability Analysis

After completing the module details, the scanner extracts keyword tags and infers functional capability labels for each module. These enable downstream agents to quickly locate modules by functional keywords.

##### Keyword Extraction Algorithm

For each module, derive keywords from three sources:

**Source 1 — Exported symbol names:**
Split camelCase/PascalCase/snake_case names into constituent words:
- `getUserProfileById` → [`get`, `user`, `profile`, `by`, `id`]
- `OrderService` → [`order`, `service`]
- `MAX_RETRY_COUNT` → [`max`, `retry`, `count`]

**Source 2 — Parent directory basename:**
Use the module's directory basename as a keyword:
- `src/features/auth/` → `auth`
- `src/shared/utils/` → `utils`, `shared`

**Source 3 — File basenames (without extension):**
- `login-service.ts` → [`login`, `service`]
- `user-repository.ts` → [`user`, `repository`]

**Combine and filter:**
1. Merge all tokens, lowercase, de-duplicate
2. Remove English stop words: `get`, `set`, `from`, `to`, `by`, `with`, `and`, `the`, `is`, `on`, `of`, `in`, `for`, `a`, `an`, `it`, `at`, `or`, `as`, `be`, `not`, `this`, `that`, `has`, `have`, `do`, `does`, `will`, `can`, `all`, `new`, `use`, `used`, `into`, `its`
3. Sort alphabetically
4. Truncate to top 30 keywords (for modules with very large public API surfaces)

##### Functional Capability Labeling

For each module, infer 1-5 functional capability labels by analyzing the aggregate of exported symbol names:

**Step 1 — Extract verb prefixes** from exported function/method names:
| Verb Pattern | Category Label |
|-------------|----------------|
| `login`, `logout`, `authenticate`, `authorize`, `verify` | Authentication |
| `create*`, `update*`, `delete*`, `get*ById`, `save*` | CRUD Operations |
| `validate*`, `sanitize*`, `check*`, `verify*` | Validation |
| `format*`, `parse*`, `transform*`, `convert*` | Data Transformation |
| `fetch*`, `send*`, `request*`, `upload*`, `download*` | API Communication |
| `render*`, `display*`, `show*`, `draw*` | UI Rendering |
| `calculate*`, `compute*`, `aggregate*`, `sum*` | Computation |
| `subscribe*`, `emit*`, `dispatch*`, `notify*`, `publish*` | Event Handling |
| `save*`, `load*`, `persist*`, `cache*`, `read*`, `write*` | Data Persistence |
| `init*`, `setup*`, `bootstrap*`, `start*`, `stop*` | Lifecycle Management |

**Step 2 — Extract noun targets** from exported class/interface/type names:
| Noun Pattern | Domain Label |
|-------------|-------------|
| `User*`, `Account*`, `Profile*`, `Role*`, `Permission*` | User Management |
| `Order*`, `Cart*`, `Checkout*`, `Payment*`, `Transaction*` | Order Processing |
| `Product*`, `Inventory*`, `Catalog*`, `Item*`, `SKU*` | Product Catalog |
| `Config*`, `Settings*`, `Env*`, `Option*` | Configuration |
| `Log*`, `Metric*`, `Monitor*`, `Trace*`, `Event*` | Observability |
| `Notification*`, `Alert*`, `Email*`, `Message*` | Messaging & Notifications |
| `Report*`, `Analytics*`, `Dashboard*`, `Chart*` | Reporting & Analytics |
| `File*`, `Upload*`, `Download*`, `Storage*`, `Asset*` | File Management |
| `Api*`, `Client*`, `Service*`, `Endpoint*` | API/Service Layer |
| `Route*`, `Router*`, `Navigation*`, `Page*` | Routing & Navigation |

**Step 3 — Combine categories + domains** to produce capability labels (max 80 chars):
- Exports [`login`, `logout`, `getCurrentUser`, `AuthService`, `UserToken`]
  → "Authentication & Session Management"
- Exports [`formatDate`, `formatCurrency`, `parseISO`, `DateFormat`]
  → "Date/Time Formatting Utilities"
- Exports [`createUser`, `getUserById`, `updateUser`, `deleteUser`, `UserRepository`]
  → "User CRUD Operations"

**Fallback:** If no verb/noun patterns match, use the module type + name as the capability label (e.g., "Feature: Auth" for a business module, "Utility: SharedUtils" for a utility module).

##### Generating the Keyword Index Table

1. For each keyword, list all modules that match it
2. Group related keywords into a single row when they all point to the same module
3. Include the primary functional capability label beside each entry
4. Sort rows alphabetically by the first keyword in each group
5. If a module has no keywords (empty public API), list it with keyword "—" and capability "No Public API"

##### Quality Gates
- Every module has at least 3 keyword tags generated (WARNING if fewer)
- No English stop words appear in the keyword index
- Every module has at least 1 functional capability label (WARNING if none)
- Module with zero exports (empty public API) is flagged with capability "No Public API"

---

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
    "by_type": { "core": 1, "business": 3, "ui": 2, "infrastructure": 1, "utility": 1 },
    "details": [
      {
        "id": "MOD-001",
        "name": "AppEntry",
        "type": "core",
        "path": "src/",
        "file_count": 3,
        "total_lines": 156,
        "complexity": false,
        "keywords": ["app", "entry", "index", "main", "server", "start"],
        "capabilities": ["Application Bootstrap"],
        "exports": [
          { "name": "startServer", "kind": "function", "signature": "async (port: number): Promise<Server>", "keywords": ["start", "server"] }
        ],
        "dependencies": {
          "depends_on": [{ "module_id": "MOD-007", "weight": "light", "files_imported": 1 }],
          "depended_by": []
        }
      }
    ]
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
  "public_methods": {
    "total": 87,
    "by_classification": { "API": 12, "CRUD": 23, "VAL": 8, "QUERY": 18, "XFMR": 10, "LIFE": 5, "EVENT": 6, "COMP": 3, "UI": 0, "OTHER": 2 },
    "methods": [
      {
        "name": "createUser",
        "signature": "async (input: CreateUserInput): Promise<User>",
        "classification": "CRUD",
        "provider_module": "MOD-004",
        "provider_file": "src/features/user/services/user-service.ts",
        "line": 42,
        "is_async": true,
        "consumers": [
          { "module_id": "MOD-001", "files": ["src/index.ts"] },
          { "module_id": "MOD-006", "files": ["src/features/admin/user-admin.ts"] }
        ],
        "is_shared": true,
        "is_dead_code": false
      }
    ]
  },
  "constants": {
    "total": 134,
    "by_kind": { "const": 78, "enum": 8, "enum-member": 40, "as-const": 8 },
    "by_category": { "API": 15, "ERR": 22, "CFG": 30, "ENUM": 48, "MSG": 6, "FLAG": 5, "REGEX": 3, "DEFAULT": 4, "MAGIC": 8, "OTHER": 6 },
    "constants": [
      {
        "name": "API_BASE_URL",
        "kind": "const",
        "value": "'https://api.example.com'",
        "location": "src/shared/constants/api.ts:3",
        "defining_module": "MOD-008",
        "category": "API",
        "naming_case": "UPPER_SNAKE_CASE",
        "is_exported": true,
        "string_prefix": "API_",
        "consumers": [
          { "module_id": "MOD-004", "files": ["src/features/auth/login.ts"] },
          { "module_id": "MOD-005", "files": ["src/features/dashboard/api.ts"] }
        ],
        "is_shared": true
      }
    ],
    "magic_value_files": [
      { "file": "src/utils/validation.ts", "inline_literals": 12, "statement_pct": 8.2 }
    ]
  },
  "terminology": {
    "total_terms": 45,
    "domain_clusters": [
      {
        "label": "Authentication & Session Management",
        "terms": ["auth", "login", "token", "session", "credential"],
        "primary_module": "MOD-003"
      }
    ],
    "terms": [
      {
        "term": "user",
        "definition": "A user entity with identity (id, email), role assignment, and associated CRUD operations",
        "found_in_modules": ["MOD-001", "MOD-003", "MOD-004", "MOD-006"],
        "related_terms": ["auth", "profile", "role", "permission", "session"],
        "related_methods": ["createUser", "getUserById", "updateUser", "deleteUser", "validateUser"],
        "related_constants": ["USER_ROLES", "DEFAULT_USER_ROLE", "ERR_USER_NOT_FOUND"]
      }
    ]
  },
  "keyword_index": {
    "auth": [{ "module_id": "MOD-003", "module_name": "Auth", "capability": "Authentication & Session Management" }],
    "login": [{ "module_id": "MOD-003", "module_name": "Auth", "capability": "Authentication & Session Management" }],
    "user": [{ "module_id": "MOD-004", "module_name": "User", "capability": "User CRUD Operations" }, { "module_id": "MOD-003", "module_name": "Auth", "capability": "Authentication & Session Management" }]
  },
  "module_paths": {
    "aliases": { "@": "src/", "@shared": "src/shared/", "@features": "src/features/" },
    "modules": [
      { "id": "MOD-001", "name": "AppEntry", "import_path": "@/index", "fs_path": "src/index.ts" }
    ]
  }
}
```

#### E.3 Public Method Catalog (`docs/public-method-catalog.md`)

Generate the comprehensive public method catalog following `framework/artifacts/public-method-catalog.template.md`. Populate from the B.4 Public Method Inventory data.

Quality checks:
- Every exported method from B.4 appears in the catalog (Section 1 Method-to-Module Index)
- Classification is assigned to every method (fallback: `OTHER`)
- Consumer module lists are complete (all import references from Phase D resolved)
- Section 4 (Unused Exports) is populated from B.4 zero-consumer flags
- Section 1 (Method-to-Module) and Section 2 (Module-to-Methods) are internally consistent
- All consumer module IDs are valid (exist in `docs/module-map.md`)

#### E.4 Constant Catalog (`docs/constant-catalog.md`)

Generate the comprehensive constant catalog following `framework/artifacts/constant-catalog.template.md`. Populate from the B.5 Constant Inventory data.

Quality checks:
- Every constant from B.5 appears in the catalog (Section 1 Constant-to-Module Index)
- Category is assigned to every constant (fallback: `OTHER`)
- Section 3 (Shared Constants) lists all constants with 2+ consumer modules
- Section 4 (Magic Value Report) lists files exceeding 5% inline-literal threshold
- Section 5 (Constant Naming Summary) statistics are accurate
- Section 1 and Section 2 are internally consistent
- All consumer file paths are valid (exist in `docs/module-map.md` cross-reference)

#### E.5 Terminology Glossary (`docs/terminology-glossary.md`)

Generate the auto-derived terminology glossary following `framework/artifacts/terminology-glossary.template.md`. Populate from the B.6 Terminology Extraction data.

Quality checks:
- Every term from B.6 appears in the glossary (minimum 10 terms for non-empty projects, no cap for large)
- Every term has at least a fallback definition ("Domain entity: {term}")
- Section 2 (Term-to-Module) cross-references are complete
- Section 3 (Domain Cluster Map) has 1+ clusters for projects with >=3 modules
- Section 4 (Terms with No Definition) explicitly lists underdefined terms
- Related method and constant references in Section 2 are valid (exist in public-method-catalog.md and constant-catalog.md)
- All module IDs in "Related Modules" are valid (exist in `docs/module-map.md`)

#### E.6 Architecture Document Contributions
The scanner provides data for these sections of `docs/architecture.md`:
- **§2 Module Architecture** — full module inventory with types, keywords, and functional capabilities
- **§3 Module Dependency Graph** — ASCII diagram + dependency matrix (enhanced with functional dependency annotations)
- **§4 Data Architecture** — detected model/entity files and their relationships
- **§5 API/Interface Architecture** — public API surface grouped by module, annotated with capability labels

---

### Phase F: Generate Project-Level Coding Standard

This phase analyzes actual code to detect the project's existing conventions, then generates the **authoritative project-level coding standard**. This document (`docs/coding-standards.md` §0) is the primary reference for all downstream agents — when it conflicts with the generic language standard, the project-level convention **always** takes precedence.

The project-level standard covers three domains: public methods, constant definitions, and code organization conventions.

#### F.1 Public Method / Shared Utility Naming Detection

Analyze all exported functions in `utility` and `service` classified files:

| Detection Target | Method | Example Output |
|-----------------|--------|----------------|
| **Naming pattern** | Sample exported function names, detect dominant casing | `camelCase` (87%), `snake_case` (0%) |
| **Verb prefix** | Categorize first word of function names | `get*` (23), `create*` (12), `fetch*` (8), `validate*` (5) |
| **Return type pattern** | Check return type annotations for consistency | `Promise<T>` for async (74%), direct `T` for sync (26%) |
| **Parameter pattern** | Analyze first parameter naming | `id: string` (15), `input: CreateXInput` (8), `options: XOptions` (6) |

**Detection algorithm:**
1. Extract all exported function signatures from the module public API (Phase C)
2. For each function, record: name, parameter names/types, return type, async/sync
3. Aggregate statistics across the entire codebase
4. Determine dominant conventions (threshold: >60% usage)

**Output — Public Method Convention Report:**
```markdown
## Detected Public Method Conventions
| Convention | Dominant Pattern | Coverage | Examples |
|-----------|-----------------|----------|----------|
| Naming | camelCase, verb-prefixed | 100% | `getUserById`, `createOrder`, `validateEmail` |
| Async | async/await with Promise<T> return | 74% | `async function getUserById(id: string): Promise<User>` |
| Error handling | throw custom AppError subclasses | 82% | `throw new NotFoundError('User', id)` |
| Parameter naming | Single object param for >3 args | 68% | `createUser({ name, email, role }: CreateUserInput)` |
```

#### F.1.5 Constant Definition Detection

Detect how the project defines and organizes constants, enums, and magic values:

| Detection Target | Method | Example Output |
|-----------------|--------|----------------|
| **Constant naming** | Check `const`/`readonly` declarations at module/file scope | `UPPER_SNAKE_CASE` (92%), `camelCase` (8%) |
| **Enum naming** | Check enum member naming patterns | `PascalCase` members (100%) |
| **Constant location** | Detect where constants are defined | `src/shared/constants/` (78%), co-located with usage (22%) |
| **Grouping pattern** | How constants are grouped | Single `constants.ts` per domain (65%), `enum` objects (35%) |
| **Magic value tolerance** | Count inline literals in business logic vs extracted constants | Extracted (82%), inline magic numbers (18%) |
| **Type of constant** | `const` vs `enum` vs `as const` objects | `enum` for related sets (55%), `as const` (28%), bare `const` (17%) |
| **Export pattern** | Named export vs default, barrel re-export | Named export (96%), via barrel `index.ts` (74%) |
| **String constant pattern** | How string constants are defined (API paths, error codes, config keys) | `const API_*` prefix pattern (88%), nested objects (12%) |

**Detection algorithm:**
1. Scan all files for top-level `const`, `enum`, `readonly` declarations
2. Classify each by: naming case, location (shared vs co-located), export pattern
3. Count inline literals (magic numbers/strings) in business logic files
4. Flag files where >5% of statements are inline literals → magic value cleanup candidates
5. Determine dominant patterns (threshold: >70% coverage)

**Output — Constant Convention Report:**
```markdown
## Detected Constant Conventions
| Convention | Dominant Pattern | Coverage | Deviations |
|-----------|-----------------|----------|------------|
| Constant naming | `UPPER_SNAKE_CASE` for module-level `const` | 92% | `src/features/auth/config.ts:12` uses camelCase |
| Enum members | `PascalCase` | 100% | — |
| Constant location | `src/shared/constants/{domain}.ts` | 78% | 6 constants co-located in feature files |
| Grouping | One `constants.ts` per domain | 65% | `src/shared/constants/misc.ts` — mixed domains |
| Magic values | Extracted to named constants (82%) | 82% | `src/utils/validation.ts:45` inline `1000` |
| Type preference | `enum` for related sets, `as const` for unions | 55/28 split | — |
| Export pattern | Named export via barrel `index.ts` | 74% | 3 constant files exported directly (no barrel) |
| String pattern | `UPPER_SNAKE` with `API_`/`ERR_`/`CFG_` prefixes | 88% | — |
```

**Rules derived for project-level standard:**
- New constants MUST use `UPPER_SNAKE_CASE` at module scope
- New enum-like constants MUST use `enum` (preferred) or `as const`
- Constants shared across 2+ modules MUST live in `src/shared/constants/`
- String constants MUST follow detected prefix convention (`API_*`, `ERR_*`, `CFG_*`)
- Magic values in business logic → flagged for extraction

#### F.2 File Organization Pattern Detection

Analyze the directory structure and file placement conventions:

| Detection Target | Method | Example |
|-----------------|--------|---------|
| **Directory naming** | Detect case convention for directories | `kebab-case` (feature-based), `PascalCase` (component dirs), `camelCase` |
| **File naming** | Detect case convention per file role | `.tsx` = PascalCase, `.ts` = kebab-case, `.test.ts` = `*.spec.ts` |
| **Barrel file usage** | Check which directories have `index.ts` with re-exports | 89% of module directories have barrel exports |
| **Co-location pattern** | Check if tests, styles, types are co-located or separated | Tests: co-located `*.spec.ts` (100%) |
| **Component file structure** | For component dirs, detect file count/type per component | 1 file (42%), 3-file pattern[index+styles+test] (38%) |
| **Feature module structure** | For feature dirs, detect internal sub-structure | `components/` + `services/` + `types/` + `index.ts` (100%) |

**Detection algorithm:**
1. Walk directory tree, record directory names and contained file patterns
2. For each file type, detect the dominant naming convention by counting case patterns
3. For each module, categorize its internal directory structure
4. Flag inconsistencies where a pattern is used by <60% of instances

**Output — File Organization Report:**
```markdown
## Detected File Organization Conventions
| Convention | Dominant Pattern | Coverage | Inconsistencies |
|-----------|-----------------|----------|-----------------|
| Directory naming | kebab-case | 95% | `src/features/UserProfile/` uses PascalCase |
| File naming (components) | PascalCase `.tsx` | 100% | — |
| File naming (utilities) | kebab-case `.ts` | 100% | — |
| Test file naming | `*.spec.ts` co-located | 100% | — |
| Barrel exports | `index.ts` at module root | 89% | `src/features/reports/` missing barrel |
| Feature structure | `{components, services, hooks, types, index.ts}` | 100% | — |
```

#### F.3 Coding Style Convention Detection

Analyze actual code to detect:

| Detection Target | Method | Checks |
|-----------------|--------|--------|
| **Indentation** | Count leading whitespace in first 100 non-blank lines | Spaces (2/4) vs tabs |
| **Quote style** | Count string literal delimiters | Single quotes (91%) vs double (9%) |
| **Semicolons** | Check statement terminators | Semicolons present (100%) vs absent (0%) |
| **Trailing commas** | Check last element in multi-line arrays/objects | With trailing comma (78%) vs without (22%) |
| **Arrow vs function** | Count `const X = () =>` vs `function X()` | Arrow functions (64%), `function` keyword (36%) |
| **Type annotations** | Check if return types are explicit | Explicit return types (88%) vs inferred (12%) |
| **Import style** | Named vs default imports, import ordering | Named imports (97%), sorted by source (82%) |
| **Comment style** | `//` vs `/* */`, JSDoc usage | `//` inline (95%), JSDoc on public API (72%) |
| **Null handling** | `null` vs `undefined` usage in returns | `null` for "not found" (89%), `undefined` rarely (11%) |
| **Error pattern** | try/catch style, custom error classes | `try/catch` with typed catch (64%), custom `AppError` (78%) |
| **Max line length** | Count lines per file, detect outliers | Mean 94 chars, max 186, 3% of lines >120 chars |

**Detection algorithm:**
1. Sample up to 20 files per module (or all if fewer) for detailed analysis
2. For each convention, use regex/heuristic counting to determine the dominant pattern
3. Apply threshold: if a pattern is used in >70% of cases, it is the "dominant convention"
4. If no pattern exceeds 70%, mark as "mixed — no dominant convention"
5. Flag files that deviate from dominant patterns

**Output — Coding Style Report:**
```markdown
## Detected Coding Style Conventions
| Category | Dominant Pattern | Coverage | Deviations |
|----------|-----------------|----------|------------|
| Indentation | 2 spaces | 97% | `src/legacy/auth.ts` uses 4 spaces |
| Quotes | Single quotes `'` | 91% | `src/config/constants.ts` uses double quotes |
| Semicolons | Present `;` | 100% | — |
| Trailing commas | Yes, in multi-line | 78% | — |
| Function style | Arrow functions for expressions, `function` for declarations | 64/36 split | — |
| Return types | Explicit for public API, inferred for internals | 88/12 split | — |
| Import ordering | External → internal → relative, alphabetized | 82% | 13 files unsorted |
| Null vs undefined | `null` for absent values, avoid `undefined` | 89% | — |
| Error pattern | `throw new AppError(code, message)` via custom classes | 78% | 6 files use raw `Error` |
| Line width | ≤120 chars preferred | 97% within | `src/shared/utils/validation.ts:234` (186 chars) |
```

#### F.4 Generate Project-Level Coding Standard

The orchestrator uses the scanner's convention reports to produce the **authoritative project-level coding standard** at `docs/coding-standards.md`. This is NOT a merge of equals — the project-level standard (§0) is the **primary reference** for all downstream agents.

**Generation algorithm:**
1. Copy the matched language standard from `framework/standards/coding-standards.<lang>.md`
2. Prepend **§0: Project-Level Coding Standard** containing all three detection reports (F.1 public methods, F.1.5 constants, F.2 file organization, F.3 coding style)
3. For each generic rule, apply merge action:
   - **confirm:** Rule matches project → keep, annotate "✓ confirmed"
   - **override:** Project differs → **REPLACE** with project convention, annotate "⚠ project standard overrides"
   - **add:** Project has a convention not in generic → add as **new mandatory rule**, annotate "⬢ project-specific rule"
4. Flag inconsistencies (<70% coverage) as remediation items with specific file paths
5. Append a mandatory footer: **"本项目级编码规范由 Scanner Agent 自动检测生成。所有智能体编码时必须优先遵守 §0 项目级约定，其次遵循后续通用语言规范。当两者冲突时，以 §0 为准。"**

**Output — `.scanner-report.json` additions:**
```json
{
  "detected_conventions": {
    "public_methods": {
      "naming": { "pattern": "camelCase", "coverage": 1.0 },
      "verb_prefixes": ["get", "create", "update", "delete", "fetch", "validate"],
      "async_pattern": { "dominant": "async/await", "coverage": 0.74 },
      "error_pattern": { "dominant": "custom_AppError", "coverage": 0.82 }
    },
    "constants": {
      "naming": { "pattern": "UPPER_SNAKE_CASE", "coverage": 0.92 },
      "type_preference": { "enum": 0.55, "as_const": 0.28, "bare_const": 0.17 },
      "location": { "pattern": "src/shared/constants/{domain}.ts", "coverage": 0.78 },
      "grouping": { "pattern": "one file per domain", "coverage": 0.65 },
      "export_pattern": { "pattern": "named via barrel index.ts", "coverage": 0.74 },
      "magic_value_rate": { "extracted": 0.82, "inline": 0.18 },
      "string_prefixes": ["API_", "ERR_", "CFG_", "MSG_"]
    },
    "file_organization": {
      "directory_naming": { "pattern": "kebab-case", "coverage": 0.95 },
      "file_naming_components": { "pattern": "PascalCase.tsx", "coverage": 1.0 },
      "file_naming_utils": { "pattern": "kebab-case.ts", "coverage": 1.0 },
      "test_naming": { "pattern": "*.spec.ts co-located", "coverage": 1.0 },
      "barrel_exports": { "coverage": 0.89, "missing": ["src/features/reports/"] },
      "feature_structure": { "pattern": "{components,services,hooks,types,index.ts}", "coverage": 1.0 }
    },
    "coding_style": {
      "indentation": { "style": "2_spaces", "coverage": 0.97, "deviations": ["src/legacy/auth.ts"] },
      "quotes": { "style": "single", "coverage": 0.91 },
      "semicolons": { "style": "present", "coverage": 1.0 },
      "trailing_commas": { "style": "present_multiline", "coverage": 0.78 },
      "return_types": { "style": "explicit_public_api", "coverage": 0.88 },
      "import_order": { "style": "external_internal_relative_alphabetized", "coverage": 0.82 },
      "null_handling": { "style": "null_for_absent", "coverage": 0.89 },
      "line_width": { "limit": 120, "compliance": 0.97 }
    }
  },
  "merge_actions": [
    { "standard_section": "1.1 文件命名", "action": "confirm", "detail": "kebab-case 命名在 100% 文件中使用" },
    { "standard_section": "1.2 变量命名", "action": "confirm", "detail": "camelCase 在 100% 函数中使用" },
    { "standard_section": "4. 格式化", "action": "override", "detail": "项目使用单引号 (91%)，通用标准推荐双引号 → 覆盖为单引号" },
    { "standard_section": "7.3 路径别名", "action": "confirm", "detail": "@/*, @shared/*, @features/* 别名已配置并使用" },
    { "standard_section": "—", "action": "add", "detail": "新增 §0.4: 项目检测到自定义 AppError 错误模式 (82% 覆盖率)" }
  ]
}
```

#### F.5 Convention Quality Gates
- All detected public methods have their naming pattern documented
- File organization pattern coverage ≥80% for dominant conventions
- Coding style coverage statistics are based on ≥20 sampled files
- Inconsistencies (<70% coverage) are explicitly flagged with file paths
- Merge actions list is complete (every generic standard section evaluated)
- Detected conventions are written to both `docs/coding-standards.md` §0 and `.scanner-report.json`

---

## Proactive Challenge Checks

The Scanner Agent operates primarily during Phase 0, but its output (`docs/coding-standards.md` §0, `.scanner-report.json`) serves as the baseline for ongoing standards enforcement.

### CAT-1: Requirement Gaps (applies when re-scanning)
- [ ] Scanner-detected module structure matches what architecture docs claim
- [ ] All detected modules have corresponding entries in `docs/architecture.md` §2
- [ ] Shared resources catalog is complete — no shared component/method used by 2+ modules that was missed
- **If gap found:** Raise CAT-1 challenge against the agent whose output is inconsistent with scan results

### CAT-2: Standards Violations (primary scanner role)
- [ ] Files with naming inconsistent with detected conventions (§F.2) → raise CAT-2 challenge with file paths
- [ ] Files placed in wrong directory per detected organizational pattern → raise CAT-2 challenge
- [ ] Code using prohibited patterns from standards → raise CAT-2 challenge
- [ ] New circular dependencies or architecture rule violations → raise CAT-2 challenge
- [ ] Shared components importing from feature modules (architecture rule violation) → raise CAT-2 challenge
- **If violation found:** Raise CAT-2 challenge immediately with `docs/coding-standards.md` §0 citation and file:line evidence

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
