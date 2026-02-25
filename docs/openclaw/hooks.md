# OpenClaw Hooks

Event-driven automation and context injection system.

> ⚠️ **MAJOR UPDATE (2026.2.24+)**: The hook system has been FIXED! Both hook systems now work correctly. See sections below for complete documentation.

---

## Two Hook Systems

OpenClaw has **TWO separate hook systems**:

| System | Registration | Use For |
|--------|-------------|---------|
| **Internal Hooks** (HOOK.md) | Static, file-based | Gateway events, automation |
| **Plugin Lifecycle Hooks** (api.on()) | Dynamic, in plugin code | Agent lifecycle, tool interception |

Both systems NOW WORK as of OpenClaw 2026.2.24+.

---

## System 1: Plugin Lifecycle Hooks (api.on())

Dynamic hooks registered inside plugin code using `api.on()`. These intercept the agent's internal loop.

### Available Hooks

#### Agent Lifecycle Hooks

| Hook | When | Use For |
|------|------|---------|
| `before_model_resolve` | Pre-session, before model resolution | Override provider/model |
| `before_prompt_build` | After session load, before prompt submission | Inject context, prependContext |
| `before_agent_start` | Legacy compatibility | Either phase |
| `agent_end` | After completion | Inspect final messages, metadata |
| `before_reset` | Before session reset | Cleanup |

#### LLM Hooks

| Hook | When | Use For |
|------|------|---------|
| `llm_input` | Before LLM input submission | Observe/modify input |
| `llm_output` | After LLM response | Observe output |

#### Compaction Hooks

| Hook | When | Use For |
|------|------|---------|
| `before_compaction` | Before compaction cycles | Observe/annotate |
| `after_compaction` | After compaction cycles | Observe/annotate |

#### Tool Hooks

| Hook | When | Use For |
|------|------|---------|
| `before_tool_call` | Before tool execution | Intercept/modify params, block calls |
| `after_tool_call` | After tool execution | Observe results |
| `tool_result_persist` | Before writing to transcript | Transform results |

#### Message Hooks

| Hook | When | Use For |
|------|------|---------|
| `message_received` | Inbound message received | Process input |
| `message_sending` | Outbound message (can modify) | Modify before send |
| `message_sent` | Outbound sent successfully | Confirm delivery |
| `before_message_write` | Before writing to session | Modify session |

#### Session Lifecycle Hooks

| Hook | When | Use For |
|------|------|---------|
| `session_start` | Session begins | Initialize state |
| `session_end` | Session ends | Cleanup, checkpoint |

#### Gateway Lifecycle Hooks

| Hook | When | Use For |
|------|------|---------|
| `gateway_start` | Gateway starts (after channels) | Startup tasks |
| `gateway_stop` | Gateway stops | Shutdown cleanup |

### Registration Pattern

```typescript
// In plugin src/index.ts
export default function register(api: PluginAPI) {
  // Soft injection - inject context before prompt
  api.on('before_prompt_build', async (event, ctx) => {
    return {
      prependContext: '## Custom Context\nUse my-plugin for...'
    };
  });

  // Tool interception - modify tool results
  api.on('after_tool_call', async (event, ctx) => {
    console.log(`Tool ${event.toolName} returned:`, event.result);
    return { result: event.result }; // Pass through
  });

  // Block dangerous tools
  api.on('before_tool_call', async (event, ctx) => {
    if (event.toolName === 'exec' && event.params.command.includes('rm -rf')) {
      return { block: true, result: 'Blocked: dangerous command' };
    }
  });
}
```

### Return Values

| Hook | Return | Effect |
|------|--------|--------|
| `before_prompt_build` | `{ prependContext, systemPrompt }` | Inject into prompt |
| `before_tool_call` | `{ block, result }` | Block or modify params |
| `after_tool_call` | `{ result }` | Modify tool result |
| `tool_result_persist` | `{ result }` | Transform before storage |
| Other hooks | `{}` or nothing | No modification |

---

## System 2: Internal Hooks (HOOK.md)

---

## Hook Configuration

OpenClaw hooks are configured via two components:
- **Instructions**: `HOOK.md` file containing guidance
- **Logic**: `handler.ts` (or shell script) containing executable code

