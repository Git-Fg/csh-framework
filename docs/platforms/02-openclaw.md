# Platform Guide: OpenClaw

This document covers the integration patterns, skill system, hooks, and session lifecycle when using the <framework> framework with OpenClaw.

---

## How Skills Are Discovered and Loaded

### Discovery Mechanism

OpenClaw uses a dynamic skill discovery system with feature gating:

1. **Scanned Locations** (in precedence order):
   - Workspace skills: `~/.csh/workspace/*/SKILL.md`
   - Managed skills: `~/.csh/skills/`
   - Bundled skills: Distributed with the gateway

2. **Feature Gating**: Skills can declare runtime requirements:
   ```yaml
   requires:
     bins: ["curl", "jq"]
     env: ["API_KEY"]
   ```

3. **Metadata-First Loading**: OpenClaw loads YAML frontmatter first to optimize prompt density before loading full skill instructions.

### Loading Behavior

Skills are discovered via CLI:
```bash
<framework> skills list
<framework> memory list --types
```

### Permission Control

Skills can be explicitly enabled or disabled:
```bash
<framework> skills enable <skill-name>
<framework> skills disable <skill-name>
```

---

## Hook System

### Hook Configuration

OpenClaw hooks are configured via two components:
- **Instructions**: `HOOK.md` file containing guidance
- **Logic**: `handler.ts` (or shell script) containing executable code

### Hook Discovery Precedence

Hooks are scanned with this priority:
1. Workspace hooks
2. Managed hooks
3. Bundled hooks

### Automation Triggers

Hooks can be triggered by:
- **Messages**: Incoming commands from users
- **Heartbeats**: Periodic batched checks (~30 minute intervals)
- **Webhooks**: External HTTP callbacks
- **Cron**: Exact schedule timing

### Supported Hook Types

| Category | Events |
|----------|--------|
| **Lifecycle** | `session_start`, `session_end` |
| **Command** | `before_command`, `after_command` |
| **System** | `webhook`, `cron`, `heartbeat` |

### Hook Handler Pattern

```typescript
// hooks/my-hook/handler.ts
function handler(event: HookEvent) {
  const { type, payload } = event;

  switch (type) {
    case 'before_command':
      // Pre-execution logic
      return { continue: true };
    case 'after_command':
      // Post-execution logic
      return { success: true };
    default:
      return { continue: true };
  }
}

module.exports = { handler };
```

### Bundled Hooks

Hooks can be bundled within plugins to provide out-of-the-box infrastructure automation. Configure in manifest:

```json
{
  "hooks": {
    "before_command": "./hooks/pre-command"
  }
}
```

---

## Manifest Format

### Plugin Manifest

OpenClaw plugins use `openclaw.plugin.json`:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Example plugin for OpenClaw",
  "skills": ["./skills"],
  "hooks": {
    "before_command": "./hooks/pre-command"
  },
  "settings": {
    "allowBundled": ["skill-a", "skill-b"]
  }
}
```

### Allowlist Configuration

**CRITICAL**: When `skills.allowBundled` is set to an empty array `[]`, it becomes RESTRICTIVE - no bundled skills will load.

Options:
1. **Omit entirely** - All bundled skills allowed (recommended)
2. **List explicitly** - Only listed skills allowed: `["skill-a", "skill-b"]`
3. **Avoid empty array** - `[]` blocks all bundled skills

### Skill Metadata (SKILL.md Frontmatter)

```yaml
---
name: example-skill
description: Example skill demonstrating OpenClaw format
trigger: /example
platform: openclaw
requires:
  bins: ["curl"]
  env: ["API_KEY"]
versioned: true
---
```

---

## Session Lifecycle

### Session Management

OpenClaw manages sessions centrally through the gateway:

1. **Queue Modes**:
   - `collect`: Coalesce multiple messages before processing
   - `followup`: Sequential processing
   - `steer`: Injection at tool boundaries

2. **Isolation**:
   - `secure DM mode`: Isolates context per sender/channel
   - Prevents cross-talk in multi-user environments

3. **Persistence**:
   - Sessions survive restarts via JSONL transcript storage
   - State stored in `~/.csh/state/session.json`

### State File

OpenClaw provides a visible, editable state file:
- Location: `~/.csh/state/session.json`
- Accessible via: `<framework> memory get`
- Developer-friendly: Human-readable JSON format

### Lifecycle Events

1. **session_start** - Gateway session initialized
2. **session_end** - Gateway session terminates
3. **before_command** - Before any command executes
4. **after_command** - After command completes

### Session Optimization

The gateway optimizes the system prompt by injecting only the most relevant skill frontmatter, avoiding context bloat.

---

## Integration Patterns

### Hybrid Setup

When using OpenClaw alongside other platforms:

1. **Naming Convention**: Use unique prefixes for skills
   - OpenClaw skills: `<framework>-<skill-name>`
   - Other platform skills: `<platform>-<skill-name>`

2. **Hook Order**: OpenClaw hooks typically run before other platform hooks to ensure global context is loaded first

3. **Memory Coordination**: Use OpenClaw for persistent "Vault" memory; use native platform features for ephemeral session notes

### Multi-Channel Access

OpenClaw supports:
- Desktop CLI access
- Mobile/Telegram integration
- Webhook triggers

### Best Practices

- Leverage feature gating for OS/environment-specific skills
- Use workspace hooks for project-specific automation
- Keep state files in `~/.csh/` for persistence
- Version skills using git for distribution

---

## Summary

| Aspect | OpenClaw Behavior |
|--------|-------------------|
| **Skill Discovery** | Dynamic scan with feature gating |
| **Skill Loading** | Metadata-first, gated by `requires` |
| **Hook Events** | session_start/end, before/after_command, webhook, cron, heartbeat |
| **Hook Format** | `HOOK.md` + `handler.ts` |
| **Manifest** | `openclaw.plugin.json` |
| **Session** | Gateway-managed, persistent |
| **Persistence** | JSONL transcripts + state file |
| **Multi-channel** | Desktop, Mobile, Webhook |

