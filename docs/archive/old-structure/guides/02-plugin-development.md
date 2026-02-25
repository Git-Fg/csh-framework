# Plugin Development Guide

> ⚠️ **CSH FRAMEWORK CONTEXT**: For **CSH Framework (CLI + Skills + Hooks)**, plugins are packaging for:
> - CLI tools with agent-usable `--help` and `--json` interfaces
> - SKILL.md files for workflow guidance
> - Hooks for automation and reinforcement
>
> **NOT relevant**: MCP server integration (alternative architecture)
> **Deprecated**: HTTP+SSE transport (use modern MCP or skip entirely)

---

## Table of Contents

- [What is a Plugin?](#what-is-a-plugin)
- [Architecture Overview](#architecture-overview)
- [When to Use Which Platform](#when-to-use-which-platform)
- [Development Workflow](#development-workflow)
- [Plugin Manifests](#plugin-manifests)
- [Building CLI Tools](#building-cli-tools)
- [Writing Skills](#writing-skills)
- [Adding Hooks](#adding-hooks)
- [OpenClaw Plugin API](#openclaw-plugin-api)
- [Best Practices](#best-practices)
- [Testing](#testing)
- [Publishing](#publishing)
- [Common Patterns](#common-patterns)
- [Not Relevant for CSH](#not-relevant-for-csh)
- [Deprecated Features](#deprecated-features)
- [Troubleshooting](#troubleshooting)

---

## What is a Plugin?

A plugin bundles:
- **Skills** — Workflow definitions (SKILL.md files)
- **Hooks** — Automation handlers (command hooks)
- **Configuration** — Manifest and metadata

**Plugin Purpose in CSH**:
- Package CLI tools for AI agents
- Provide skills with workflow guidance
- Add hooks for automation and reinforcement
- Enable distribution via npm or git

---

## Core Content Standards

To maintain **portability** across platforms (Claude Code, OpenClaw, and future hosts), CSH plugins distinguish between standard and platform-specific contents.

### CSH Standard (Portable)
These directories and files follow a cross-platform specification.
- `skills/` — Namespaced skill directories containing `SKILL.md` files.
- `hooks/` — Automation scripts or definitions (standard JSON/Bash).
- `package.json` — Standard npm manifest with `csh` metadata.
- `LICENSE`, `README.md`, `CHANGELOG.md` — Project documentation.

### Platform-Specific (Avoid for Generic Plugins)
These directories are internal to specific hosts and should not be used for logic intended to be portable.
- `agents/` — **OpenClaw Internal**. Defines OpenClaw-specific agent personas and parameters. Logic here won't work in Claude Code.
- `commands/` — **OpenClaw Internal**. Defines custom gateway CLI commands.
- `discovery/` — **OpenClaw Internal**. Custom plugin discovery logic.
- `.claude-plugin/` — **Claude Code Internal**. Platform-specific overrides.

**Rule of Thumb**: If it doesn't live in `skills/` or `hooks/`, it is likely platform-specific. Keep your intelligence (Skills) and orchestration (Hooks) in the standard folders.

### Core Differences Between Platforms

| Aspect | Claude Code | OpenClaw |
|--------|-----------|-----------|
| **Nature** | CLI Agent with Plugin System | Multi-Channel Gateway with Plugins |
| **Manifest** | `.claude-plugin/plugin.json` | `openclaw.plugin.json` (or root exports) |
| **Installation** | `/plugin install <spec>` | `openclaw plugins install <spec>` |
| **Configuration** | Project `settings.json` or plugin `settings.json` | `~/.openclaw/openclaw.json` (`plugins.entries.*`) |
| **Skills Location** | `~/.claude/skills/`, `.claude/skills/`, `<plugin>/skills/` | `~/.openclaw/skills/`, `<workspace>/skills/`, `<plugin>/skills/` |
| **Skill Discovery** | Auto-scan on startup | Via plugin manifest + filtering |
| **Skill Execution** | Agent can always execute skills directly | Agent can execute skills or use CLI commands |
| **Skill Control** | Agent has high freedom | Skills can be explicitly enabled/disabled |
| **Hooks Format** | `hooks/hooks.json` (JSON structure) | `HOOK.md` + `handler.ts` (Event-driven) |
| **Hook Events** | 17 events (SessionStart, PreToolUse, etc.) | 8+ events (session_start, webhook, cron, etc.) |
| **Hook Scope** | Project-level hooks, limited events | Session, Command, Gateway, Custom |
| **Namespacing** | Namespaced: `/plugin:skill` | Global (precedence: Workspace > Managed > Bundled) |
| **State** | In-memory/Session-bound | Persistent (Gateway-owned) |
| **Session Persistence** | No native recovery | Survives restarts via JSONL transcript storage |

### Skill Discovery and Loading

**Claude Code**:
- Skills automatically discovered from standard directories
- Namespaced: Skills in plugins are `/plugin:skill`
- No gating mechanism: All skills load by default
- Skills tied to Claude project configuration

**OpenClaw**:
- Skills discovered via plugin manifest
- Dynamic filtering: Skills filtered by `requires.bins`, `requires.env`, `requires.os`
- Discovery order: Workspace → Managed → Bundled (later overrides earlier)
- Skill gating: Only eligible skills appear in agent context
- Progressive disclosure: Metadata loaded first, full instructions on use

### Hook Systems Comparison

**Claude Code Hooks** (17 events):
- **Session Lifecycle**: `SessionStart`, `SessionEnd`, `PreCompact`
- **Tool/Action**: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`
- **Agent**: `SubagentStart`, `SubagentStop`, `Stop`, `TeammateIdle`, `TaskCompleted`
- **User Interaction**: `UserPromptSubmit`, `Notification`, `ConfigChange`
- **Worktree**: `WorktreeCreate`, `WorktreeRemove`

**OpenClaw Hooks** (8+ events):
- **Session**: `session_start`, `session_end`, `session_recover`
- **Command**: `command:new`, `command:reset`, `command:stop`
- **Gateway**: `gateway:startup`, `webhook`, `cron`, `heartbeat`

**See**: [Hook Events Enhanced Reference](../reference/03-hook-events-enhanced.md) for complete documentation.

---

## When to Use Which Platform

### Decision Framework

```
Need...
├─> Claude Desktop only?
│  ├─> ✅ Use Claude Code
│  └─> ❌ OpenClaw not needed
│
├─> Multi-platform (Desktop + Mobile + Telegram)?
│  ├─> ✅ Use OpenClaw
│  └─> ✅ Claude Code possible with OpenClaw extension
│
├─> ClawVault integration?
│  ├─> ✅ Use OpenClaw
│  └─> ❌ Claude Code not supported
│
├─> Maximum agent autonomy?
│  ├─> ✅ Use OpenClaw
│  └─> ✅ Hybrid approach (both)
│
└─> Simplest setup?
   ├─> ✅ Claude Code (no gateway)
   └─> ❌ OpenClaw (requires gateway)
```

### Choose Claude Code When

- ✅ Working exclusively within Claude ecosystem
- ✅ Desktop-only workflow
- ✅ Tighter integration with Claude Desktop
- ✅ Simpler setup (no gateway needed)
- ✅ Project-local hooks (easier to manage)

### Choose OpenClaw When

- ✅ Multi-platform support (Desktop, Mobile, Telegram, etc.)
- ✅ Universal access (multiple agent platforms)
- ✅ ClawVault integration
- ✅ Maximum agent autonomy
- ✅ Persistent memory across sessions
- ✅ Advanced gateway features (webhooks, cron, channels)

### Use Both Together (Hybrid)

✅ **Possible**: Claude Desktop with OpenClaw extension installed

**What happens**:
- OpenClaw skills load into agent context
- OpenClaw hooks inject prompts for operations
- Both systems share the same Claude agent

**Considerations**:
- Skill naming conflicts (use prefixes: `claude-*` vs `clawvault-*`)
- Hook execution order (undefined, may cause conflicts)
- Memory access coordination (don't double-create)
- Token efficiency (both systems add overhead)

---

## Development Workflow

### 1. Create Plugin Structure

```bash
# For Claude Code
mkdir -p my-plugin/hooks
mkdir -p my-plugin/skills/my-skill

# For OpenClaw
mkdir -p my-plugin/hooks/my-hook
mkdir -p my-plugin/skills/my-skill
touch my-plugin/SKILL.md
```

### 2. Define Manifest

Choose your target platform(s) and create appropriate manifest(s).

### 3. Build CLI Tool

**For CSH Framework**: Build agent-usable CLI tool

**Key Requirements**:
- `--help` flag (self-documenting)
- `--json` output (machine-readable)
- Exit codes (0=success, 1=error, 2=usage)
- Verbose mode (`--verbose`)
- Discovery commands (`--types`, `--list`)

**Example Commander.js CLI**:
```typescript
#!/usr/bin/env node
import { Command } from 'commander';

const program = new Command();
program
  .name('my-tool')
  .description('My CLI tool for AI agents')
  .option('--json', 'JSON output')
  .option('--verbose', 'Verbose logging')
  .option('--types', 'List available types')
  .action(async (options) => {
    // Discovery mode
    if (options.types) {
      console.log(JSON.stringify({
        types: ['search', 'create', 'update'],
        descriptions: {
          search: 'Search for items',
          create: 'Create new item',
          update: 'Update existing item'
        }
      }, null, 2));
      process.exit(0);
    }

    // Main operation
    try {
      const result = await performOperation(options);
      if (options.json) {
        console.log(JSON.stringify(result, null, 2));
      } else {
        console.log(formatHumanReadable(result));
      }
    } catch (error) {
      if (options.json) {
        console.error(JSON.stringify({ error: error.message }, null, 2));
      } else {
        console.error(`Error: ${error.message}`);
      }
      process.exit(1);
    }
  });

program.parse(process.argv);
```

See [Agent-Usable CLI Tools](../patterns/agent-usable-cli-tools.md) for comprehensive guide (400 lines).

### 4. Write Skills

Create `SKILL.md` following the [format specification](../reference/02-skill-format.md).

```yaml
---
name: my-skill
description: Guide for using my-tool CLI
requires:
  bins: ["my-tool"]
---

# My Skill

## Search Operation

Use `my-tool search` to find items:

```bash
my-tool search "<query>"
```

## Create Operation

Use `my-tool create` to create new items:

```bash
my-tool create --title "Title" --description "Description"
```

## JSON Output

All operations support `--json` flag for programmatic access:

```bash
my-tool search --json "query"
```

Returns array of results.
```

### 5. Add Hooks (Optional)

#### Claude Code Hooks

Create `hooks/hooks.json`:

```json
{
  "description": "My plugin hooks",
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/init.sh"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

#### OpenClaw Hooks

Create `hooks/my-hook/HOOK.md`:

```yaml
---
name: my-hook
description: "Initialize my-tool context"
metadata:
  openclaw:
    emoji: "🔧"
    events: ["session_start", "command:new"]
    requires:
      bins: ["my-tool"]
---

# My Hook

This hook initializes context for my-tool operations.
```

Create `hooks/my-hook/handler.ts`:

```typescript
const handler = async (event) => {
  const { execFileSync } = require('child_process');

  // Session start
  if (event.type === "session") {
    console.log("[my-hook] Session started");
    return;
  }

  // Command event
  if (event.type === "command" && event.action === "new") {
    try {
      const context = execFileSync('my-tool', ['context'], { encoding: 'utf8' });
      event.messages.push(`My-tool context loaded:\n${context}`);
    } catch (error) {
      console.error(`[my-hook] Error loading context: ${error.message}`);
    }
  }
};

export default handler;
```

### 6. Test Locally

#### For Claude Code

```bash
# Standard install
claude plugin install ./my-plugin

# Test in Claude Code
cd my-project
claude "Use my-skill"
```

#### For OpenClaw

```bash
# Standard install
openclaw plugins install ./my-plugin

# For development/iteration (link mode for hot reload)
openclaw plugins install -l ./my-plugin

# Verify installation
openclaw plugins list | grep my-plugin

# Test
openclaw agent -m "Use my-skill" --local
```

### 7. Package and Publish

---

## Plugin Manifests

### Claude Code Manifest

**File**: `.claude-plugin/plugin.json` or `package.json` (for npm packages)

**Format**:
```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/init.sh"
          }
        ]
      }
    ]
  }
}
```

**Required fields**: None (optional for simple plugins)

**Optional fields**:
- `id`: Unique plugin ID
- `version`: Plugin version
- `name`: Display name
- `description`: Short description
- `hooks`: Hook definitions

### OpenClaw Manifest

**File**: `openclaw.plugin.json` (REQUIRED)

**Format**:
```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "main": "./src/index.ts",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": {
        "type": "string",
        "description": "API Key"
      },
      "enabled": {
        "type": "boolean",
        "default": true
      }
    }
  },
  "uiHints": {
    "apiKey": {
      "label": "API Key",
      "sensitive": true
    }
  },
  "skills": ["./skills"]
}
```

**Required fields**:
- `id`: Unique plugin ID
- `configSchema`: JSON Schema (can be empty `{}`)

**Optional fields**:
- `version`, `name`, `description`
- `main`: Entry point
- `configSchema`: Configuration schema
- `uiHints`: UI hints for configuration
- `skills`: Skill directories
- `hooks`: Hook registrations (optional - auto-discovered from `hooks/` directory)
- `channels`: Messaging channels
- `providers`: Model providers
- `author`, `homepage`, `repository`, `license`, `keywords`

---

## Building CLI Tools

For comprehensive guide, see [Agent-Usable CLI Tools](../patterns/agent-usable-cli-tools.md).

### Key Principles

1. **Self-Documenting**: `--help` provides usage info
2. **Machine-Readable**: `--json` output for agent consumption
3. **Exit Codes**: 0=success, 1=error, 2=usage
4. **Discovery**: `--types` or `--list` for feature discovery
5. **Verbose Mode**: `--verbose` for debugging

### Minimal Example

```typescript
#!/usr/bin/env node
const program = new Command();

program
  .name('my-tool')
  .description('My tool')
  .option('--json', 'JSON output')
  .action(async (options) => {
    const result = { status: 'ok' };
    if (options.json) {
      console.log(JSON.stringify(result, null, 2));
    } else {
      console.log('Status: ok');
    }
  });

program.parse(process.argv);
```

---

## Writing Skills

See [Skill Format Reference](../reference/02-skill-format.md) for complete specification.

### Minimal Skill

```yaml
---
name: my-skill
description: Use my-tool for operations
requires:
  bins: ["my-tool"]
---

# My Skill

Use my-tool to perform operations.

## Search

Search for items with `my-tool search`.

## Create

Create new items with `my-tool create`.
```

### Skill Frontmatter

```yaml
---
name: my-skill
description: Guide for using my-tool CLI
requires:
  bins: ["my-tool", "jq"]
  env: ["MYTOOL_API_KEY"]
  config:
    - config.json
---

# My Skill

## Usage

### Discovery

Check available operations:

```bash
my-tool --types
```

### Search

Search for items:

```bash
my-tool search "<query>"
```

### JSON Output

Get machine-readable output:

```bash
my-tool search --json "query"
```
```

---

## Adding Hooks

See [Hook Events Enhanced Reference](../reference/03-hook-events-enhanced.md) for complete hook event documentation.

### Hook Execution Mechanisms

**Command Hooks**: Shell scripts for deterministic automation
**Prompt Hooks**: Single-turn LLM evaluation for semantic decisions
**Agent Hooks**: Multi-turn verification with tools for complex validation

### Example: PreToolUse Hook

**Claude Code**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/validate.sh"
          }
        ]
      }
    ]
  }
}
```

**OpenClaw**:
```typescript
const handler = async (event) => {
  if (event.type === "command" && event.name === "bash") {
    // Validate command
    if (isDangerous(event.command)) {
      return { blocked: true, reason: "Dangerous command" };
    }
  }
};
```

---

## OpenClaw Plugin API

> **Source**: [OpenClaw Plugins Documentation](https://docs.openclaw.ai/tools/plugin)

Plugins can register various capabilities via the `api` object passed to plugin registration function.

### Registration Methods

| Method | Purpose | Use For |
|--------|---------|----------|
| `api.registerHook(event, handler, options)` | Event-driven automation | Hook registration |
| `api.registerProvider(provider)` | Model provider auth (OAuth, API key) | Auth providers |
| `api.registerChannel(config)` | Messaging channels (WhatsApp, Telegram) | Channel plugins |
| `api.registerGatewayMethod(method, handler)` | Gateway RPC methods | Custom gateway methods |
| `api.registerCli(config, options)` | CLI commands | CLI integration |
| `api.registerCommand(command)` | Auto-reply slash commands | Bot commands |
| `api.registerService(service)` | Background services | Daemon processes |

### Hook Registration

```typescript
export default function(api) {
  api.registerHook(
    "session_start",
    async (event) => {
      const { execFileSync } = require('child_process');
      const context = execFileSync('my-cli', ['context'], { encoding: 'utf8' });
      return { instructions: context };
    },
    {
      name: "my-plugin.session_start",
      description: "Runs when session starts",
    },
  );
}
```

### Gateway RPC Method

```typescript
export default function(api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true, version: "1.0.0" });
  });
}
```

### CLI Commands

```typescript
export default function(api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello from plugin!");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### Auto-Reply Commands

```typescript
export default function(api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin running! Channel: ${ctx.channel}`,
    }),
  });
}
```

### Background Services

```typescript
export default function(api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("Service started"),
    stop: () => api.logger.info("Service stopped"),
  });
}
```

---

## Best Practices

### 1. Keep Plugins Focused

✅ **Single Responsibility**: Each plugin should do one thing well
✅ **Clear Boundaries**: Know what the plugin does and doesn't do
✅ **Minimal Dependencies**: Avoid unnecessary dependencies

**Example**: A database plugin should NOT also handle notifications

### 2. Version Semantically

✅ **Semantic Versioning**: Use `MAJOR.MINOR.PATCH` format
✅ **Breaking Changes**: Increment MAJOR version
✅ **Features**: Increment MINOR version
✅ **Bug Fixes**: Increment PATCH version

```json
{
  "version": "2.1.3"
}
```

### 3. Document Prerequisites

✅ **Skill Requirements**: Skills should state required tools via `requires.bins`
✅ **Hook Requirements**: Hooks should use `requires.bins` in metadata
✅ **Installation Guide**: Provide clear setup instructions

```yaml
---
requires:
  bins: ["my-tool", "jq"]
  env: ["MYTOOL_API_KEY"]
---
```

### 4. Handle Errors Gracefully

✅ **Try-Catch**: Always wrap operations in try-catch
✅ **Meaningful Errors**: Provide helpful error messages
✅ **Exit Codes**: Use correct exit codes (0=success, 1=error, 2=usage)

```typescript
try {
  const result = await operation();
  return result;
} catch (error) {
  console.error(`Operation failed: ${error.message}`);
  process.exit(1);
}
```

### 5. Keep Hooks Lightweight

✅ **Fast Execution**: Hooks run in hot path, keep under 100ms
✅ **Idempotent**: Hooks should handle multiple calls safely
✅ **No Blocking**: Avoid long-running operations in hooks

**Bad**: Slow hook (5 seconds)
```bash
#!/bin/bash
npm run full-test-suite  # Takes 5 seconds - blocks Claude
```

**Good**: Quick validation (10ms)
```bash
#!/bin/bash
if grep -q "danger" <<< "$1"; then
  exit 2
fi
exit 0
```

### 6. Use Environment Variables

✅ **$CLAUDE_PROJECT_DIR**: Project root (Claude Code)
✅ **$CLAUDE_PLUGIN_ROOT**: Plugin root (Claude Code)
✅ **$CLAUDE_ENV_FILE**: Environment file (Claude Code SessionStart)

```bash
# Use variables for portability
command="$CLAUDE_PROJECT_DIR/.claude/hooks/script.sh"
"$command" --arg "$value"
```

---

## Testing

### Unit Test Hooks

```bash
# Test hook with sample input
echo '{"tool_name":"Bash","tool_input":{"command":"rm -rf"}}' | ./hooks/block-danger.sh
echo $?  # Check exit code
```

### Integration Test

```bash
# Deploy and test in session
openclaw agent -m "Use the hello-world skill" --local
```

### Test CLI Tool

```bash
# Test discovery
my-tool --types

# Test JSON output
my-tool search --json "query"

# Test verbose mode
my-tool search --verbose "query"
```

---

## Publishing

### npm Package

```json
{
  "name": "csh-plugin-my-plugin",
  "version": "1.0.0",
  "description": "My CSH plugin",
  "main": "./src/index.ts",
  "csh": {
    "plugins": ["./"]
  },
  "files": [
    "skills",
    "hooks",
    "openclaw.plugin.json"
  ],
  "keywords": ["csh", "plugin", "ai", "cli"]
}
```

```bash
npm publish
```

### GitHub Repository

```bash
git tag v1.0.0
git push --tags
```

---

## Common Patterns

### Skill-Hook Bridge

Use hooks to prepare context that skills consume:

```
Hook: session:start → writes env vars
Skill: reads env vars → uses in workflow
```

### Context Injection

```bash
# hooks/session-start.sh
#!/bin/bash

cat > /tmp/plugin-context.json << EOF
{
  "project": "my-app",
  "environment": "development"
}
EOF
```

### CLI-as-Template-Generator

1. CLI returns file paths (not data)
2. Agent uses Read tool to load files
3. Agent finds relevant information

```bash
# 1. CLI returns paths
$ my-tool search "database"
[
  {"path": "/vault/decisions/2024-postgres.md", "relevance": 0.95}
]

# 2. Agent reads file
# 3. Agent extracts relevant content
```

---

## Not Relevant for CSH Framework

### MCP Server Integration

> ⚠️ **NOT RELEVANT FOR CSH FRAMEWORK**

**What is MCP**:
- Model Context Protocol (MCP) is a **server-client architecture**
- MCP servers expose tools via HTTP/SSE transport

**Why NOT use MCP for CSH**:

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

## Deprecated Features

### HTTP+SSE Transport (MCP)

> ⚠️ **DEPRECATED** (2024-11-05)

**What was deprecated**:
- HTTP+SSE transport for MCP servers
- Replaced by newer transport mechanism

**What to use instead**:
- Use modern MCP transport (WebSocket or HTTP/2)
- Or better: Use CSH Framework (CLI + Skills + Hooks) - no MCP needed

---

## Troubleshooting

### Plugin Loads But Skills Don't Appear

**Cause**: `skills.allowBundled` set to empty array `[]` in openclaw.json

**Fix**: Remove `allowBundled` entirely (no config = all bundled allowed)

### Hook Instructions Not Followed

**Cause**: Agent follows AGENTS.md instead of hook instructions

**Fix**: Add explicit rule in AGENTS.md, or ensure skills are loaded

### require() Fails in Handler

**Cause**: Using ESM imports at module level

**Fix**: Use require() inside handler function:

```javascript
// ❌ Wrong - may fail in hook context
const { execFileSync } = require('child_process');
function handler(event) { ... }

// ✅ Correct - require inside handler
function handler(event) {
  const { execFileSync } = require('child_process');
  const result = execFileSync('my-cli', ['arg'], { encoding: 'utf8' });
  return result;
}
```

### Skills Manifest Wrong Format

**Cause**: Listed individual skill paths instead of directory

**Fix**: Use `"skills": ["./skills"]` pointing to directory

---

## Related Docs

- [Hook Events Enhanced Reference](../reference/03-hook-events-enhanced.md) - Complete hook event reference (17 Claude Code + 8+ OpenClaw events)
- [Skill Format Reference](../reference/02-skill-format.md) - SKILL.md format specification
- [Hooks Patterns Guide](../patterns/03-hooks.md) - Hook design patterns
- [Agent-Usable CLI Tools](../patterns/agent-usable-cli-tools.md) - CLI development guide
- [Platform Comparison](../platforms/03-comparison.md) - Claude Code vs. OpenClaw detailed comparison
- [Cross-Platform Skills](../patterns/04-cross-platform.md) - Multi-platform skill metadata
