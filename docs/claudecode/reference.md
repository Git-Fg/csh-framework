---
title: "Claude Code CLI Reference"
summary: "Common Claude Code CLI commands for CSH Framework development - plugin management, skills, settings."
read_when:
  - You need Claude Code CLI commands
  - You are debugging plugin issues
  - You want to manage skills/settings
---

# Claude Code CLI Reference

Common Claude Code CLI commands for CSH Framework development and testing.

---

## Plugin Management

### List Plugins

```bash
claude plugin list
```

Shows all installed plugins with their status and metadata.

### Install Plugin

```bash
# Install from local directory
claude plugin install ./my-plugin

# Install from npm registry
claude plugin install my-plugin@latest

# Install specific version
claude plugin install my-plugin@1.0.0
```

Installs a Claude Code plugin.

### Uninstall Plugin

```bash
claude plugin uninstall my-plugin
```

Removes a plugin from the system.

### Update Plugin

```bash
claude plugin update my-plugin
```

Updates a plugin to the latest version.

---

## Project Management

### Create New Project

```bash
claude new my-project
```

Creates a new Claude Code project.

### Initialize Existing Project

```bash
cd my-project
claude init
```

Initializes a Claude Code project in an existing directory.

### Open Project

```bash
claude open
```

Opens the current project in Claude Code.

---

## Session Management

### Start Session

```bash
claude
# Or with specific model
claude --model claude-sonnet-4

# With specific agent
claude --agent Explore
```

Starts a new Claude Code session.

### Resume Session

```bash
claude --continue
```

Resumes the previous session.

### Clear Session

```bash
claude --clear
```

Starts a fresh session without previous context.

---

## Agent Management

### List Agents

```bash
claude agent list
```

Lists all available agents.

### Show Agent Info

```bash
claude agent info Explore
```

Shows detailed information about a specific agent.

---

## Configuration

### Configuration File Location

Claude Code configuration is stored in:
```
~/.claude/settings.json
```

### Project Configuration

Project-specific settings:
```
.claude/settings.json
```

### Configuration Schema

Common configuration sections:

```json
{
  "skills": {
    "my-skill": {
      "enabled": true
    }
  },
  "plugins": {
    "my-plugin": {
      "enabled": true,
      "config": {}
    }
  }
}
```

### Environment Variables

| Variable | Description |
|-----------|-------------|
| `CLAUDE_PLUGIN_ROOT` | Absolute path to plugin directory (available in hooks) |

---

## Skills

### List Available Skills

```bash
# View available skills in current session
# Use within Claude Code session:
/skills
```

Shows all available skills in the current session.

### Invoke Skill

```bash
# Within Claude Code session:
/my-skill

# Plugin skill:
/my-plugin:skill-name
```

Invokes a specific skill.

---

## Plugin Development

### Plugin Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── my-skill/
│       └── SKILL.md
└── hooks/
    ├── session-start.sh
    └── pre-tool-use.sh
```

### Plugin Manifest (plugin.json)

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "A CSH Framework plugin",
  "publisher": "my-name",
  "skills": {
    "my-skill": {
      "description": "Search across all memories"
    }
  },
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Testing & Debugging

### Enable Debug Mode

```bash
claude --debug
```

Enables verbose debugging output.

### View Logs

Claude Code logs are available via:
```
Claude Code > View > Logs
```

### Test Plugin Installation

```bash
# Install plugin in development
claude plugin install ./my-plugin

# Start Claude Code with plugin
claude

# Within session, verify plugin loaded
# Check for plugin skills:
/skills
```

### Test Hooks

```bash
# Test hook script manually
cd my-plugin
echo '{"hook":{"event":"SessionStart"},"payload":{}}' | ./hooks/session-start.sh

# Test with sample PreToolUse event
echo '{"hook":{"event":"PreToolUse"},"payload":{"tool":"Bash","input":{"command":"ls"}}}' | ./hooks/pre-tool-use.sh
```

---

## Common Patterns

### Development Workflow

```bash
# 1. Create plugin structure
mkdir my-plugin
cd my-plugin
claude plugin init

# 2. Add skills and hooks
# Create skills/ and hooks/ directories

# 3. Test locally
claude plugin install .
claude

# 4. Verify functionality
# Within session, test skills and hooks

# 5. Iterate
# Make changes
# Restart session: claude --clear
```

### Hot Reload During Development

```bash
# Edit skill files in my-plugin/skills/
# Start new session to reload
claude --clear

# Edit hook scripts
# Hooks reload on session restart
claude --continue  # Maintains context, reloads hooks
```

### Troubleshooting Plugin Issues

```bash
# Check plugin is installed
claude plugin list | grep my-plugin

# View plugin details
claude plugin info my-plugin

# Check for errors in logs
# Claude Code > View > Logs

# Test plugin manifest
cat .claude-plugin/plugin.json | jq .

# Test hook scripts
./hooks/my-hook.sh < test-input.json
```

---

## Exit Codes

| Code | Meaning |
|-------|---------|
| 0 | Success |
| 1 | Generic error |
| 2 | Invalid usage (wrong parameters) |
| 130 | Interrupted by user (Ctrl+C) |

---

## References

- **Claude Code Documentation**: [code.claude.com/docs](https://code.claude.com/docs)
- **Skill Format**: [SKILL.md Format Specification](../openclaw/skill-format.md)
- **Skill System**: [Claude Code Skills](./skills.md)
- **Hook System**: [Claude Code Hooks](./hooks.md)
- **Core Concepts**: [CSH Core Concepts](../core/)

---

*See Also*: [Claude Code Overview](./01-overview.md) for general platform information
