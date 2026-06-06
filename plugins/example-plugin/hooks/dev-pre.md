# Hook: dev-pre — Example Plugin

## Context
This pre-development hook ensures the Dev Agent has verified all prerequisites before writing code.

## Instructions
Before beginning Phase 5 (Development Coding), the Dev Agent MUST confirm:

1. SE Design is reviewed and accepted (no pending challenges)
2. Dev Story is reviewed and accepted by SE Agent
3. Test Plan exists and has been reviewed
4. All plugin hooks for prior phases have been executed
5. The development environment is ready (dependencies installed, database available)

If any prerequisite is missing, report it and do not proceed until resolved.
