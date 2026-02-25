# Claude Code Hooks

Event-driven hooks for automation and context injection in Claude Code.

---

## Hook Configuration

Claude Code hooks are configured via JSON in the plugin manifest:

```json
{
  "name": "my-plugin",
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

### Hook Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── hooks/
    ├── session-start.sh
    └── pre-tool-use.sh
```

---

## Supported Events

Claude Code supports hook events across multiple categories:

| Category | Events |
|----------|---------|
| **Lifecycle** | `SessionStart`, `SessionEnd`, `Stop`, `TeammateIdle`, `TaskCompleted`, `PreCompact` |
| **Tool/Action** | `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` |
| **Agent/Team** | `SubagentStart`, `SubagentStop` |
| **System** | `Notification`, `UserPromptSubmit` |

### Event Details

#### SessionStart
- **Fires**: When session starts (new or resumed)
- **Use**: Load context, initialize state, set environment variables
- **Matcher Values**: `startup`, `resume`, `clear`, `compact`

#### SessionEnd
- **Fires**: When session ends
- **Use**: Cleanup, checkpointing, analytics

#### PreToolUse
- **Fires**: Before a tool is invoked
- **Use**: Validation, context injection, logging
- **Matcher**: Tool name (e.g., `Bash`, `Read`, `Write`)

#### PostToolUse
- **Fires**: After a tool completes
- **Use**: Output processing, logging, validation
- **Matcher**: Tool name

#### PostToolUseFailure
- **Fires**: When a tool execution fails
- **Use**: Error recovery, logging, fallback logic

#### Stop
- **Fires**: When agent is stopped
- **Use**: Final cleanup, state persistence

---

## Hook Execution Mechanisms

Claude Code provides three mechanisms for hook execution:

### 1. Command Hook (Shell Execution)

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

**Use Cases**:
- Formatting code after edits
- Logging tool usage
- Validating outputs
- Running tests after code changes

### 2. Prompt Hook (LLM Evaluation)

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

**Use Cases**:
- Safety checks on commands
- Semantic analysis of inputs
- Complex decision logic
- Content filtering

### 3. Agent Hook (Agentic Verification)

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

**Use Cases**:
- Security reviews
- Complex validation
- Multi-step verification
- Quality assurance

---

## Handler Format

### Shell Script Handler

Hooks receive input as JSON on `stdin`:

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

**Example Handler**:
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

### Environment Variables

| Variable | Description |
|-----------|-------------|
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to plugin directory |

Always use this variable for paths inside your plugin:

```bash
command="${CLAUDE_PLUGIN_ROOT}/scripts/my-script.sh"
```

---

## Hook Best Practices

### 1. Use Correct Event Names

```json
// Good: Correct event name
{
  "hooks": {
    "SessionStart": [ ... ]  // Correct
  }
}

// Bad: Incorrect event name
{
  "hooks": {
    "session_start": [ ... ]  // Wrong - case sensitive
  }
}
```

### 2. Matcher Patterns

Use matcher to control when hooks fire:

```json
// Fire on multiple tools
{
  "matcher": "Write|Edit|Create"
}

// Fire on all
{
  "matcher": "*"
}

// Fire on specific matcher value
{
  "matcher": "startup"
}
```

### 3. Keep Hooks Lightweight

```bash
# Good: Fast, minimal work
#!/bin/bash
read -r event
echo "✅ Hook fired"

# Bad: Heavy computation
#!/bin/bash
read -r event
# Heavy operation that takes 30 seconds!
long_running_task
```

### 4. Use Hooks for Guidance, Not Implementation

```bash
# Good: Guidance
echo "Use 'my-skill' for searching files."

# Bad: Implementation
# Don't implement skills in hooks!
search_files() { ... }
```

---

## Automation vs. Injection

### Automation Hooks

Perform side effects without returning text to agent:

```bash
#!/bin/bash
read -r event

# Just log, don't return text
echo "$(date): Tool used" >> ~/.claude/hooks.log
```

### Injection Hooks

Return text to inject into agent context:

```bash
#!/bin/bash
read -r event

# Return guidance for agent
echo "💡 TIP: Use 'my-skill' for this task."
```

---

## Troubleshooting

### Hook Not Firing

**Symptom**: Hook configured but never executes

**Check**:
```bash
# Verify event name (case-sensitive)
# Correct: "SessionStart"
# Incorrect: "session_start", "SESSIONSTART"

# Check matcher pattern
# Use correct tool names: "Bash", "Read", "Write"

# Verify script is executable
chmod +x hooks/my-hook.sh

# Check for errors in Claude Code logs
# View: Claude Code > View > Logs
```

### Hook Syntax Errors

**Symptom**: Hook registered but fails to execute

**Check**:
```bash
# Test script manually
echo '{"tool":"Bash"}' | jq .

# Test hook with sample input
echo '{"hook":{"event":"PreToolUse"},"payload":{"tool":"Bash"}}' | ./hooks/my-hook.sh

# Check bash syntax
bash -n hooks/my-hook.sh
```

### Hook Returns Wrong Output

**Symptom**: Hook executes but output is not as expected

**Check**:
```bash
# Verify output format
# Automation hooks: No output needed
# Injection hooks: Plain text output

# Debug output
./hooks/my-hook.sh 2>&1 | tee /tmp/hook-debug.log
```

---

## References

- **Hook Events**: [Complete Hook Events Reference](../05-reference/02-hook-events.md)
- **Hook Patterns**: [Hook Design Patterns](../02-core-concepts/04-hooks.md)
- **Best Practices**: [Testing Validation Best Practices](../02-core-concepts/06-best-practices.md)
- **Skill System**: [Claude Code Skills](./02-skills.md)

---

*See Also*: [Claude Code Overview](./01-overview.md) for general platform information
