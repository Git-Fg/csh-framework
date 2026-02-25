# OpenClaw CLI Reference

Common OpenClaw CLI commands for CSH Framework development and testing.

---

## Plugin Management

### List Plugins

```bash
openclaw plugins list
```

Shows all installed plugins with their status and metadata.

### Install Plugin

```bash
# Standard install
openclaw plugins install /path/to/plugin

# Development mode (hot reload)
openclaw plugins install -l /path/to/plugin
```

Installs a plugin. Use `-l` flag for development to enable hot reload.

### Uninstall Plugin

```bash
openclaw plugins uninstall <plugin-id>
```

Removes a plugin from the system.

---

## Skill Management

### List Skills

```bash
openclaw skills list

# List only enabled skills
openclaw skills list --enabled
```

Shows all discovered skills and their status.

### Enable Skill

```bash
openclaw skills enable <skill-name>
```

Enables a skill, making it available to agents.

### Disable Skill

```bash
openclaw skills disable <skill-name>
```

Disables a skill, removing it from agent context.

### Show Skill Details

```bash
openclaw skills show <skill-name>
```

Displays detailed information about a specific skill.

### Reindex Skills

```bash
openclaw skills reindex
```

Forces re-scanning of skill directories and reloading.

---

## Gateway Management

### Gateway Status

```bash
openclaw gateway status
```

Shows gateway state (running/stopped), control plane status, and RPC probe.

### Start Gateway

```bash
openclaw gateway start

# Start with debug logging
openclaw gateway start --debug
```

Starts the OpenClaw gateway.

### Stop Gateway

```bash
openclaw gateway stop
```

Stops the OpenClaw gateway.

### Restart Gateway

```bash
openclaw gateway restart

# Restart with specific configuration
openclaw gateway restart --config /path/to/config.json
```

Restarts the OpenClaw gateway.

---

## Health & Diagnostics

### Health Check

```bash
openclaw health
```

Comprehensive health check including gateway, models, and dependencies.

### Gateway Health

```bash
openclaw gateway health
```

Quick health check for gateway status.

### Logs

```bash
# Follow logs in real-time
openclaw logs --follow

# Show last N lines
openclaw logs --tail 50

# Show only errors
openclaw logs --level error

# Show JSON output
openclaw logs --json
```

View and search gateway logs.

### Config Management

```bash
# Get current configuration
openclaw config get

# Get specific config value
openclaw config get plugins.entries

# Validate configuration
openclaw config validate
```

View and validate configuration.

---

## Session Management

### List Sessions

```bash
openclaw session list

# Show active sessions only
openclaw session list --active

# Show details
openclaw session list --details
```

Lists all sessions with metadata.

### Stop Session

```bash
openclaw session stop <session-id>
```

Stops a running session.

### Start Session

```bash
openclaw session start --model <model-id>

# Start with specific agent
openclaw session start --agent-id <agent-id>
```

Starts a new session.

---

## Agent Management

### List Agents

```bash
openclaw agent list
```

Lists all configured agents.

### Agent Status

```bash
openclaw agent get --agent-id <agent-id>
```

Shows detailed agent information and status.

---

## Configuration

### Configuration File Location

OpenClaw configuration is stored in:
```
~/.openclaw/openclaw.json
```

### Configuration Schema

Common configuration sections:

```json
{
  "plugins": {
    "entries": {
      "my-plugin": {
        "enabled": true,
        "config": {}
      }
    }
  },
  "skills": {
    "enabled": ["skill-name"]
  },
  "gateway": {
    "host": "127.0.0.1",
    "port": 18789
  }
}
```

### Environment Variables

| Variable | Description |
|-----------|-------------|
| `OPENCLAW_WORKSPACE` | Path to workspace directory |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication token |
| `OPENCLAW_CONFIG` | Path to configuration file |

---

## Common Patterns

### Hot Reload During Development

```bash
# Install plugin in development mode
openclaw plugins install -l /path/to/my-plugin

# Make changes to plugin files
# No restart needed - changes auto-detected

# Verify changes loaded
openclaw plugins list | grep my-plugin
```

### Testing Plugin Installation

```bash
# 1. Check gateway health
openclaw health

# 2. Install plugin
openclaw plugins install /path/to/my-plugin

# 3. Verify plugin loaded
openclaw plugins list | grep my-plugin

# 4. Verify skills discovered
openclaw skills list | grep my-skill

# 5. Check logs for errors
openclaw logs --follow --tail 20 | grep -i error
```

### Troubleshooting Plugin Issues

```bash
# Check plugin status
openclaw plugins list | grep my-plugin

# View detailed logs
openclaw logs --follow --tail 100 | grep my-plugin

# Test plugin installation
openclaw plugins install --dry-run /path/to/my-plugin

# Validate configuration
openclaw config validate
```

### Session Testing via HTTP

```bash
# Test agent interaction
curl -s -X POST http://127.0.0.1:18789/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN" \
  -d '{
    "model": "agent:main",
    "messages": [
      {"role": "user", "content": "Hello from OpenClaw"}
    ]
  }'
```

---

## Exit Codes

| Code | Meaning |
|-------|---------|
| 0 | Success |
| 1 | Generic error |
| 2 | Invalid usage (wrong parameters) |
| 4 | Not found |
| 5 | Permission denied |

---

## References

- **OpenClaw Documentation**: [docs.openclaw.ai](https://docs.openclaw.ai/)
- **Testing Guide**: [OpenClaw Testing Guide](./04-testing.md)
- **Skill System**: [OpenClaw Skills](./02-skills.md)
- **Hook System**: [OpenClaw Hooks](./03-hooks.md)
- **Core Concepts**: [CSH Core Concepts](../02-core-concepts/)

---

*See Also*: [OpenClaw Overview](./01-overview.md) for general platform information
