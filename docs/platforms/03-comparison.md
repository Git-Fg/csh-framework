# Platform Comparison

Choosing between Claude Code and OpenClaw for your framework integration.

---

## Overview

| Aspect | Claude Code | OpenClaw |
|--------|-------------|-----------|
| **Type** | Local CLI agent | Gateway server + clients |
| **Sessions** | Ephemeral | Persistent |
| **Skills** | `~/.claude/skills/` | Plugin-based |
| **Hooks** | `hooks.json` | `openclaw.plugin.json` |
| **Extensibility** | Limited | Full plugin API |

---

## Skill System Comparison

### Claude Code

- **Location**: `~/.claude/skills/`
- **Format**: `SKILL.md` files
- **Discovery**: Auto-scan on startup
- **Invocation**: `/skill-name` or agent invokes

### OpenClaw

- **Location**: Plugin `skills/` directories
- **Format**: `SKILL.md` files
- **Discovery**: Via plugin manifest
- **Invocation**: `/skill read skill-name`

---

## Hook System Comparison

### Claude Code Hooks

```json
{
  "hooks": {
    "events": {
      "SessionStart": {
        "command": "./start.sh"
      },
      "PostToolUse": {
        "command": "./track.sh"
      }
    }
  }
}
```

**Available Events**: SessionStart, SessionEnd, Stop, TaskCompleted, PreToolUse, PostToolUse, Notification, UserPromptSubmit

### OpenClaw Hooks

```json
{
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/start.sh"
    }
  ]
}
```

**Available Events**: session_start, session_end, before_command, after_command

**Plus Automation**: webhook, cron, heartbeat triggers

---

## When to Choose Each

### Choose Claude Code When

- Running locally for personal use
- Need quick iteration
- Minimal infrastructure
- Single-user scenario

### Choose OpenClaw When

- Need persistent sessions
- Require gateway management
- Multi-user or team scenario
- Need automation triggers (cron, webhooks)
- Building production systems

---

## Cross-Platform Development

See [Cross-Platform Skills](../patterns/04-cross-platform.md) for techniques to support both platforms.

---

## Summary

| Requirement | Claude Code | OpenClaw |
|-------------|-------------|-----------|
| Personal automation | ✅ | ✅ |
| Team collaboration | ❌ | ✅ |
| Persistent sessions | ❌ | ✅ |
| Cron/webhook automation | ❌ | ✅ |
| Quick prototyping | ✅ | ✅ |
| Production deployment | ❌ | ✅ |

For framework development, OpenClaw offers more flexibility while Claude Code provides simpler local development.