### Hook Directory Structure

```
my-plugin/
├── openclaw.plugin.json
└── hooks/
    └── my-hook/
        ├── HOOK.md          # Instructions + metadata
        └── handler.ts       # Logic
```

### Hook Registration

Hooks are registered in the plugin manifest OR auto-discovered from HOOK.md:

```json
{
  "id": "my-plugin",
  "hooks": [
    {
      "event": "command:new",
      "handler": "./hooks/my-hook/handler.ts",
      "description": "Load plugin context"
    }
  ]
}
```

**Note**: Modern plugins can also auto-discover hooks from the `hooks/` directory by including HOOK.md with proper metadata.

---

## Hook Discovery Precedence

Hooks are scanned with this priority:

1. **Workspace hooks**: Hooks in workspace `hooks/` directory
2. **Managed hooks**: Hooks installed via plugin system
3. **Bundled hooks**: Hooks distributed with gateway

**Priority**: Later entries override earlier ones (if conflicts exist).

---

## Currently Available Events

> ⚠️ **IMPORTANT**: The following events are CONFIRMED available as of the latest OpenClaw documentation. Events marked as "Future" are NOT yet implemented.

### Command Events

| Event | Description | Status |
|-------|-------------|--------|
| `command` | All command events (general listener) | ✅ Available |
| `command:new` | When `/new` command is issued | ✅ Available |
| `command:reset` | When `/reset` command is issued | ✅ Available |
| `command:stop` | When `/stop` command is issued | ✅ Available |

### Agent Events

| Event | Description | Status |
|-------|-------------|--------|
| `agent:bootstrap` | Before workspace bootstrap files are injected | ✅ Available |

### Gateway Events

| Event | Description | Status |
|-------|-------------|--------|
| `gateway:startup` | After channels start and hooks are loaded | ✅ Available |

---

## Events via api.on() (NOW AVAILABLE)

> ✅ **UPDATED (2026.2.24+)**: These events are now available via `api.on()` in plugins!

| Event | System | Description |
|-------|--------|-------------|
| `session_start` | api.on() | Session begins |
| `session_end` | api.on() | Session ends |
| `message_received` | api.on() | Message received |
| `message_sent` | api.on() | Message sent |
| `before_compaction` | api.on() | Before compaction |
| `after_compaction` | api.on() | After compaction |

Use `api.on()` inside plugins for these lifecycle events.

---

## Supported Hook Events (Legacy)

> ⚠️ **DEPRECATED**: This section contains outdated event information. See "Currently Available Events" and "Future Events" sections above for up-to-date documentation.

For event details, see the official OpenClaw documentation: https://docs.openclaw.ai/automation/hooks

---

## Hook Handler Pattern

### TypeScript Handler

```typescript
// hooks/my-hook/handler.ts
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Check event type and action
  if (event.type === "command" && event.action === "new") {
    // Event: command:new - /new command issued
    console.log(`[my-hook] New command triggered`);
    console.log(`  Session: ${event.sessionKey}`);

    // Inject message to user
    event.messages.push("✨ My hook executed!");
    return;
  }

  if (event.type === "command" && event.action === "reset") {
    // Event: command:reset - /reset command issued
    console.log("[my-hook] Reset command triggered");
    return;
  }

  if (event.type === "command" && event.action === "stop") {
    // Event: command:stop - /stop command issued
    console.log("[my-hook] Stop command triggered");
    return;
  }

  if (event.type === "agent" && event.action === "bootstrap") {
    // Event: agent:bootstrap - Before workspace bootstrap
    console.log("[my-hook] Agent bootstrap");
    // Can modify context.bootstrapFiles here
    return;
  }

  if (event.type === "gateway" && event.action === "startup") {
    // Event: gateway:startup - Gateway started
    console.log("[my-hook] Gateway started");
    return;
  }
};

export default myHandler;
```

### Event Context Structure

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

### CLI-Based Handler (Alternative)

For simple scripts, you can also create a CLI-based handler:

```bash
#!/bin/bash
# hooks/my-hook/handler.sh

# Event passed as first argument
EVENT=$1

if [ "$EVENT" = "command:new" ]; then
  echo "Processing /new command"
  # Run your logic here
fi
```

