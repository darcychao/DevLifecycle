# Challenge Mechanism (挑战机制)

## Overview
The challenge mechanism ensures quality and compliance by allowing any agent to formally contest another agent's output when it violates documented standards, specifications, or requirements. Challenges have the highest priority in the framework — all agents must resolve challenges before proceeding.

The Code Review phase (Phase 6) extends the challenge mechanism with **system-generated challenges**: the SE Agent, acting as code reviewer, automatically files formal challenges against the Dev Agent for each failed checklist item. These follow the same rules as agent-discretionary challenges.

## Priority
**Highest.** The challenge mechanism overrides all other workflow rules. When a challenge exists, the workflow pauses at the current phase until the challenge is resolved.

## Challenge Lifecycle

```
┌──────────┐     ┌───────────┐     ┌──────────┐     ┌───────────┐
│  Agent B │ ──▶ │ Challenge │ ──▶ │  Agent A │ ──▶ │ Resolution│
│ detects  │     │  Filed    │     │ responds │     │  (Accept  │
│  issue   │     │ (formal)  │     │          │     │  /Reject) │
└──────────┘     └───────────┘     └──────────┘     └───────────┘
                                                        │
                                          ┌──────────────┴──────────────┐
                                          ▼                             ▼
                                    Challenge                    Challenge
                                    Accepted                     Rejected
                                          │                             │
                                          ▼                             ▼
                                   Agent A revises              Agent B must
                                   and re-outputs               provide more
                                   document/code                evidence or
                                                                withdraw
                                          │                             │
                                          ▼                             ▼
                                   Agent B re-reviews           If persists →
                                   (Code Review:                escalate to
                                    re-execute failed           User for final
                                    checklist items)            decision
```

## Rules for Raising a Challenge

### Required Elements
A valid challenge MUST include:
1. **Target:** Which agent and which document/output is challenged
2. **Basis:** Specific standard/specification/requirement clause that is violated (quote it)
3. **Evidence:** How the output deviates from the cited basis (file:line for code)
4. **Impact:** Why this deviation matters (blocks implementation, causes inconsistency, runtime error, security risk, etc.)
5. **Suggested Fix:** What the corrected output should look like (if known)

### Valid Basis Sources (must cite one or more)
- `framework/standards/coding-standards.<lang>.md` — any specific chapter or clause
- `framework/standards/project-structure.template.md` — any specific rule
- `framework/standards/validation-standards.template.md` — any specific validation criteria
- `docs/prd.md` — any specific requirement (FR-XXX, US-XXX)
- `docs/se-design.md` — any specific design specification
- `docs/dev-story.md` — any specific task (TASK-XXX) or implementation detail
- `docs/architecture.md` — any architectural constraint or module dependency rule
- `docs/test-plan.md` — any specific test case or acceptance criterion
- `framework/artifacts/*.template.md` — missing required sections
- Code Review Checklist — any of the 8 items (CR-1 through CR-8)
- `docs/public-method-catalog.md` — Method-to-Module Index for proving method duplication or absence
- `docs/constant-catalog.md` — Constant-to-Module Index for proving constant non-registration or duplication
- `docs/terminology-glossary.md` — Term Index for proving terminology inconsistency

### Invalid Challenges
Challenges are REJECTED if:
- No specific standard/clause is cited ("I don't like this approach")
- The basis is personal preference, not a documented standard
- The cited standard does not apply to the challenged output
- The challenge is about a future phase's concern (not relevant yet)

## Rules for Handling a Challenge

### Agent Receiving a Challenge MUST:
1. **Stop current work immediately** — challenge has highest priority
2. **Read the challenge fully** — understand what is being contested
3. **Verify the basis** — check the cited standard/specification
4. **Respond with one of:**
   - **Accept** — Acknowledge the valid challenge, fix the issue, re-output the document/code
   - **Reject with evidence** — Explain why the challenge is invalid, citing counter-evidence from standards/specs
   - **Request clarification** — If the challenge basis is unclear, ask for more specific citation

### Resolution Criteria
- **Accepted:** Agent revises output, challenge is marked resolved, workflow resumes
- **Rejected:** If challenger accepts rejection, challenge closed. If challenger persists, escalate to requirements provider (user) for final decision.
- **Escalation:** User decides. User's decision is final and may update standards/specs.

## Challenge Positions in the Workflow

Challenges can occur at these handoff points:

| Phase Transition | Potential Challenger | Potential Target | Common Challenge Topics |
|-----------------|---------------------|-----------------|------------------------|
| PRD → SE Design | SE Agent | User | PRD ambiguity, missing requirements, missing UX constraints |
| SE Design → UX Spec | UX Agent | SE Agent | UI architecture incomplete, component tree missing |
| UX Spec → Story Design | Dev Agent | UX Agent | UX Spec contradicts framework capabilities or is too vague |
| SE Design → Story | Dev Agent | SE Agent | Unimplementable design, missing interfaces |
| Story → Test Plan | Test Agent | Dev Agent | Untestable story, missing edge cases |
| SE Design ← Story | SE Agent | Dev Agent | Story deviates from design |
| UX Spec ← Story | UX Agent | Dev Agent | UI tasks lack UX specification coverage |
| Test Plan → Dev | Dev Agent | Test Agent | Invalid test criteria, impossible test cases |
| Dev → Code Review | SE Agent (system-generated) | Dev Agent | **8-item checklist failures (CR-1~CR-8)** |
| Dev → UX Review | UX Agent (system-generated) | Dev Agent | **CAT-3: CR-9 UX Spec compliance failures** |
| Code Review → Validation | Test Agent | Dev Agent | Post-review issues found during validation |
| Code Review → Validation | Test Agent | SE Agent | Code review missed issues now found in validation |
| Code Review → Validation | UX Agent | Dev Agent | UX issues found during validation |
| Any → Any | Any Agent | User (plugin owner) | Plugin hook produces incorrect/harmful modifications |

## Code Review Challenges (special case)

Code Review challenges differ from standard challenges in these ways:

| Aspect | Standard Challenge | Code Review Challenge |
|--------|-------------------|----------------------|
| Trigger | Agent discretion | System-generated (checklist failure) |
| Basis | Agent must find and cite | Checklist item ID (CR-1..CR-8) + specific evidence |
| Scope | Single issue | Can be multiple items in one review report |
| Resolution | Agent accepts/rejects each individually | Dev Agent must fix ALL before re-review |
| Re-review | Manual by challenging agent | SE Agent re-executes only failed checklist items |
| Escalation | After first rejection deadlock | After same item fails twice |

## Multiple Simultaneous Challenges
- If Agent A receives challenges from both Agent B and Agent C, address in order of receipt
- If challenges are related, resolve together
- If challenges conflict (one says X is wrong, other says X is right), escalate to user
- Code review challenges are bundled in one review report; Dev Agent addresses all together

## Catalog Compliance Challenge Sub-Categories (CAT-2 Extension)

The following sub-categories extend CAT-2 (Standards Violation Challenge) for catalog-driven compliance enforcement. These challenges are triggered when new code introduces symbols that conflict with cataloged project assets.

### CAT-2a: Method Duplication (方法重复)
- **Trigger condition:** Newly created exported function has equivalent signature and behavior to an existing method in `docs/public-method-catalog.md`
- **Detection timing:** Phase 5 Self-Validation (Dev Agent self-check), Phase 6 Code Review (SE Agent CR-3), Phase 6.5 Catalog Consistency Verification
- **Basis:** `docs/coding-standards.md` §0.2.1 MUST-01
- **Challenged party:** Dev Agent
- **Resolution:** Delete the duplicate method and use the cataloged method instead; or prove functional non-equivalence with documented evidence
- **Challenge record must cite:** Method-to-Module Index row showing the existing equivalent method

### CAT-2b: Constant Non-Registration (常量未归档)
- **Trigger condition:** Newly created constant is referenced by 2+ modules but does not appear in `docs/constant-catalog.md` Shared Constants table
- **Detection timing:** Phase 5 Self-Validation, Phase 5.5 Catalog Compliance Self-Check, Phase 6 Code Review (SE Agent CR-3), Phase 6.5 Catalog Consistency Verification
- **Basis:** `docs/coding-standards.md` §0.3.1 MUST-05
- **Challenged party:** Dev Agent
- **Resolution:** Register the constant in constant-catalog.md Shared Constants table; or move the constant to the unified constants file
- **Challenge record must cite:** Shared Constants table showing the constant is absent

### CAT-2c: Terminology Inconsistency (术语不一致)
- **Trigger condition:** Newly created type/interface/function name conflicts with an existing term definition in `docs/terminology-glossary.md` (e.g., same concept using a different name, or same name used for a different concept)
- **Detection timing:** Phase 3 Story Design (Dev Agent), Phase 6 Code Review (SE Agent CR-3), Phase 6.5 Catalog Consistency Verification
- **Basis:** `docs/coding-standards.md` §0.7.1 MUST-09, MUST-10
- **Challenged party:** Dev Agent or SE Agent
- **Resolution:** Rename to unify with the glossary-defined term; or register the new term with a clear distinction from the existing term
- **Challenge record must cite:** Term Index row showing the conflicting existing term

### Catalog Challenge Resolution Flow

