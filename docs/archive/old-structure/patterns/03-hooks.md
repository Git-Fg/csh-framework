# Hook Patterns

How to use hooks in your plugins to automate functionality and provide context.

---

## The "Automate + Inject" Methodology

A well-designed hook performs **two distinct roles simultaneously**:

### 1. AUTOMATE (The Side Effect)
Performing silent technical tasks without requiring agent intervention:
- Reindexing databases
- Creating checkpoints
- Cleaning up temporary files
- Preparing environments

### 2. ELICIT (The Instruction/Elicitation)
Providing text guidance back to the agent to trigger (elicit) the correct skill usage:
- "⚠️ PRODUCTION MODE: Use `dry-run` first"
- "✅ VAULT STATUS: Healthy (47 memories loaded)"
- "ℹ️ TIP: Use `db-query` skill for this operation"

**Goal**: Eliminate "mode-switching penalty" by making the right choice obvious through **Capacity of Elicitation**.

> **Critical Insight**: Current LLMs struggle with skill invocation (~30% rate even with perfect descriptions). Hooks work around this limitation by providing gentle guidance at key moments.

---

## What Hooks Do

Hooks let your plugin **react to events** in the agent session:

| Use Case | Example |
|----------|---------|
| **Automation** | Run commands when agent does something |
| **Context** | Inject reminders about your skills |
| **Logging** | Track agent activities |

---

## Infrastructure Patterns: Claude Code vs. OpenClaw

