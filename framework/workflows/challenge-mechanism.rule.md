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
| PRD → SE Design | SE Agent | User | PRD ambiguity, missing requirements |
| SE Design → Story | Dev Agent | SE Agent | Unimplementable design, missing interfaces |
| Story → Test Plan | Test Agent | Dev Agent | Untestable story, missing edge cases |
| SE Design ← Story | SE Agent | Dev Agent | Story deviates from design |
| Test Plan → Dev | Dev Agent | Test Agent | Invalid test criteria, impossible test cases |
| Dev → Code Review | SE Agent (system-generated) | Dev Agent | **8-item checklist failures:** missing requirements, dev story misalignment, standards violations, architectural issues, correctness bugs, missing tests, security issues, code omissions |
| Code Review → Validation | Test Agent | Dev Agent | Post-review issues found during validation |
| Code Review → Validation | Test Agent | SE Agent | Code review missed issues now found in validation |
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

## Challenge Prevention

Agents should minimize challenges by:
- Following document templates exactly
- Self-checking output against all relevant standards before submission
- Including explicit rationale for design decisions
- Flagging assumptions and open questions in the document itself
- **Dev Agent:** Self-validating against the 8-item Code Review Checklist before submitting code for review