```
CAT-2a/2b/2c raised
       │
       ├── Agent accepts → Fix code to match catalog → Re-run self-check → Catalog verified → Phase proceeds
       │
       └── Agent rejects (claims non-equivalence) → Submit evidence to challenger → Challenger reviews
                    │
                    ├── Evidence accepted → Challenge CLOSED (REJECTED with documentation)
                    │
                    └── Evidence rejected → ESCALATED to User for final decision
```

**Escalation rule:** If the same catalog compliance issue is challenged twice without resolution → immediate escalation. Catalog integrity is critical for long-term project maintainability.

### CAT-3: UX Specification Violation Challenge (UX规范违反)

Challenge category for code deviations from the extracted UX Requirements Specification. The UX Agent does NOT design — it extracts requirements from existing design artifacts. CAT-3 challenges can ONLY cite violations of explicitly stated UX Specification items.

**Trigger conditions:**
- UI component visual states deviate from UX Spec §2 extracted requirements → UX Agent vs Dev Agent
- Interaction behavior doesn't match UX Spec §3 extracted requirements → UX Agent vs Dev Agent
- Responsive layout breaks at UX Spec §4 specified breakpoints → UX Agent vs Dev Agent
- Accessibility requirements extracted in UX Spec §5 not met → UX Agent vs Dev Agent
- Hardcoded styles instead of design tokens per UX Spec §6 mapping → UX Agent vs Dev Agent
- Missing UX states per UX Spec §2 state requirements → UX Agent vs Dev Agent
- Animation/transition doesn't match UX Spec extracted requirements → UX Agent vs Dev Agent
- Component uses wrong variant or size per UX Spec requirements → UX Agent vs Dev Agent
- Dev Story UI tasks lack UX Specification coverage → UX Agent vs Dev Agent (Phase 3)

**Invalid CAT-3 challenges (REJECTED):**
- Challenge based on personal aesthetic preference without UX Spec citation
- Challenge for a UI aspect not covered by the UX Specification ("not specified" = out of scope)
- Challenge that invents requirements the user never provided ("UX Agent thinks it should look different")

**Challenge format for CAT-3:**
```
Basis: "UX Spec §X.Y: {extracted requirement} — {deviation description}"
       OR
       "docs/coding-standards.md §0.6.Y: {UX convention} — {deviation description}"
Evidence: file:line with actual vs expected (include visual diff description when applicable)
Impact: {user experience degradation / accessibility barrier / design inconsistency}
Suggested Fix: "Change {X} to {Y} per UX Spec §Z"
```

**Detection timing:**
- Phase 2.6 UX Spec Extraction: UX Agent flags gaps in SE Design UI architecture against UX guidelines
- Phase 3 Story Design: UX Agent reviews Dev Story UI tasks for UX spec coverage
- Phase 6 Code Review: UX Agent executes CR-9 checklist (UX Specification Compliance)
- Phase 7 Validation: UX Agent verifies final UI output meets spec

**Challenged party:** Dev Agent (implementation), SE Agent (UI architecture gaps), or PRD author (missing UX constraints)

**Resolution:**
1. If CAT-3 raised against Dev Agent: Dev Agent fixes UI code to match UX Spec → UX Agent re-reviews → CR-9 re-executed → same item fails twice → escalate to user
2. If CAT-3 raised against SE Agent: SE Agent updates UI architecture → UX Agent re-reviews
3. If CAT-3 raised against UX Agent (spec error): UX Agent corrects spec extraction error → challenger re-reviews
4. If spec is correct but unimplementable: escalate to user for UX requirement adjustment

### CAT-3 Challenge Resolution Flow

```
CAT-3 raised (CR-9 failure — code deviates from UX Spec)
       │
       ├── Agent accepts → Fix UI code to match UX Spec → CR-9 re-executed → Phase proceeds
       │
       └── Agent rejects (claims UX Spec is unimplementable or conflicts with framework) 
                    │
                    ├── UX Agent & Dev Agent negotiate: can UX Spec be reasonably implemented?
                    │    ├── Yes: Dev Agent implements → UX Agent re-reviews
                    │    ├── Spec is too vague: UX Agent requests user clarification (does NOT invent)
                    │    └── Spec conflicts with framework: ESCALATED to User
                    │
                    └── Same CR-9 sub-item fails twice → ESCALATED to User
```

**Escalation rule:** UX disputes that can't be resolved between UX Agent and Dev Agent in one cycle → immediate user escalation. The user is the final arbiter — UX Agent only enforces requirements the user has provided.

## Challenge Prevention

Agents should minimize challenges by:
- Following document templates exactly
- Self-checking output against all relevant standards before submission
- Including explicit rationale for design decisions
- Flagging assumptions and open questions in the document itself
- **Dev Agent:** Self-validating against the 8-item Code Review Checklist before submitting code for review
