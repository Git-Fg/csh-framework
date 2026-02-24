# Hook Events Reference

Events relevant for building plugins with CLI tools.

---

## OpenClaw Events for Plugin Developers

### Command Events

| Event | Fires When | Use For |
|-------|------------|---------|
| `command:new` | `/new` command | Load context from previous sessions |
| `command:reset` | `/reset` command | Cleanup before reset |
| `command:stop` | `/stop` command | Graceful shutdown |

### Agent Events

| Event | Fires When | Use For |
|-------|------------|---------|
| `agent:bootstrap` | Before workspace bootstrap | Inject plugin context |

### Gateway Events

| Event | Fires When | Use For |
|-------|------------|---------|
| `gateway:startup` | Gateway starts | Plugin initialization |

### Plugin API

| Event | Fires When | Use For |
|-------|------------|---------|
| `tool_result_persist` | Before tool result saved | Transform CLI output |

---

## Claude Code Events

| Event | Fires When |
|-------|------------|
| `SessionStart` | Session begins |
| `SessionEnd` | Session ends |
| `PreToolUse` | Before tool invocation |
| `PostToolUse` | After tool completes |
| `TaskCompleted` | Task finishes |

---

## OpenClaw Hook Structure

### Directory

```
my-plugin/
├── hooks/
│   └── my-hook/
│       ├── HOOK.md
│       └── handler.ts
└── openclaw.plugin.json
```

### HOOK.md

```yaml
---
name: my-hook
description: "Does something useful"
metadata:
  openclaw:
    emoji: "🔧"
    events: ["command:new", "gateway:startup"]
    requires:
      bins: ["my-cli"]
---

# My Hook

This hook does something useful.
```

### Metadata Options

| Field | Description |
|-------|-------------|
| `events` | Array of events: `command:new`, `command:reset`, `agent:bootstrap`, `gateway:startup` |
| `requires.bins` | CLI tools that must exist |
| `requires.env` | Environment variables needed |
| `requires.config` | Config paths required |
| `always` | Load regardless of eligibility |

### Handler (handler.ts)

```typescript
const handler = async (event) => {
  // Gateway startup
  if (event.type === "gateway") {
    console.log("[my-hook] Gateway started");
    return;
  }

  // Command event
  if (event.type === "command" && event.action === "new") {
    // Run CLI to get context
    const { execFileSync } = require('child_process');
    const context = execFileSync('my-cli', ['context'], { encoding: 'utf8' });
    event.messages.push("Plugin context loaded");
  }
};

export default handler;
```

---

## Plugin Manifest (openclaw.plugin.json)

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What it does",
  "configSchema": {
    "type": "object",
    "properties": {}
  },
  "skills": ["./skills"],
  "hooks": [
    {
      "event": "gateway:startup",
      "handler": "./hooks/my-hook"
    }
  ]
}
```

### Required Fields

| Field | Description |
|-------|-------------|
| `id` | Unique plugin ID |
| `configSchema` | JSON Schema for config (can be empty `{}`) |

### Optional Fields

| Field | Description |
|-------|-------------|
| `version` | Plugin version |
| `name` | Display name |
| `description` | Short summary |
| `skills` | Skill directories to load |
| `hooks` | Hook handlers |

---

## Hook Discovery (for Plugins)

Hooks bundled in plugins are automatically loaded when plugin is installed.

```
Plugin directory:
├── openclaw.plugin.json
├── hooks/
│   └── my-hook/
│       ├── HOOK.md
│       └── handler.ts
└── skills/
    └── my-skill/
        └── SKILL.md
```

---

## CLI Commands for Testing

```bash
# Reload plugin
openclaw plugins install -l /path/to/plugin
openclaw gateway restart

# List hooks
openclaw hooks list

# Check eligibility
openclaw hooks check
```

---

## Related Docs

- [Plugin Development](../guides/02-plugin-development.md)
- [Skill Format](../reference/02-skill-format.md)