> ⚠️ **NOTE**: Legacy handlers may receive events differently. For production plugins, prefer the TypeScript handler approach above.
  // Perform cleanup, reindexing, etc.
  console.log("Performing maintenance...");
}
```

### Shell Script Handler

```bash
#!/bin/bash
# hooks/my-hook/handler.sh

# Read event from stdin (JSON)
read -r event
type=$(echo "$event" | jq -r '.type')
payload=$(echo "$event" | jq -r '.payload')

# Event: session_start
if [[ "$type" == "session_start" ]]; then
  # Load context
  echo "✅ Plugin loaded"
  echo "Available: my-skill, another-skill"
fi

# Event: command:new
if [[ "$type" == "command:new" ]]; then
  command=$(echo "$payload" | jq -r '.command')

  # Inject context
  if [[ "$command" == *"search"* ]]; then
    echo "💡 TIP: Use 'my-skill' for better results."
  fi
fi
```

---

## Hook Instructions (HOOK.md)

The `HOOK.md` file provides metadata for hook auto-discovery:

```markdown
---
name: my-hook
description: "Does something useful"
metadata:
  { "openclaw": { "emoji": "🔧", "events": ["command:new", "command:reset"], "requires": { "bins": ["my-cli"] } } }
---

# My Hook

This hook does something useful when you issue `/new`.

## Behavior

- **command:new**: Loads context when /new is issued
- **command:reset**: Cleanup before reset

## Testing

Test hook firing by checking logs:
```bash
openclaw logs --follow --tail 50 | grep "Registered hook"
```
```

### Metadata Fields

The `metadata.openclaw` object supports:

- **`emoji`**: Display emoji for CLI (e.g., `"🔧"`)
- **`events`**: Array of events to listen for (e.g., `["command:new", "command:reset"]`)
- **`requires`**: Optional requirements
  - **`bins`**: Required binaries on PATH (e.g., `["git", "node"]`)
  - **`anyBins`**: At least one of these binaries must be present
  - **`env`**: Required environment variables
  - **`config`**: Required config paths (e.g., `["workspace.dir"]`)
  - **`os`**: Required platforms (e.g., `["darwin", "linux"]`)
- **`always`**: Bypass eligibility checks (boolean)

---

## Hook Types: Automation vs. Injection

### Automation Hooks

Perform side effects without returning text:

```typescript
// Automation hook (no return value)
export default async function handler(event: HookEvent): Promise<void> {
  if (event.type === "command" && event.action === "stop") {
    // Save session data on /stop
    await saveSessionData(event.context);
  }
}
```

**Use Cases**:
- Saving checkpoints on command:stop
- Cleaning up temporary files
- Logging analytics
- Reindexing databases

### Injection Hooks

Return text to inject into agent context:

```typescript
// Injection hook (returns text)
export default async function handler(event: HookEvent): Promise<string> {
  if (event.type === "command" && event.action === "new") {
    // Inject context when /new is issued
    event.messages.push("✅ Plugin context loaded");
    return "";
  }
}
```

**Use Cases**:
- Reminding about available skills
- Warning about constraints
- Providing context
- Suggesting better approaches

---

## Hook Best Practices

### 1. Keep Hooks Lightweight

```typescript
// Good: Fast, minimal work
export default async function handler(event: HookEvent): Promise<string> {
  if (event.type === "session_start") {
    return "✅ Ready";
  }
}

// Bad: Heavy computation in hook
export default async function handler(event: HookEvent): Promise<string> {
  if (event.type === "session_start") {
    await heavyComputation(); // 30 seconds!
    return "✅ Ready";
  }
}
```

### 2. Use Hooks for Guidance, Not Implementation

```typescript
// Good: Guidance
export default async function handler(event: HookEvent): Promise<string> {
  return "Use 'my-skill' for searching files.";
}

