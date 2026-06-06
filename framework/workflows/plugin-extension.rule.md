# Plugin Extension Mechanism (插件附加流程机制)

## Overview
The plugin extension mechanism allows projects to customize and extend the standard development lifecycle through hook-based plugins. Each plugin can modify or augment the behavior at specific hook points in the workflow.

## Hook Points

Hook points are located at every phase transition in the standard lifecycle:

```
[PRD] ──prd-post──▶ [SE] ──se-post──▶ [Story] ──story-post──▶ [Test] ──test-post──▶ [Dev] ──dev-post──▶ [Review] ──review-post──▶ [Validation]
                       │                  │                      │                    │                    │
                     se-pre           story-pre             test-pre             dev-pre            review-pre
                                                                                                   │
                                                                                          validation-post
```

### Available Hooks

| Hook ID | Trigger Point | When It Runs | Context Available |
|---------|--------------|--------------|-------------------|
| `prd-post` | After PRD is provided | Before SE Design begins | PRD document |
| `se-pre` | Before SE Design begins | After PRD post-hooks | PRD, architecture |
| `se-post` | After SE Design is accepted | Before Story Design | PRD, SE Design |
| `story-pre` | Before Story Design begins | After SE post-hooks | SE Design |
| `story-post` | After Story is accepted | Before Test Plan | SE Design, Story |
| `test-pre` | Before Test Plan begins | After Story post-hooks | PRD, Story |
| `test-post` | After Test Plan is accepted | Before Development | Test Plan, Story |
| `dev-pre` | Before Development begins | After Test post-hooks | Story, Test Plan |
| `dev-post` | After code is written | Before Code Review | Source code, Story |
| `review-pre` | Before Code Review begins | After Dev post-hooks | Source code, Story, Test Plan |
| `review-post` | After Code Review is APPROVED | Before Validation | Code, Review Report, Story |
| `validation-post` | After Validation passes | End of lifecycle | All artifacts |

## Plugin Directory Structure

```
plugins/
├── README.md                    # Plugin development guide
└── <plugin-name>/
    ├── plugin.json              # Plugin manifest (required)
    ├── skill.md                 # Skill definition (optional)
    ├── hooks/                   # Hook implementations
    │   ├── se-pre.md
    │   ├── dev-post.md
    │   └── ...
    └── scripts/                 # Executable scripts (optional)
        └── validate.sh
```

## Plugin Manifest Format

`plugins/<plugin-name>/plugin.json`:
```json
{
  "name": "<plugin-name>",
  "version": "1.0.0",
  "description": "Brief description of what this plugin does",
  "hooks": {
    "se-pre": "hooks/se-pre.md",
    "dev-post": "hooks/dev-post.md"
  },
  "scripts": {
    "validate": "scripts/validate.sh"
  }
}
```

## Hook Execution

### Pre-hooks (`*-pre`)
- Execute BEFORE the agent begins its phase
- Can: modify inputs, add constraints, inject additional context
- Cannot: skip the phase entirely (must return control to the agent)

### Post-hooks (`*-post`)
- Execute AFTER the agent completes its phase, before handoff
- Can: add validation checks, transform outputs, append documentation
- Cannot: reject the output unconditionally (use challenge mechanism instead)

## Plugin Discovery

The orchestrator discovers plugins at startup:
1. Scan `plugins/` directory for subdirectories containing `plugin.json`
2. Load each manifest
3. Build hook registry mapping hook IDs to plugin implementations
4. Before each lifecycle phase, consult the registry for relevant hooks

Example hook execution order for SE Design phase:
1. `plugin-a:se-pre` → injects custom constraints
2. `plugin-b:se-pre` → appends additional context
3. SE Agent executes standard SE Design phase
4. `plugin-a:se-post` → runs custom validation
5. `plugin-b:se-post` → generates supplementary docs

## Plugin vs Challenge Mechanism

If a plugin detects an issue:
- **Minor issues:** Plugin should report them in its output, allowing the next agent to decide
- **Standards violations:** Plugin should recommend the next agent raise a formal challenge
- **Blocking issues:** Plugin should halt and notify the orchestrator, which may trigger a challenge

Plugins themselves can be challenged by agents:
- If an agent believes a plugin's hook is producing incorrect or harmful modifications
- The agent must cite the specific plugin behavior and why it violates standards
- The challenge is directed to the user (plugin owner), not another agent

## Built-in Plugins

The framework ships with these reference plugins:

### Example Plugin (`plugins/example-plugin/`)
Demonstrates hook structure. Adds a pre-development checklist and post-development log generation.

### Validation Plugin (language-specific, generated at init)
Located in `plugins/validation-<lang>/`. Provides standardized input/output validation per `framework/standards/validation-standards.template.md`.
