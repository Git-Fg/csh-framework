# Platform Guide: Claude Code

This document covers the integration patterns, skill system, hooks, and session lifecycle when using the <framework> framework with Claude Code.

---

## How Skills Are Discovered and Loaded

### Discovery Mechanism

Claude Code automatically discovers skills from standard directories:

- Global skills: `~/.claude/skills/`
- Project-level skills: `.claude/skills/` in the project root

Skills are stored as individual directories containing a `SKILL.md` file with YAML frontmatter for metadata.

### Loading Behavior

Skills are loaded with metadata (YAML frontmatter) prioritized first to optimize prompt density. The agent automatically gains access to discovered skills without explicit registration.

When skills are bundled within plugins, they use namespaced notation: `/<plugin>:<skill>`.

### Configuration

Skill discovery can be influenced through:
- Project-level `.claude/settings.json`
- Global Claude Code settings

---

## Hook System

### Hook Configuration

Hooks are configured via `hooks/hooks.json` files located in:
- Plugin directories (for plugin-scoped hooks)
- Project root (for project-level hooks)

### Supported Events

Claude Code supports 14 distinct hook events:

| Category | Events |
|----------|--------|
| **Lifecycle** | `SessionStart`, `SessionEnd`, `Stop`, `TeammateIdle`, `TaskCompleted`, `PreCompact` |
| **Tool/Action** | `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` |
| **Agent/Team** | `SubagentStart`, `SubagentStop` |
| **System** | `Notification`, `UserPromptSubmit` |

### Handler Format

Hooks receive input as JSON on `stdin`. Handlers must parse this input (typically using `jq`) to extract relevant data. The handler executes its logic and returns results via `stdout`.

```json
{
  "hook": {
    "event": "PreToolUse",
    "id": "hook_abc123"
  },
  "payload": {
    "tool": "Bash",
    "input": { "command": "ls -la" }
  }
}
```

### Example Hook Handler

```bash
#!/bin/bash
# hooks/pre-tool-use.sh

read -r event
tool=$(echo "$event" | jq -r '.payload.tool')

if [[ "$tool" == "Bash" ]]; then
  # Inspect command before execution
  command=$(echo "$event" | jq -r '.payload.input.command')
  echo "Command to execute: $command"
fi
```

---

## Manifest Format

### Plugin Manifest

Claude Code plugins use `.claude-plugin/plugin.json`:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Example plugin for Claude Code",
  "skills": ["./skills"],
  "hooks": ["./hooks/hooks.json"]
}
```

### Skill Metadata (SKILL.md Frontmatter)

```markdown
---
name: example-skill
description: Example skill demonstrating Claude Code format
trigger: /example
platform: claude-code
---

# Example Skill

This skill demonstrates the format for Claude Code skills.
```

---

## Session Lifecycle

### Session Boundaries

Claude Code sessions are:
- **Interactive**: Tied to the process lifecycle
- **Identified**: Each session has a unique `CLAUDE_SESSION_ID`
- **Process-bound**: Sessions exist only while the Claude Code process is running

### Lifecycle Events

1. **SessionStart** - Fired when a new Claude Code session begins
2. **TaskCompleted** - Fired when a task finishes
3. **SessionEnd** - Fired when the session terminates
4. **Stop** - Fired when the session is interrupted

### State Management

- **In-memory**: Session state exists only in RAM
- **Session-bound**: State is not persisted across restarts
- **Process scope**: Each CLI invocation creates a new session

---

## Integration Patterns

### Hybrid Setup

When using Claude Code alongside other platforms:

1. **Naming Convention**: Use unique prefixes for skills to prevent collisions
   - Claude Code skills: `claude-<skill-name>`
   - Other platform skills: `<platform>-<skill-name>`

2. **Context Coordination**: Use ephemeral session notes for temporary context; persistent memory should be handled by the gateway layer

3. **Skill Discovery**: Claude Code skills are automatically available; ensure naming uniqueness to avoid conflicts

### Best Practices

- Keep skills focused and single-purpose
- Use namespaced skill names when bundling multiple skills
- Leverage project-level settings for environment-specific configuration
- Remember that session state is ephemeral

---

## Summary

| Aspect | Claude Code Behavior |
|--------|---------------------|
| **Skill Discovery** | Automatic from `~/.claude/skills/` |
| **Skill Loading** | Metadata-first for prompt optimization |
| **Hook Events** | 14 distinct events (lifecycle, tool, agent, system) |
| **Hook Format** | JSON on stdin, parse with `jq` |
| **Manifest** | `.claude-plugin/plugin.json` |
| **Session** | Process-bound, in-memory |
| **Persistence** | None (ephemeral) |