// Bad: Implementation
export default async function handler(event: HookEvent): Promise<string> {
  // Don't implement the skill in the hook!
  const results = await searchFiles(event.payload.query);
  return JSON.stringify(results);
}
```

### 3. Handle Errors Gracefully

```typescript
export default async function handler(event: HookEvent): Promise<string> {
  try {
    if (event.type === "session_start") {
      return "✅ Ready";
    }
  } catch (error) {
    console.error("Hook error:", error);
    // Don't crash, return gracefully
    return "⚠️ Hook encountered an error";
  }
}
```

---

## Cron Scheduling

Hooks can be scheduled using cron expressions:

```json
{
  "id": "my-plugin",
  "hooks": [
    {
      "event": "cron",
      "schedule": "0 0 * * *", // Daily at midnight
      "handler": "./hooks/daily-cleanup/handler.ts"
    }
  ]
}
```

**Cron Expression Format**:
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6)
│ │ │ │ │
* * * * *
```

**Examples**:
- `0 0 * * *`: Daily at midnight
- `0 */6 * * *`: Every 6 hours
- `0 9 * * 1-5`: 9 AM on weekdays
- `*/30 * * * *`: Every 30 minutes

---

## IMPORTANT: Plugin Hooks vs Workspace Hooks

> ✅ **UPDATED (2026.2.24+)**: Both hook systems now work correctly!

### Workspace Hooks (Internal Hooks)
- Location: `~/.openclaw/hooks/` or `<workspace>/hooks/`
- Loaded via: `hooks:loader` subsystem
- Events: `command:new`, `agent:bootstrap`, `gateway:startup`
- Best for: Gateway automation, checkpointing

### Plugin Lifecycle Hooks (api.on())
- Location: Inside plugin's `src/index.ts`
- Registered via: `api.on()` in plugin code
- Events: 18+ hooks (before_prompt_build, after_tool_call, etc.)
- Best for: Soft injection, tool interception, agent lifecycle

### Plug-and-Play: Bundle Everything in Plugin

For true plug-and-play, use plugin lifecycle hooks (api.on()) inside the plugin:

```typescript
// In plugin: self-contained, no workspace config needed
api.on('before_prompt_build', (event) => ({
  prependContext: 'Use my-skill for this task...'
}));
```

This way users just install the plugin and everything works - no manual hook configuration.

---

## Legacy Capacities

> ⚠️ **IMPORTANT**: OpenClaw supports legacy hook configurations for backwards compatibility. However, new plugins should use the modern HOOK.md + handler.ts approach.

### Legacy Config Format

The old config format in `openclaw.json` is still supported:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Note**: `module` must be a workspace-relative path. Absolute paths and traversal outside the workspace are rejected.

### Migration

To migrate from legacy to modern hooks:

1. Create hook directory: `mkdir -p ~/.openclaw/hooks/my-hook`
2. Move handler: `mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts`
3. Create HOOK.md with proper metadata
4. Remove legacy config from openclaw.json

---

## Troubleshooting

### Hook Not Firing

**Symptom**: Hook configured but never executes

**Check**:
```bash
# List registered hooks
openclaw hooks list

# Check hook event name (case-sensitive)
# Correct: "command:new", "agent:bootstrap", "gateway:startup"
# Deprecated: "session_start" (not available yet)

# Check logs
openclaw logs --follow --tail 50 | grep -i hook
```

### Hook Throws Error

**Symptom**: Hook executes but throws error

**Check**:
```bash
# Check logs for error
openclaw logs --follow --tail 50 | grep -i error

# Test hook manually
node -e "$(cat hooks/my-hook/handler.ts)"

# Check handler exports
cat hooks/my-hook/handler.ts | grep "export default"
```

### Hook Not Found

**Symptom**: Plugin loaded but hook not registered

**Check**:
```bash
# Verify handler path
ls -la hooks/my-hook/handler.ts

# Check manifest registration
cat openclaw.plugin.json | jq '.hooks'

# Verify file permissions
chmod +x hooks/my-hook/handler.ts
```

---

## References

- **Hook Events**: [Complete Hook Events Reference](./hook-events.md)
- **Hook Patterns**: [Hook Design Patterns](../core/hooks.md)
- **Best Practices**: [Testing Validation Best Practices](../core/best-practices.md)
- **Skill System**: [OpenClaw Skills](./skills.md)

---

*See Also*: [OpenClaw Overview](./01-overview.md) for general platform information
