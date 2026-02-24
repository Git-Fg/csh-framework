# Hook Patterns

How to use hooks in your plugins to automate functionality and provide context.

---

## What Hooks Do

Hooks let your plugin **react to events** in the agent session:

| Use Case | Example |
|----------|---------|
| **Automation** | Run commands when agent does something |
| **Context** | Inject reminders about your skills |
| **Logging** | Track agent activities |

---

## Hook Types

### Context Reinforcement

Remind the agent to use your skills at the right time:

```bash
#!/bin/bash
# hooks/session-start.sh

echo "Skills available:"
echo "- Use 'db-query' for database operations"
echo "- Use 'migrate' for schema changes"
```

This injects context when session starts, helping agent discover your skills.

### Automation

Run commands automatically when agent uses certain tools:

```bash
#!/bin/bash
# hooks/after-command.sh

# Read the command that was executed
COMMAND=$(jq -r '.tool_name' /dev/stdin)

if [[ "$COMMAND" == "Bash" ]]; then
  # Log command for analytics
  echo "$(date): Agent ran bash command" >> ~/.csh/logs/activity.log
fi
```

### Output Processing

Transform or enrich tool output:

```bash
#!/bin/bash
# Process git output to show cleaner info
git status --porcelain | awk '{print "Changed:", $2}'
```

---

## How to Register Hooks

### OpenClaw

In your `openclaw.plugin.json`:

```json
{
  "id": "my-plugin",
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/start.sh",
      "description": "Load plugin context"
    },
    {
      "event": "after:command",
      "handler": "./hooks/track.sh",
      "description": "Track command usage"
    }
  ]
}
```

### Claude Code

In your `.claude-plugin/plugin.json`:

```json
{
  "hooks": {
    "events": {
      "SessionStart": {
        "command": "./hooks/start.sh"
      }
    }
  }
}
```

---

## Best Practices

### DO

- **Inject context**: Remind agent about your skills
- **Automate setup**: Prepare environment for your tools
- **Track usage**: Understand how agent uses your plugin

### DON'T

- **Block commands**: Agent autonomy is key
- **Police behavior**: Don't try to prevent actions
- **Add friction**: Keep hooks lightweight

---

## Example: Database Plugin Hooks

```bash
# hooks/session-start.sh
echo "Database Plugin: Use 'db-query' for safe queries"

# hooks/after-command.sh
# Auto-commit queries to audit log
jq -r '.tool_name' /dev/stdin | grep -q "Bash" && \
  echo "Query executed at $(date)" >> ~/.csh/logs/queries.log
```

---

## Event Reference

See [Hook Events Reference](../reference/03-hook-events.md) for all available events.
