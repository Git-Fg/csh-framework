# Hook Events Reference

Complete reference for available hook events across platforms.

---

## OpenClaw Hook Events

### Lifecycle Hooks

| Event | Fires When | Common Use |
|-------|-------------|------------|
| `session:start` | New session begins | Load context, set variables |
| `session:end` | Session completes | Save state, cleanup |
| `session:new` | New command in session | Capture context |

### Command Hooks

| Event | Fires When | Common Use |
|-------|-------------|------------|
| `before:command` | Before tool execution | Context injection |
| `after:command` | After tool execution | Output processing |

### Automation Triggers

| Trigger | Fires When | Common Use |
|---------|-------------|------------|
| `webhook` | HTTP request received | External integrations |
| `cron` | Scheduled time | Periodic tasks |
| `heartbeat` | Interval | Health checks |

---

## Claude Code Hook Events

### Session Events

| Event | Fires When |
|-------|------------|
| `SessionStart` | Session begins |
| `SessionEnd` | Session ends |
| `Stop` | Session stopped |

### Task Events

| Event | Fires When |
|-------|------------|
| `TaskCompleted` | Task finishes |
| `TaskFailed` | Task fails |

### Tool Events

| Event | Fires When |
|-------|------------|
| `PreToolUse` | Before tool invocation |
| `PostToolUse` | After tool completes |
| `PostToolUseFailure` | Tool fails |

### Other Events

| Event | Fires When |
|-------|------------|
| `Notification` | Notification received |
| `UserPromptSubmit` | User submits prompt |
| `SubagentStart` | Subagent starts |
| `SubagentStop` | Subagent stops |

---

## Hook Handler Patterns

### Shell Script Handler

```bash
#!/bin/bash
# Read event from stdin
EVENT=$(jq -r '.type' /dev/stdin)

case "$EVENT" in
  "session:start")
    echo "Session started"
    ;;
esac
```

### Prompt Injection Handler

```bash
#!/bin/bash
# Output to be injected into context
echo "Remember: When working with databases, use the db-query skill for safe queries."
```

---

## Best Practices

### What Hooks ARE For

- **Context reinforcement**: Remind agent to use relevant skills
- **Automation**: Run commands based on agent actions
- **Session management**: Setup and teardown
- **Output processing**: Transform results

### What Hooks Are NOT For

- **Command policing**: Don't block or prevent actions
- **Security validation**: That's the agent's judgment
- **Enforcement**: Agent should remain autonomous

### Example: Context Reminder Hook

```bash
#!/bin/bash
# hooks/session-start.sh
# Suggests relevant skills based on context

CURRENT_DIR=$(pwd)

if [[ "$CURRENT_DIR" == *"/"* ]]; then
  echo "Tip: When searching code, use the grep-skill for efficient searching."
fi

if [[ -f "package.json" ]]; then
  echo "Note: For npm tasks, use the npm-skill for correct command patterns."
fi
```

---

## Related Docs

- [Hooks Patterns](../patterns/03-hooks.md)
- [Plugin Development](../guides/02-plugin-development.md)
