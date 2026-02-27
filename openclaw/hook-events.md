---
title: "Hook Events Reference"
summary: "Complete reference for all hook events in OpenClaw and Claude Code - session lifecycle, tool execution, and context injection."
read_when:
  - You need to implement specific hook events
  - You are choosing which hooks to use
  - You want to understand hook parameters
---

# Hook Events - Complete Reference

> ⚠️ **IMPORTANT**: For **CSH Framework (CLI + Skills + Hooks)**, focus on the **command hooks** pattern:
> - Build CLI tools with `--help` and `--json`
> - Use Skills for workflow guidance
> - Use Hooks for automation and reinforcement
>
> **MCP Server**: ❌ **NOT RELEVANT** - MCP is an alternative architecture (server-client model) that conflicts with CSH's "CLI + Skills + Hooks" model. CSH uses direct CLI execution, not remote tool servers.

---

## Table of Contents

- [Claude Code Hooks](#claude-code-hooks)
  - [Session Lifecycle Events](#session-lifecycle-events)
  - [Tool Events](#tool-events)
  - [Agent Events](#agent-events)
  - [User Interaction Events](#user-interaction-events)
  - [Worktree Events](#worktree-events)
  - [Notification Events](#notification-events)
- [OpenClaw Hooks](#openclaw-hooks)
- [Hook Design Patterns](#hook-design-patterns)
- [When to Use Which Hook](#when-to-use-which-hook)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Claude Code Hooks

Claude Code supports **17 hook events** for automation. Hooks fire at specific points in the session lifecycle and can control behavior, inject context, and enforce project rules.

### Quick Reference: All Events by Category

| Category | Events | Purpose |
|-----------|---------|----------|
| **Session Lifecycle** | `SessionStart`, `SessionEnd`, `PreCompact` | Load/unload context, manage session state |
| **Tool/Action** | `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` | Control tool execution, validate inputs, track usage |
| **Agent** | `SubagentStart`, `SubagentStop`, `Stop`, `TeammateIdle`, `TaskCompleted` | Manage subagents, control task completion |
| **User Interaction** | `UserPromptSubmit`, `Notification`, `ConfigChange` | Validate prompts, send notifications, audit changes |
| **Worktree** | `WorktreeCreate`, `WorktreeRemove` | Manage isolated worktrees |

---

## Session Lifecycle Events

### SessionStart

**Fires when**: Session begins (new) or resumes (continue)

**Can block?**: No

**Use for**:
- Loading development context (issues, TODOs, recent changes)
- Setting environment variables for the session
- Initializing project-specific state
- Re-injecting critical context after compaction

**Matcher values** (how session started):
| Matcher | When it fires |
|-----------|---------------|
| `startup` | New session started from scratch |
| `resume` | Session resumed with `--resume`, `--continue`, or `/resume` |
| `clear` | After `/clear` command |
| `compact` | After context compaction (manual or auto) |

**Input fields**:
```json
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../transcript.jsonl",
  "cwd": "/Users/.../myproject",
  "permission_mode": "default",
  "hook_event_name": "SessionStart",
  "source": "startup",  // startup, resume, clear, compact
  "model": "claude-sonnet-4-6",
  "agent_type": "Explore"  // optional, if started with --agent
}
```

**Decision control**:
- Any text written to stdout is added as context for Claude
- `additionalContext`: String added to Claude's context

**When to use**:
- ✅ **Every session start**: Load project conventions, recent work
- ✅ **After compaction**: Re-inject critical context that was lost
- ✅ **Resume only**: Load specific context when resuming previous work

**Example**:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Current sprint: auth refactor.'"
          }
        ]
      }
    ]
  }
}
```

---

### SessionEnd

**Fires when**: Session terminates (any reason)

**Can block?**: No

**Use for**:
- Cleanup tasks (remove temporary files)
- Logging session statistics
- Saving state for next session
- Cleanup worktrees if needed

**Matcher values** (why session ended):
| Matcher | When it fires |
|-----------|---------------|
| `clear` | User ran `/clear` |
| `logout` | Session ended normally |
| `prompt_input_exit` | User interrupted with Ctrl+C |
| `bypass_permissions_disabled` | Session exited due to bypass permissions |
| `other` | Any other reason |

**When to use**:
- ✅ **Cleanup on /clear**: Remove temporary files created during session
- ✅ **Always**: Log session statistics for analytics
- ✅ **Logout**: Save state for next session

**Example**:
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "matcher": "clear",
        "hooks": [
          {
            "type": "command",
            "command": "rm -f /tmp/claude-scratch-*.txt"
          }
        ]
      }
    ]
  }
}
```

---

### PreCompact

**Fires when**: Context window fills up and compaction is about to happen

**Can block?**: No

**Use for**:
- Preparing critical context before compression
- Summarizing important information that should survive
- Saving references to important documents

**Matcher values**:
| Matcher | When it fires |
|-----------|---------------|
| `manual` | User manually triggered compaction |
| `auto` | Auto-compaction triggered by context limit |

**When to use**:
- ✅ **Always**: Prepare context summary before compaction
- ✅ **Auto**: Highlight critical decisions that should be preserved

**Example**:
```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "auto",
        "hooks": [
          {
            "type": "command",
            "command": "echo '## Critical Context\\n- Current sprint: auth refactor\\n- Key decision: Use Bun runtime' > /tmp/compact-context.txt && cat /tmp/compact-context.txt"
          }
        ]
      }
    ]
  }
}
```

---

## Tool Events

### PreToolUse

**Fires when**: Claude creates tool parameters and BEFORE tool execution

**Can block?**: ✅ **YES** - Can allow, deny, or ask for permission

**Use for**:
- Validating tool inputs (prevent dangerous commands)
- Auto-approving safe operations
- Modifying tool inputs before execution
- Injecting additional context for specific tools

**Matcher values** (tool names):
| Matcher | Tools matched |
|-----------|--------------|
| `Bash` | Shell command execution |
| `Edit\|Write` | File editing tools |
| `Read` | File reading |
| `Glob` | File pattern matching |
| `Grep` | Content search |
| `WebFetch` | Web content fetching |
| `WebSearch` | Web search |
| `Task` | Subagent spawning |

**Input fields**:
```json
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../transcript.jsonl",
  "cwd": "/Users/.../myproject",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",  // or Edit, Write, Read, etc.
  "tool_input": {
    // Tool-specific fields
  },
  "tool_use_id": "tool_use_123"
}
```

**Tool-specific input schemas**:

#### Bash
```json
{
  "command": "npm test",
  "description": "Run test suite",
  "timeout": 120000,
  "run_in_background": false
}
```

#### Write
```json
{
  "file_path": "/path/to/file.txt",
  "content": "file content"
}
```

#### Edit
```json
{
  "file_path": "/path/to/file.txt",
  "old_string": "original text",
  "new_string": "replacement text",
  "replace_all": false
}
```

#### Read
```json
{
  "file_path": "/path/to/file.txt",
  "offset": 10,
  "limit": 50
}
```

#### Glob
```json
{
  "pattern": "**/*.ts",
  "path": "/path/to/dir"
}
```

#### Grep
```json
{
  "pattern": "TODO.*fix",
  "path": "/path/to/dir",
  "glob": "*.ts",
  "output_mode": "content",
  "i": true,
  "multiline": false
}
```

**Decision control** (via `hookSpecificOutput`):
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow" | "deny" | "ask",
    "permissionDecisionReason": "Reason for decision",
    "updatedInput": { /* Modified tool input */ }
  }
}
```

**Permission options**:
- `allow`: Proceed without showing permission prompt
- `deny`: Cancel tool call, send reason to Claude
- `ask`: Show permission prompt to user (default behavior)

**When to use**:
- ✅ **Security**: Block dangerous commands (`rm -rf`, `drop table`)
- ✅ **Validation**: Ensure tool inputs meet project rules
- ✅ **Auto-approve**: Allow safe operations without prompts
- ✅ **Modify inputs**: Add defaults or transform inputs

**Example: Block dangerous commands**
```bash
#!/bin/bash
# hooks/block-dangerous.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0
fi
```

---

### PostToolUse

**Fires when**: Tool call succeeds (after execution completes)

**Can block?**: No

**Use for**:
- Logging tool usage
- Post-processing (formatting after edits)
- Triggering tests after code changes
- Building artifacts after successful operations

**Matcher values**: Same as `PreToolUse` (tool names)

**Decision control** (via top-level `decision`):
```json
{
  "decision": "block",
  "reason": "Reason for blocking"
}
```

**When to use**:
- ✅ **After edits**: Run formatters (Prettier, ESLint)
- ✅ **After writes**: Build documentation
- ✅ **After Bash**: Log command execution

**Example: Auto-format after edits**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

---

### PostToolUseFailure

**Fires when**: Tool call fails

**Can block?**: No

**Use for**:
- Error logging
- Failure notification
- Recovery attempts
- Rolling back changes if possible

**Matcher values**: Same as `PreToolUse` (tool names)

**Decision control**: Same as `PostToolUse` (top-level `decision`)

**When to use**:
- ✅ **Error logging**: Track failed operations
- ✅ **Notification**: Alert on critical failures
- ✅ **Recovery**: Attempt automatic fixes

---

### PermissionRequest

**Fires when**: Claude shows permission dialog to user

**Can block?**: ✅ **YES** - Can auto-approve or auto-deny

**Use for**:
- Auto-approving safe operations
- Pre-denying dangerous operations
- Modifying tool inputs for approval
- Applying permission rules

**Matcher values**: Same as `PreToolUse` (tool names)

**Decision control** (via `hookSpecificOutput`):
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow" | "deny",
      "updatedInput": { /* Modified tool input */ }
    }
  }
}
```

**When to use**:
- ✅ **Auto-approve**: Allow read operations, safe scripts
- ✅ **Auto-deny**: Block writes to protected files
- ✅ **Modify inputs**: Add safety parameters

**Example: Auto-approve safe scripts**
```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "check-safe-script.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Agent Events

### SubagentStart

**Fires when**: Subagent is spawned

**Can block?**: No

**Use for**:
- Injecting context into subagent
- Configuring subagent behavior
- Initializing subagent state

**Matcher values** (agent types):
| Matcher | When it fires |
|-----------|---------------|
| `Bash` | Bash subagent |
| `Explore` | Explore subagent |
| `Plan` | Plan subagent |
| * | Any subagent (custom names) |

**When to use**:
- ✅ **Context injection**: Provide subagent with relevant context
- ✅ **Configuration**: Set subagent-specific settings

**Example: Inject context into subagent**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Specific context for subagent: focus on performance analysis.'"
          }
        ]
      }
    ]
  }
}
```

---

### SubagentStop

**Fires when**: Subagent finishes

**Can block?**: ✅ **YES** - Can prevent subagent from stopping

**Use for**:
- Validating subagent results
- Preventing premature termination
- Processing subagent output

**Matcher values**: Same as `SubagentStart`

**Decision control** (via top-level `decision`):
```json
{
  "decision": "block",
  "reason": "Subagent must complete all tasks"
}
```

**When to use**:
- ✅ **Verification**: Ensure subagent completed all tasks
- ✅ **Quality gate**: Require specific outputs before allowing stop

**Example: Block stop if no tests found**
```bash
#!/bin/bash
if ! grep -r "test" /tmp/subagent-output; then
  echo '{"decision": "block", "reason": "No tests found in subagent output"}'
fi
```

---

### Stop

**Fires when**: Main agent finishes responding (after each turn)

**Can block?**: ✅ **YES** - Can prevent Claude from stopping

**Use for**:
- Verifying task completion
- Enforcing quality gates
- Preventing premature termination

**Decision control** (via top-level `decision`):
```json
{
  "decision": "block",
  "reason": "All tasks must be complete before stopping"
}
```

**When to use**:
- ✅ **Quality gates**: Require test completion before stopping
- ✅ **Task verification**: Check that all work is done

**Example: Prevent stop if linting fails**
```bash
#!/bin/bash
if ! npm run lint; then
  echo '{"decision": "block", "reason": "Linting failed. Fix errors before stopping."}'
fi
```

---

### TeammateIdle

**Fires when**: Agent team teammate is about to go idle

**Can block?**: ✅ **YES** - Can prevent teammate from going idle

**Use for**:
- Quality checks before idle
- Additional work assignment
- Preventing incomplete task handoff

**Decision control**: Exit code 2 blocks (non-blocking error allows idle)

**When to use**:
- ✅ **Agent teams**: Ensure work quality before idle
- ✅ **Coordination**: Coordinate multiple agents

---

### TaskCompleted

**Fires when**: Task is being marked as complete

**Can block?**: ✅ **YES** - Can prevent task from being marked complete

**Use for**:
- Enforcing completion criteria
- Quality gates
- Validation checks

**Decision control**: Exit code 2 blocks completion

**When to use**:
- ✅ **Strict completion**: Require tests to pass
- ✅ **Validation**: Check all acceptance criteria

---

## User Interaction Events

### UserPromptSubmit

**Fires when**: User submits prompt, BEFORE Claude processes it

**Can block?**: ✅ **YES** - Can block prompt from being processed

**Use for**:
- Adding context based on prompt content
- Validating prompts (prevent certain types)
- Blocking unauthorized commands
- Preprocessing user input

**Input fields**:
```json
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../transcript.jsonl",
  "cwd": "/Users/.../myproject",
  "hook_event_name": "UserPromptSubmit",
  "prompt": "Write a function to calculate factorial of a number"
}
```

**Decision control**:
```json
{
  "decision": "block",
  "reason": "This type of request is not allowed",
  "additionalContext": "Additional context to inject"
}
```

**Adding context**:
- Plain text on stdout: Added as context
- JSON `additionalContext`: Added more discretely

**When to use**:
- ✅ **Context injection**: Add relevant context based on prompt
- ✅ **Validation**: Block dangerous requests
- ✅ **Preprocessing**: Transform or enhance user input

**Example: Inject context based on keyword**
```bash
#!/bin/bash
PROMPT=$(jq -r '.prompt' /dev/stdin)
if [[ "$PROMPT" == *"database"* ]]; then
  echo "Context: Database schema for this project is located in /docs/schema.sql"
fi
```

---

### Notification

**Fires when**: Claude Code sends notification (needs user input, permission, idle, etc.)

**Can block?**: No

**Use for**:
- Desktop notifications when Claude needs attention
- Custom notification routing (Slack, Discord, etc.)
- Notification logging

**Matcher values** (notification types):
| Matcher | When it fires |
|-----------|---------------|
| `permission_prompt` | Permission dialog shown |
| `idle_prompt` | Idle dialog shown |
| `auth_success` | Authentication successful |
| `elicitation_dialog` | Elicitation for preferences |

**When to use**:
- ✅ **Desktop notifications**: Alert when Claude needs input
- ✅ **Custom routing**: Forward notifications to team chat
- ✅ **Logging**: Track notification events

**Example: Desktop notification**
```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

---

### ConfigChange

**Fires when**: Configuration file changes during session

**Can block?**: ✅ **YES** - Can block change from taking effect

**Use for**:
- Auditing configuration changes
- Compliance logging
- Blocking unauthorized modifications
- Validating settings

**Matcher values** (configuration source):
| Matcher | When it fires |
|-----------|---------------|
| `user_settings` | `~/.claude/settings.json` changed |
| `project_settings` | `.claude/settings.json` changed |
| `local_settings` | `.claude/settings.local.json` changed |
| `policy_settings` | Managed policy changed |
| `skills` | Skills file changed |

**Decision control**: Exit code 2 blocks change

**When to use**:
- ✅ **Compliance**: Log all configuration changes
- ✅ **Security**: Block unauthorized modifications
- ✅ **Auditing**: Track who changed what and when

**Example: Block unauthorized skill modification**
```bash
#!/bin/bash
SOURCE=$(jq -r '.source' /dev/stdin)
if [[ "$SOURCE" == "skills" ]]; then
  echo "Unauthorized modification of skills blocked." >&2
  exit 2
fi
```

---

## Worktree Events

### WorktreeCreate

**Fires when**: Worktree is being created (via `--worktree` or `isolation: "worktree"`)

**Can block?**: ✅ **YES** - Non-zero exit causes creation to fail

**Use for**:
- Custom VCS integration
- Pre-populating worktree
- Setting up isolated environment

**When to use**:
- ✅ **VCS customization**: Override default git worktree behavior
- ✅ **Setup**: Pre-configure isolated environment

**Example: Pre-populate worktree with data**
```bash
#!/bin/bash
mkdir -p "$WORKTREE_PATH/data"
cp -r /shared/standard-data/* "$WORKTREE_PATH/data/"
```

---

### WorktreeRemove

**Fires when**: Worktree is being removed (session exit or subagent finish)

**Can block?**: No

**Use for**:
- Cleanup tasks
- Saving worktree state
- Post-processing worktree contents

**When to use**:
- ✅ **Cleanup**: Remove temporary files from worktree
- ✅ **Save state**: Preserve worktree contents before removal

**Example: Save worktree logs before removal**
```bash
#!/bin/bash
cp "$WORKTREE_PATH/session.log" /root/logs/worktree-$(date +%s).log
```

---

## OpenClaw Hooks (api.on - PROVEN WORKING)

> ⚠️ **REVISED (2026-02-26)**: This section documents hooks that are **PROVEN TO WORK** via `api.on()` in plugins. These are tested in production with ClawVault extension.

### Hook Types: Soft vs Hard

| Type | Capability | Use Case |
|------|------------|----------|
| **Soft Hook** | `prependContext`, `appendContext` | Inject instructions without blocking |
| **Hard Hook** | `block: true`, `result: {...}` | Modify params, block execution |

---

## PROVEN: Hooks That Actually Work

> **CRITICAL**: Only hooks listed in **official OpenClaw docs** (agent-loop.md) are guaranteed to work and be stable.

### Official Hooks (from OpenClaw docs - agent-loop.md)

These hooks are documented and guaranteed to work:

| Event | Soft Hook | Hard Hook | ClawVault Usage |
|-------|-----------|----------|----------------|
| `before_agent_start` | ✅ `prependContext` | ❌ | Inject context before run starts |
| `agent_end` | ✅ `prependContext` | ❌ | Log session end |
| `before_compaction` | ✅ `prependContext` | ❌ | Warn about memory limit |
| `after_compaction` | ✅ `prependContext` | ❌ | Re-inject context after compact |
| `before_tool_call` | ✅ `prependContext` | ⚠️ `{ block, result }` | Detect vault writes, suggest templates |
| `after_tool_call` | ✅ `prependContext` | ❌ | Suggest ClawVault after memory_search |
| `tool_result_persist` | ✅ `prependContext` | ❌ | Transform tool results before persist |
| `message_received` | ✅ `prependContext` | ❌ | Detect "remember this" intent |
| `message_sending` | ✅ `prependContext` | ❌ | (available) |
| `message_sent` | ✅ `prependContext` | ❌ | (available) |
| `session_start` | ✅ `prependContext` | ❌ | Inject skills menu at session start |
| `session_end` | ✅ `prependContext` | ❌ | Auto-checkpoint before exit |
| `gateway_start` | ✅ `prependContext` | ❌ | Gateway startup event |
| `gateway_stop` | ✅ `prependContext` | ❌ | Gateway shutdown event |

### Undocumented Hooks (Testing In Progress)

> ⚠️ These hooks are NOT in official docs. ClawVault adds **meaningful logging** to verify if they actually work:
> - Look for `✅ HOOK_NAME FIRED` in gateway logs
> - Proof requires: log appears + prependContext is applied

| Event | Test Status | Evidence Needed |
|-------|-------------|----------------|
| `before_prompt_build` | 🔍 Testing | Need agent run to verify |
| `llm_output` | 🔍 Testing | Need agent run to verify |
| `before_reset` | 🔍 Testing | Need /reset command to verify |
| `before_message_write` | 🔍 Testing | Need message to verify |
| `before_model_resolve` | ❓ Unknown | Not registered in ClawVault |
| `llm_input` | ❓ Unknown | Not registered in ClawVault |

---

## Hard Hook Examples (from ClawVault)

### before_tool_call - Block or Modify

```typescript
api.on("before_tool_call", async (event: any, ctx: any) => {
  // Hard hook: Block dangerous writes
  if (event.toolName === 'write') {
    const vaultPaths = ['/root/clawvault', '/mastervault'];
    if (vaultPaths.some(vp => event.params?.file_path?.includes(vp))) {
      return {
        prependContext: "\n\n[ClawVault: You're writing to vault. Use template!]",
        block: true  // Hard hook - blocks execution
      };
    }
  }
  // Soft hook - just adds context
  return { prependContext: "\n\n[Some instruction]" };
});
```

### before_tool_call - Modify Params

```typescript
api.on("before_tool_call", async (event: any, ctx: any) => {
  // Hard hook: Modify tool parameters
  if (event.toolName === 'Bash' && event.params?.command?.includes('npm')) {
    return {
      result: { command: event.params.command + ' --legacy-peer-deps' }
    };
  }
});
```

---

## Soft Hook Examples (from ClawVault)

### session_start - Inject Skills Menu

```typescript
api.on("session_start", async (event: any, ctx: any) => {
  return {
    prependContext: `
## 🐘 ClawVault Skills Available
- use skill 'memory-capture' - Remember decisions
- use skill 'memory-search' - Find past memories
- use skill 'session-checkpoint' - Save progress
`
  };
});
```

### message_received - Detect Intent

```typescript
api.on("message_received", async (event: any, ctx: any) => {
  const msg = event.message?.content?.toLowerCase() || '';
  const patterns = ['remember this', 'note this', 'save this'];

  if (patterns.some(p => msg.includes(p))) {
    return {
      prependContext: `[MANDATORY] User wants to remember! Use memory-capture skill.`
    };
  }
});
```

### llm_output - Detect Decisions

```typescript
api.on("llm_output", async (event: any, ctx: any) => {
  const content = event.content?.toLowerCase() || '';
  const decisionPatterns = ['i decided', 'we chose', 'best approach'];

  if (decisionPatterns.some(p => content.includes(p))) {
    return {
      prependContext: `[CRITICAL] Decision detected! YOU MUST use memory-capture skill.`
    };
  }
});
```

---

## Legacy HOOK.md Hooks (Still Work)

These use the HOOK.md format in plugin's `hooks/` directory:

| Event | Type | Description |
|-------|------|-------------|
| `gateway:startup` | Soft | Run at gateway start |
| `agent:bootstrap` | Soft | Before agent bootstraps |
| `command:new` | Soft | On `/new` command |
| `command:reset` | Soft | On `/reset` command |
| `command:stop` | Soft | On `/stop` command |

---

## Quick Reference: When to Use What

| Scenario | Hook | Type |
|----------|------|------|
| Inject skills at session start | `session_start` | Soft |
| Detect "remember this" in user message | `message_received` | Soft |
| Agent made a decision | `llm_output` | Soft |
| User about to write to vault | `before_tool_call` | Hard |
| Session ending - checkpoint | `session_end` | Soft |
| Context was compacted | `after_compaction` | Soft |

---

## Implementation in Plugin

```typescript
// extension-openclaw/src/index.ts
export default function initialize(api) {
  // Soft hooks - inject context
  api.on("session_start", async (event, ctx) => ({
    prependContext: "Welcome! Use memory-capture skill..."
  }));

  api.on("message_received", async (event, ctx) => {
    const msg = event.message?.content?.toLowerCase() || '';
    if (msg.includes('remember')) {
      return { prependContext: "User wants to remember! Use memory-capture skill." };
    }
  });

  // Hard hooks - can block
  api.on("before_tool_call", async (event, ctx) => {
    if (event.toolName === 'write' && isVaultPath(event.params?.file_path)) {
      return { block: true, prependContext: "Use template instead!" };
    }
  });
}
```

---

## What Does NOT Work (Tested)

| Event | Status | Notes |
|-------|--------|-------|
| `agent:error` | ❌ Not available | Tried, doesn't fire |
| `webhook` | ❌ Not available | Future feature |
| `cron` | ❌ Not available | Future feature |
| `heartbeat` | ❌ Not available | Separate system |

---

## Legacy: HOOK.md Format (Still Works)

### HOOK.md Format

```yaml
---
name: my-hook
description: "Does something useful"
metadata:
  openclaw:
    emoji: "🔧"
    events: ["session_start", "command:new"]
    requires:
      bins: ["my-cli"]
      env: ["API_KEY"]
---

# My Hook

This hook does something useful.
```

### Handler Format (handler.ts)

```typescript
const handler = async (event) => {
  // Gateway startup
  if (event.type === "gateway") {
    console.log("[my-hook] Gateway started");
    return;
  }

  // Command event
  if (event.type === "command" && event.action === "new") {
    // Run CLI to get context
    const { execFileSync } = require('child_process');
    const context = execFileSync('my-cli', ['context'], { encoding: 'utf8' });
    event.messages.push("Plugin context loaded");
  }
};

export default handler;
```

### When to use OpenClaw Hooks

| Event | Use Case | Example |
|--------|------------|----------|
| `session_start` | Load context, set environment | Load project conventions, API keys |
| `command:new` | Load previous session | Restore context from memory |
| `webhook` | Remote triggers | Start builds from CI/CD |
| `cron` | Scheduled automation | Daily backups, periodic reports |

---

## Hook Design Patterns

### 1. The "Skill Elicitation" Pattern (PRIMARY)
The primary value of CSH hooks is **elicitation**: ensuring the agent knows which skill to use for the current context.

**How it works**:
- Hook detects a specific state or event (e.g., `SessionStart` or `PreToolUse` for `Bash`).
- Hook injects a **Directive** for a skill: *"MANDATORY: Use the 'git-workflow' skill for all commits."*
- This overcomes the agent's tendency to use generic tools when specialized skills are available.

### 2. The "Pre-computation" Pattern
Perform expensive computation in the hook so the agent only sees the final result.

- Hook runs complex `grep | awk | jq` pipeline
- Hook returns summary string to agent
- Result: 200 tokens injected instead of 4,000 tokens of raw data

### 3. The "State Synchronization" Pattern
Keep the agent's internal context synced with external reality.

- `SessionStart`: Checks current git branch/status
- Injects: "You are on branch 'feature-auth'. There are 3 uncommitted changes."
- Benefit: Agent doesn't have to waste a tool call checking status.

### 4. The "Safety Rail" Pattern (Enforcement)
Intercept dangerous commands before they execute.

- `PreToolUse` for `Bash` with command `rm -rf /`
- Hook returns error message and blocks execution
- Result: 100% safety for critical paths

### 5. The "Redirection" Pattern (Circumstantial)
Redirect the agent from a generic native tool to a specialized CLI Primitive.

- Hook triggers on `PreToolUse` for `Grep` or `Glob`.
- Hook redirects input to your own CLI Primitive: `my-tool search --pattern "$QUERY"`.
- Benefit: Forces the agent to use your optimized discovery logic instead of expensive standard tools.

### 6. The "Synthesis" Pattern (Circumstantial)
Gather and unify context from multiple system sources in a single turn.

- Hook runs on `SessionStart`.
- Hook executes a sequence of checks: `git status`, `git log`, `ls -R`.
- Hook injects a unified summary: "Env is Ready. Current Branch: main. 3 modified files."
- Benefit: Instant situational awareness without requiring multiple tool calls from the agent.

---

## When to Use Which Hook

### Decision Tree

```
Need to...
├─ Load context at session start?
│  └─> SessionStart (Claude Code) or session_start (OpenClaw)
│
├─ Validate tool execution?
│  └─> PreToolUse (Claude Code)
│
├─ Process tool result?
│  └─> PostToolUse (Claude Code)
│
├─ Control when session stops?
│  └─> Stop (Claude Code)
│
├─ Need notifications?
│  ├─> Notification (Claude Code)
│  └─> session_recover (OpenClaw crash recovery)
│
├─ Manage subagents?
│  ├─> SubagentStart (inject context)
│  ├─> SubagentStop (validate results)
│  └─> Stop (main agent)
│
├─ Enforce project rules?
│  └─> ConfigChange (audit settings)
│
└─ External triggers?
   └─> webhook/cron (OpenClaw)
```

---

## Best Practices

### 1. Keep Hooks Fast

Hooks run in the session hot path. Keep them under 100ms for user-facing hooks.

**Bad**: Slow PostToolUse hook
```bash
#!/bin/bash
# Takes 5 seconds - blocks Claude
npm run full-test-suite
```

**Good**: Quick validation
```bash
#!/bin/bash
# Takes 10ms
if grep -q "danger" <<< "$1"; then
  exit 2
fi
exit 0
```

---

### 2. Make Hooks Idempotent

Hooks should handle being called multiple times safely.

**Bad**: Creates duplicates
```bash
echo "export VAR=value" >> .env  # Adds on every call
```

**Good**: Checks before adding
```bash
if ! grep -q "VAR=value" .env; then
  echo "export VAR=value" >> .env
fi
```

---

### 3. Use Exit Codes Correctly

- **Exit 0**: Allow action (success)
- **Exit 2**: Block action (error)
- **Other**: Non-blocking error

**Pattern**:
```bash
#!/bin/bash
INPUT=$(cat)

# Validate
if [[ "$INPUT" == *"danger"* ]]; then
  echo "Blocked: dangerous operation" >&2
  exit 2
fi

# Success
exit 0
```

---

### 4. JSON Output for Decisions

For structured control, use JSON instead of exit codes.

**Exit code approach**:
```bash
exit 2  # Block with stderr message
```

**JSON approach** (more control):
```bash
jq -n '{
  hookSpecificOutput: {
    hookEventName: "PreToolUse",
    permissionDecision: "deny",
    permissionDecisionReason: "Specific reason"
  }
}'
```

---

### 5. Test Hooks in Isolation

Test hooks with sample input before deploying.

```bash
# Test with sample input
echo '{"tool_name":"Bash","tool_input":{"command":"rm -rf"}}' | ./my-hook.sh
echo $?  # Check exit code
```

---

## Troubleshooting

### Hook Not Firing

**Symptom**: Hook configured but never executes

**Diagnosis**:
1. Check hook event name (case-sensitive)
2. Verify matcher pattern (regex)
3. Confirm hook location (user vs. project vs. plugin)
4. Check `disableAllHooks` setting

**Solution**:
```bash
# List all hooks
openclaw hooks list  # OpenClaw
# or
cat ~/.claude/settings.json | jq '.hooks'

# Check hook event name
# Use exact event name: "PreToolUse", not "preToolUse"
```

---

### JSON Parsing Error

**Symptom**: "JSON validation failed" in logs

**Causes**:
- Shell profile prints output on startup
- Trailing commas in JSON
- Comments in JSON (not allowed)

**Solution**:
```bash
# Wrap shell profile output
if [[ $- == *i* ]]; then
  echo "Interactive shell"
fi

# Validate JSON
jq '.' < my-hook.json
```

---

### Hook Runs Forever

**Symptom**: Infinite loop, Claude keeps working

**Cause**: Stop hook doesn't check `stop_hook_active`

**Solution**:
```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Allow stop
fi
# Hook logic
```

---

## MCP Server Clarification

> ⚠️ **NOT RELEVANT FOR CSH FRAMEWORK**

**What is MCP?**
- Model Context Protocol (MCP) is a **server-client architecture** for connecting AI agents to external tools
- MCP servers expose tools via HTTP/SSE transport
- Tools appear as `mcp__<server>__<tool>` in Claude Code

**Why NOT use MCP for CSH?**

| Aspect | CSH Framework (CLI + Skills + Hooks) | MCP Server |
|--------|-----------------------------------------|-------------|
| **Architecture** | Direct CLI execution | Remote server-client |
| **Token Efficiency** | 100% (no protocol overhead) | 87.5% (JSON protocol overhead) |
| **Control** | Full control over CLI tools | Limited by server capabilities |
| **Deployment** | Simple npm install | Requires running server |
| **Complexity** | Low (shell script + SKILL.md) | High (server + protocol) |
| **Portability** | Works everywhere | Depends on network |

**CSH Recommendation**:
- ✅ **Use CLI + Skills + Hooks**: Direct execution, full control, minimal complexity
- ❌ **Avoid MCP**: Unnecessary protocol layer, less control, more complexity

**When to use MCP**:
- External tool provider (third-party service)
- Multi-user shared tool access
- When CSH architecture doesn't apply

**CSH Framework does NOT use MCP** - It's an alternative architecture, not a part of the framework.

---

## Deprecated Features (Claude Code)

> ⚠️ **DEPRECATED** - Do not use these in new projects

### HTTP+SSE Transport (MCP)

**Status**: ⚠️ **DEPRECATED** (2024-11-05)

**What was deprecated**:
- HTTP+SSE transport for MCP servers
- Replaced by newer transport mechanism

**What to use instead**:
- Use modern MCP transport (WebSocket or HTTP/2)
- Or better: Use CSH Framework (CLI + Skills + Hooks) - no MCP needed

---

## Related Docs

- [Plugin Development Guide](../guides/02-plugin-development.md)
- [Skill Format Reference](../reference/02-skill-format.md)
- [Hooks Patterns Guide](../patterns/03-hooks.md)
- [Platform Comparison](../platforms/03-comparison.md)
