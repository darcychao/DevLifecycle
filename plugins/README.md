# Plugin Development Guide

Plugins extend the H5 Lifecycle Framework by injecting custom behavior at hook points in the standard development lifecycle.

## Quick Start

### 1. Create Plugin Directory
```
plugins/my-plugin/
├── plugin.json          # Required: plugin manifest
├── skill.md             # Optional: skill definition for agents
├── hooks/               # Hook implementations
│   └── se-pre.md        # Example: runs before SE Design
└── scripts/             # Optional: executable scripts
    └── custom-check.sh
```

### 2. Define Plugin Manifest
Create `plugin.json`:
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "hooks": {
    "se-pre": "hooks/se-pre.md",
    "dev-post": "hooks/dev-post.md"
  },
  "scripts": {
    "check": "scripts/custom-check.sh"
  }
}
```

### 3. Implement Hooks
Each hook file is a markdown instruction that the orchestrator injects into the agent's context at the specified hook point. See below for hook format.

## Available Hooks

| Hook | When | Use Cases |
|------|------|-----------|
| `prd-post` | After PRD provided | Validate PRD completeness, add templates |
| `se-pre` | Before SE Design | Inject architecture constraints, check existing modules |
| `se-post` | After SE Design accepted | Run design validation, generate supplementary docs |
| `story-pre` | Before Story Design | Add implementation guidelines |
| `story-post` | After Story accepted | Validate story completeness |
| `test-pre` | Before Test Plan | Inject test requirements, security test cases |
| `test-post` | After Test Plan accepted | Add regression tests |
| `dev-pre` | Before Development | Pre-dev checklist, environment setup |
| `dev-post` | After Development | Run linters, formatters, type checkers |
| `validation-post` | After Validation | Generate reports, update changelogs |

## Hook File Format

A hook file is a markdown instruction that follows this structure:

```markdown
# Hook: [Hook ID] — [Plugin Name]

## Context
[What this hook does and why]

## Instructions
[Specific instructions for the orchestrator/agent at this point]

## Output
[What this hook produces or modifies]
```

For scripts referenced in hooks, the orchestrator executes them and captures output.

## Discovery

The orchestrator discovers plugins at startup by scanning `plugins/` for subdirectories containing `plugin.json`. No registration step needed — just create the directory and manifest.

## Best Practices

1. **One concern per plugin** — don't bundle unrelated hooks
2. **Pre-hooks inform, post-hooks validate** — pre-hooks add context, post-hooks check results
3. **Don't block without cause** — hooks should not reject work unless it violates standards
4. **Use challenge mechanism for violations** — don't silently reject; raise a formal challenge
5. **Document your hooks** — agents need clear instructions to apply your plugin correctly