> **Sources**: [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks), [OpenClaw Plugins Documentation](https://docs.openclaw.ai/plugins)

### Logic Type Differences

| Feature | Claude Code Hooks | OpenClaw Hooks |
|---------|-------------------|----------------|
| **Logic Type** | Shell/Binary (`command`) or LLM (`prompt`) | TypeScript/Node.js Handler (Full API) |
| **Automation** | Via `type: command` scripts | Via synchronous side effects in `handler.cjs` |
| **Injection** | Via script `stdout` or `type: prompt` | Via return string from handler |

### Event Mapping

| Framework Phase | Claude Code Event | OpenClaw Event |
|-----------------|-------------------|----------------|
| **Bootstrap** | `SessionStart` | `agent:bootstrap` / `session_start` |
| **In-Turn Logic** | `PreToolUse` | `command:new` / `before_command` |
| **Clean Exit** | `SessionEnd` / `Stop` | `session_end` / `command:stop` |
| **Recovery** | `PostToolUseFailure` | `session_recover` |

---

## Leveraging Hooks in CSH Projects

Hooks are **composable primitives** that integrate automation into every phase of CSH project development. Rather than a fixed set of "hook types," think of hooks as tools you combine and orchestrate throughout the project lifecycle.

### Project Lifecycle Integration

| Phase | Hook Opportunities | Value |
|-------|-------------------|-------|
| **Project Creation** | Session start, Bootstrap | Load project context, initialize environments |
| **Active Development** | Before/After Command | Enforce patterns, track changes, log activity |
| **Code Review/Refinement** | Post Tool Use | Validate outputs, enforce standards |
| **Maintenance** | Session End, Periodic | Cleanup, reindex, checkpoint |

### Common Hook Combinations

**Bootstrap + Context Reinforcement**
```bash
# hooks/session-start.sh
# AUTOMATE: Initialize environment
mkdir -p ~/.my-plugin/state && touch ~/.my-plugin/state/last-session

# ELICIT: Remind agent of available capabilities
echo "✅ CSH Framework loaded"
echo "  - Use 'create-plan' for new features"
echo "  - Use 'implement-plan' for execution"
```

**Detection + Skill Redirection**
```bash
# hooks/before-command.sh
# AUTOMATE: Detect pattern
if grep -qE "grep.*-r.*\.md$" <<< "$COMMAND"; then
  # ELICIT: Suggest specialized skill
  cat <<EOF

ℹ️ Detected: Searching markdown files

TIP: Use 'content-search' skill for:
  - Markdown-aware parsing
  - Frontend metadata filtering
  - Section-level relevance scoring

EOF
fi
```

**Execution + Validation**
```bash
# hooks/after-command.sh
# AUTOMATE: Track changes
echo "$(date): $COMMAND" >> ~/.my-plugin/audit.log

# ELICIT: Validate if needed
if [[ "$COMMAND" =~ openclaw.*config.* ]]; then
  echo "⚠️ Config modified. Run 'verify-config' to validate."
fi
```

### Key Patterns for CSH Projects

| Pattern | Event | Use Case in CSH |
|---------|-------|-----------------|
| **Context Loading** | SessionStart | Load project rules, constraints, decisions |
| **Skill Orchestration** | PreToolUse/before_command | Redirect to specialized CSH skills |
| **Token Optimization** | PostToolUse/after_command | Process CLI outputs before LLM sees them |
| **Guard Rails** | PreToolUse/before_command | Enforce project conventions |
| **Audit Trail** | AfterCommand/after_command | Track all agent actions |
| **Environment Sync** | SessionStart/recover | Ensure tooling, configs in correct state |

---

## Design Patterns: CLI + Skills Orchestration

### Pattern 1: Skill Redirection (Targeted Injection)

Detect when an agent uses a raw CLI primitive for a task that has a specialized Skill.

**Goal**: Counteract "mode-switching resistance."

**Event**: `PreToolUse` (Claude) or `before_command` (OpenClaw).

**Example**:
```bash
#!/bin/bash
# hooks/before-command.sh

# Check if agent is searching codebase with grep
if grep -q "grep.*-r.*auth" <<< "$COMMAND"; then
  echo "ℹ️ TIP: You are searching the codebase."
  echo "Use 'read-codebase-patterns' Skill for localized insight and 90% better token efficiency."
fi
```

### Pattern 2: Context Reinforcement (Mandatory Injection)

Inject high-level constraints that Skills rely on for consistency.

**Goal**: Maintain Vault integrity.

**Event**: `SessionStart`.

**Example**:
```bash
#!/bin/bash
# hooks/session-start.sh

echo "⚠️ RULE: DO NOT use native memory tools."
echo "ALWAYS use CLI templates then Edit tool iteration."
echo ""
echo "Vault status: $(clawvault status --short)"
```

### Pattern 3: Automated Synthesis (Deterministic Automation)

Pre-process raw CLI output so that Skills/LLM don't have to.

**Goal**: Massive token reduction.

**Event**: `PostToolUse` / `after_command`.

**Example**:
```bash
#!/bin/bash
# hooks/after-command.sh

# If agent ran a heavy log-read CLI, intercept and process
if grep -q "journalctl.*-n 1000" <<< "$COMMAND"; then
  # Extract only error codes instead of full logs
  journalctl -n 1000 | grep ERROR | awk '{print $NF}' > /tmp/errors.txt
  cat /tmp/errors.txt
else
  # Passthrough normal commands
  eval "$COMMAND"
fi
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

**Handler Implementation** (TypeScript/Node.js):

```javascript
// hooks/start.cjs
module.exports = async function handler(context) {
  const { config, logger, runtime } = context;

  // AUTOMATE: Prepare environment
  await runtime.exec('mkdir -p ~/.my-plugin/logs');

  // INJECT: Provide context to agent
  return `
Skills available:
- Use 'db-query' for database operations
- Use 'migrate' for schema changes

Environment: ${config.env || 'production'}
  `;
};
```

### Claude Code

In your `.claude-plugin/plugin.json`:

```json
{
  "hooks": {
    "events": {
      "SessionStart": {
        "command": "./hooks/start.sh"
      },
      "PreToolUse": {
        "command": "./hooks/before-tool.sh"
      }
    }
  }
}
```

**Handler Implementation** (Shell):

```bash
#!/bin/bash
# hooks/before-tool.sh

# Read tool name from stdin
TOOL_NAME=$(jq -r '.tool_name' /dev/stdin)

# Inject context for specific tools
case "$TOOL_NAME" in
  "Bash")
    echo "ℹ️ Running bash command. Use 'db-query' skill for database operations."
    ;;
  "ReadFile")
    echo "ℹ️ Reading file. Consider using 'vault-search' for vault operations."
    ;;
