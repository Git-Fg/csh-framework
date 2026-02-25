# Platform Guide: Claude Code

> **Source**: [Claude Code Documentation](https://code.claude.com/docs/)

This document covers the integration patterns for building CSH framework extensions with Claude Code.

---

> **Scope**: The CSH framework is designed primarily for **Claude Code** and **OpenClaw** AI agents, each with specific integration patterns. However, the architecture (CLI + Skills + Hooks) is generic — it works with **any AI agent platform that supports plugin bundling** containing these three primitives.

---

## CSH Framework on Claude Code

The CSH framework (CLI + Skills + Hooks) requires **all three components working together**:

| Component | Role | In Claude Code |
|-----------|------|----------------|
| **CLI Primitives** | The actual tools/commands agents execute | Bash, Read, Edit, Write, Glob, Grep, etc. |
| **Skills** | Guidance that teaches agents correct patterns | `SKILL.md` files with YAML frontmatter |
| **Hooks** | Automation that injects context + enforces rules | `hooks/hooks.json` event handlers |

> ⚠️ **Critical**: The CSH framework is NOT "use hooks OR skills". It's **CLI + Skills + Hooks** as a unified system. Each component has a distinct purpose and they're designed to work together.

---

## How Skills Are Discovered and Loaded

### Discovery Mechanism

Claude Code discovers skills bundled within plugins. When a plugin is installed via `claude plugin install`, it automatically scans the directory for skills defined in the manifest.

- Skills in plugins are namespaced: `/<plugin>:<skill>`
- The agent automatically gains access to discovered skills without further registration.

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
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Example plugin for CSH framework on Claude Code",
  "author": { "name": "Your Name", "url": "https://github.com/you" },
  "homepage": "https://docs.example.com",
  "skills": ["./skills"],
  "hooks": ["./hooks/hooks.json"]
}
```

> **Note**: The manifest is optional. If omitted, Claude Code auto-discovers components and derives the plugin name from the directory name.

### Required vs Optional Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (kebab-case) |
| `version` | No | Semantic version (e.g., "1.2.0") |
| `description` | No | Brief explanation |
| `author` | No | Author info object |
| `homepage` | No | Documentation URL |
| `skills` | No | Skill directory paths |
| `hooks` | No | Hook config paths |

### Complete Plugin Structure

```
my-csh-plugin/
├── .claude-plugin/
│   └── plugin.json           # Manifest (optional)
├── skills/                  # Agent Skills directories
│   └── my-skill/
│       └── SKILL.md
├── hooks/
│   └── hooks.json          # Hook configuration
├── commands/               # Simple commands (legacy)
├── agents/                 # Custom subagents
├── scripts/                # Hook utility scripts
│   └── my-hook.sh
├── settings.json          # Default settings
└── README.md
```

---

## Installation Scopes

When installing a CSH plugin, use the native `claude plugin install` command with the appropriate scope:

| Scope | Command Example | Use Case |
|-------|-----------------|----------|
| `user` | `claude plugin install ./my-plugin` | Personal plugins (default) |
| `project` | `claude plugin install ./my-plugin --scope project` | Team/shared via git |

> **Portability Note**: Avoid manually copying files to `~/.claude/skills/`. Using the `plugin install` command ensures that the manifest is correctly processed and hooks are registered.


---

## Hook Execution Mechanisms

Claude Code provides three mechanisms for hook execution, each suited to different automation needs:

### Command Hook (Shell Execution)

Execute shell scripts for deterministic automation:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

### Prompt Hook (LLM Evaluation)

Use the LLM for complex decisions and semantic analysis:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Is this command safe? $ARGUMENTS"
          }
        ]
      }
    ]
  }
}
```

### Agent Hook (Agentic Verification)

Spawn a sub-agent for sophisticated verification tasks:

```json
{
  "hooks": {
    "TaskCompleted": [
      {
        "type": "agent",
        "agent": "security-reviewer",
        "prompt": "Verify this change is secure: $ARGUMENTS"
      }
    ]
  }
}
```

---

## Environment Variables

### `${CLAUDE_PLUGIN_ROOT}`

Always use this variable for paths inside your plugin:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/process.sh"
      }
    ]
  }
}
```

> ⚠️ **Critical**: Never use absolute paths. All paths must be relative and start with `./`.

---

## Plugin CLI Commands

```bash
# Install plugin
claude plugin install my-plugin

# Uninstall
claude plugin uninstall my-plugin

# Enable/disable
claude plugin enable my-plugin
claude plugin disable my-plugin

# Update
claude plugin update my-plugin

# Validate (debug)
claude plugin validate

# Test with --debug flag
claude --debug --plugin-dir ./my-plugin
```

---

## Debugging Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Plugin not loading | Invalid JSON | Run `claude plugin validate` |
| Commands not appearing | Wrong directory | Ensure at root, not in `.claude-plugin/` |
| Hooks not firing | Script not executable | `chmod +x script.sh` |
| Path errors | Absolute paths used | Use `${CLAUDE_PLUGIN_ROOT}` |

### Debug Checklist

1. Run `claude --debug` to see loading details
2. Check that each component directory is at root level
3. Verify file permissions allow reading

---

### Skill Metadata (SKILL.md Frontmatter)

```markdown
---
name: example-skill
description: Example skill demonstrating Claude Code format
trigger: /example
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

## Memory System

### No Native Memory Plugins

Unlike OpenClaw, Claude Code does **not** have built-in memory plugins (memory-core, LanceDB). Sessions are ephemeral by design.

### CSH Approach: CLI + Skills + Hooks for Persistence

Since there's no native memory to "enhance", the CSH framework builds memory using:

1. **CLI tools** for persistence:
   - File-based storage (JSON, SQLite, etc.)
   - Use filesystem for state (`~/.claude/projects/` or custom locations)

2. **Skills** that teach agents how to store/retrieve:
   - Memory save patterns
   - Memory search patterns
   - Context injection workflows

3. **Hooks** that automate persistence:
   - `SessionEnd`: Save session summary to file
   - `SessionStart`: Load previous context
   - `PreToolUse`: Intercept important information to remember

### Example: Minimal Memory Pattern

```bash
# In a skill - teach agent to remember
# Remember important info: echo '{"type":"note","content":"..."}' >> ~/.claude/memory.jsonl
```

```json
// hooks/hooks.json - auto-save on session end
{
  "hooks": {
    "SessionEnd": [{
      "command": "save-session-context.sh"
    }]
  }
}
```

> **Key Difference**: On OpenClaw, you enhance native memory. On Claude Code, you build memory from scratch using CLI + Skills + Hooks.

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

