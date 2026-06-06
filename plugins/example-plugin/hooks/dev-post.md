# Hook: dev-post — Example Plugin

## Context
This post-development hook runs after code is committed but before validation. It performs automated quality checks.

## Instructions
After Dev Agent completes Phase 5 (Development Coding), the orchestrator should:

1. Run the project linter on changed files
2. Run the project formatter check
3. Run the type checker
4. Run all existing tests to check for regressions
5. Run new tests written for this story
6. Generate a summary:
   - Files changed: [count]
   - Lines added/removed: [+N/-N]
   - Tests run: [passed/total]
   - Lint issues: [count]
   - Type errors: [count]

If any check fails, report failures to Dev Agent. Dev Agent must fix before Test Agent begins validation.