esac
```

---

## Best Practices

### Be Clear, Concise, and Specific (Pedagogy)

The "Inject" part of your hook must be designed for maximum impact with minimum noise.

**DO**:
- ✅ State exact condition and single action required
- ✅ Use bullet points and bold emphasis for critical instructions

**DON'T**:
- ❌ Inject long, rambling paragraphs that bloat context window
- ❌ Use vague warnings like "Be careful" or "Don't do that"

**Example**:
```bash
# Bad
echo "⚠️ WARNING: Please be careful because this command modifies system state and might cause issues with the database if you haven't checked the backup logs yet today."

# Good
echo "⚠️ CHECK: Verify database backup. Current: main. Action: Run \`bin backup status\` before proceeding."
```

### Idempotency is Mandatory (Hygiene)

Automation side-effects must be safe to run multiple times without unintended consequences.

**Pattern**: Use `mkdir -p`, `ln -sf`, or check for state flags.

```bash
#!/bin/bash
# Good: Idempotent
if [[ ! -f "$SETUP_DONE_FLAG" ]]; then
  mkdir -p ~/.my-plugin/logs
  touch "$SETUP_DONE_FLAG"
fi

# Bad: Not idempotent
mkdir ~/.my-plugin/logs  # Fails if exists
```

### Keep Hooks Lightweight

Hooks should execute in milliseconds, not seconds.

```bash
# Good: Fast checks
if [[ ! -f ~/.my-plugin/config.yml ]]; then
  echo "⚠️ Configuration missing. Run: myplugin init"
fi

# Bad: Slow operations
# Don't do this in a hook!
docker pull my-image  # Takes minutes
```

---

## Example: Database Plugin Hooks

```bash
# hooks/session-start.sh
echo "Database Plugin: Use 'db-query' for safe queries"
echo "Current database: $(cat ~/.myplugin/db-name)"

# hooks/after-command.sh
# Auto-commit queries to audit log
jq -r '.tool_name' /dev/stdin | grep -q "Bash" && \
  echo "Query executed at $(date)" >> ~/.openclaw/logs/queries.log
```

---

## Hook Workflow: End-to-End Example

### Scenario: Agent should use "vault-search" skill

**Problem**: Agent uses `grep` directly, missing specialized skill.

**Solution**: Hook detects and redirects.

```bash
# hooks/before-command.sh
#!/bin/bash

COMMAND=$(cat)

# Pattern: Agent searching vault directory with grep
if grep -qE "grep.*-r.*~/vault" <<< "$COMMAND"; then
  # INJECT: Gentle reminder
  cat <<EOF

ℹ️ DETECTED: You're searching the vault directory.

TIP: Use 'vault-search' skill for better results:
  - Understands vault structure
  - Filters by metadata
  - Returns relevant results only
  - 85% token reduction

Example:
  vault-search "authentication"

EOF
fi

# AUTOMATE: Log the attempt
echo "$(date): grep attempt on vault" >> ~/.openclaw/logs/vault-access.log
```

**Result**:
1. Agent sees helpful tip
2. Agent can choose to use skill or continue
3. Attempt is logged for analysis
4. No blocking, no friction

---

## Event Reference

See [Hook Events Reference](../reference/03-hook-events.md) for all available events.

---

## Key Takeaways

1. **"Automate + Inject"**: Hooks do silent work AND provide guidance
2. **Mode-switching resistance**: Hooks work around current LLM limitations
3. **Be specific**: Clear instructions beat vague warnings
4. **Be idempotent**: Safe to run multiple times
5. **Be fast**: Hooks must execute in milliseconds
6. **Don't block**: Agent autonomy is critical
7. **Context reinforcement**: Remind, don't force
