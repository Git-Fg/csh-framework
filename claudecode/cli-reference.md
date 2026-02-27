---
title: "Claude Code CLI Reference"
summary: "Command reference for Claude Code CLI commands relevant to plugin development."
read_when:
  - You need to install/uninstall plugins
  - You want to validate plugin configuration
  - You are debugging plugin issues
---

# Claude Code CLI Reference

Commands for managing plugins and debugging in Claude Code.

---

## Plugin Commands

### Install Plugin

```bash
# Install from local directory
claude plugin install ./my-plugin

# Install with specific scope
claude plugin install ./my-plugin --scope user    # Default
claude plugin install ./my-plugin --scope project  # Project-scoped

# Install from URL
claude plugin install https://example.com/plugins/my-plugin.tar.gz
```

### Uninstall Plugin

```bash
claude plugin uninstall my-plugin
claude plugin uninstall my-plugin --force  # Skip confirmation
```

### Enable/Disable

```bash
claude plugin enable my-plugin
claude plugin disable my-plugin
```

### Update Plugin

```bash
claude plugin update my-plugin
claude plugin update --all  # Update all plugins
```

### List Plugins

```bash
claude plugin list                    # List installed
claude plugin list --scope user       # User-scoped only
claude plugin list --scope project    # Project-scoped only
```

### Validate Plugin

```bash
claude plugin validate                 # Validate all
claude plugin validate ./my-plugin    # Validate specific
```

---

## Debug Commands

### Debug Mode

```bash
# Run with debug output
claude --debug

# Debug specific plugin
claude --debug --plugin-dir ./my-plugin

# Verbose output
claude -vvv
```

### View Logs

```bash
# View Claude Code logs (platform-specific)
# macOS: ~/Library/Logs/Claude/
# Linux: ~/.claude/logs/
# Windows: %APPDATA%/Claude/logs/
```

---

## Session Commands

### Start Session

```bash
# Interactive session
claude

# With specific project
claude ./my-project

# With initial prompt
claude "Your task description"
```

### Session Variables

| Variable | Description |
|----------|-------------|
| `CLAUDE_SESSION_ID` | Unique session identifier |
| `CLAUDE_PROJECT_ROOT` | Project root directory |
| `CLAUDE_PLUGIN_ROOT` | Plugin root (when in plugin context) |

---

## Skills Commands

### List Skills

```bash
# List all available skills
claude --skills-list

# Or check in Claude Code:
/skills
```

---

## Common Issues

### Plugin Not Loading

```bash
# 1. Validate plugin
claude plugin validate ./my-plugin

# 2. Check debug output
claude --debug

# 3. Verify manifest
cat my-plugin/.claude-plugin/plugin.json | jq .
```

### Hooks Not Firing

```bash
# 1. Make scripts executable
chmod +x my-plugin/hooks/*.sh

# 2. Validate hook config
cat my-plugin/hooks/hooks.json | jq .

# 3. Test manually
echo '{}' | ./my-plugin/hooks/test-hook.sh
```

---

## References

- [Claude Code Overview](./overview.md)
- [Claude Code Hooks](./hooks.md)
- [Claude Code Skills](./skills.md)
