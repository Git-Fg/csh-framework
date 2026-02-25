---
title: "OpenClaw Platform Guide"
summary: "Integration patterns, skill system, hooks, and session lifecycle when using CSH with OpenClaw."
read_when:
  - You are integrating with OpenClaw
  - You want to understand OpenClaw skill loading
  - You need to implement OpenClaw-specific features
---

# Platform Guide: OpenClaw

> **Source**: [OpenClaw Documentation](https://docs.openclaw.ai/)

This document covers the integration patterns, skill system, hooks, and session lifecycle when using the CSH framework with OpenClaw.

---

> **Scope**: The CSH framework is designed primarily for **Claude Code** and **OpenClaw** AI agents, each with specific integration patterns. However, the architecture (CLI + Skills + Hooks) is generic — it works with **any AI agent platform that supports plugin bundling** containing these three primitives.

---

## How Skills Are Discovered and Loaded

### Discovery Mechanism

OpenClaw uses a dynamic skill discovery system with feature gating.

> **NOTE**: For CSH Framework, skills are typically **plugin-bundled** (shipped with plugins) rather than using the global skill discovery paths below. Plugin skills live in `<plugin>/skills/<name>/SKILL.md` and are referenced via `"skills": ["./skills"]` in the plugin manifest.

#### Global Skill Paths (for reference)

1. **Scanned Locations** (in precedence order):
   - Workspace skills: `~/.openclaw/skills/` (or `<workspace>/skills/`)
   - Managed skills: `~/.openclaw/skills/`
   - Bundled skills: Distributed with the gateway

2. **Feature Gating**: Skills can declare runtime requirements using nested metadata:
   ```yaml
   ---
   name: my-skill
   description: Does something useful
   metadata:
     {
       "openclaw":
         {
           "requires": { "bins": ["curl", "jq"], "env": ["API_KEY"] },
         },
     }
   ---
   ```

3. **Metadata-First Loading**: OpenClaw loads YAML frontmatter first to optimize prompt density before loading full skill instructions.

### Loading Behavior

Skills are discovered via CLI:
```bash
openclaw skills list
openclaw memory list --types
```

### Permission Control

Skills can be explicitly enabled or disabled:
```bash
openclaw skills enable <skill-name>
openclaw skills disable <skill-name>
```

---

## Hook System

### Hook Configuration

OpenClaw hooks are configured via two components:
- **Instructions**: `HOOK.md` file containing guidance
- **Logic**: `handler.ts` (or shell script) containing executable code

### Hook Discovery Precedence

Hooks are scanned with this priority:
1. Workspace hooks
2. Managed hooks
3. Bundled hooks

### Automation Triggers

Hooks can be triggered by:
- **Messages**: Incoming commands from users
- **Heartbeats**: Periodic batched checks (~30 minute intervals)
- **Webhooks**: External HTTP callbacks
- **Cron**: Exact schedule timing

### Supported Hook Types

| Category | Events |
|----------|--------|
| **Lifecycle** | `session_start`, `session_end` |
| **Command** | `before_command`, `after_command` |
| **System** | `webhook`, `cron`, `heartbeat` |

### Hook Handler Pattern

```typescript
// hooks/my-hook/handler.ts
function handler(event: HookEvent) {
  const { type, payload } = event;

  switch (type) {
    case 'before_command':
      // Pre-execution logic
      return { continue: true };
    case 'after_command':
      // Post-execution logic
      return { success: true };
    default:
      return { continue: true };
  }
}

module.exports = { handler };
```

### Bundled Hooks

Hooks can be bundled within plugins to provide out-of-the-box infrastructure automation. Configure in manifest:

```json
{
  "hooks": {
    "before_command": "./hooks/pre-command"
  }
}
```

---

## Manifest Format

### Plugin Manifest

OpenClaw plugins use `openclaw.plugin.json`:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Example plugin for OpenClaw",
  "skills": ["./skills"],
  "hooks": {
    "before_command": "./hooks/pre-command"
  },
  "settings": {
    "allowBundled": ["skill-a", "skill-b"]
  }
}
```

### Allowlist Configuration

**CRITICAL**: When `skills.allowBundled` is set to an empty array `[]`, it becomes RESTRICTIVE - no bundled skills will load.

Options:
1. **Omit entirely** - All bundled skills allowed (recommended)
2. **List explicitly** - Only listed skills allowed: `["skill-a", "skill-b"]`
3. **Avoid empty array** - `[]` blocks all bundled skills

### Skill Metadata (SKILL.md Frontmatter)

```yaml
---
name: example-skill
description: Example skill demonstrating OpenClaw format
trigger: /example
platform: openclaw
requires:
  bins: ["curl"]
  env: ["API_KEY"]
versioned: true
---
```

---

## Session Lifecycle

### Session Management

OpenClaw manages sessions centrally through the gateway:

1. **Queue Modes**:
   - `collect`: Coalesce multiple messages before processing
   - `followup`: Sequential processing
   - `steer`: Injection at tool boundaries

2. **Isolation**:
   - `secure DM mode`: Isolates context per sender/channel
   - Prevents cross-talk in multi-user environments

3. **Persistence**:
   - Sessions survive restarts via JSONL transcript storage
   - State stored in `~/.openclaw/state/session.json`

### State File

OpenClaw provides a visible, editable state file:
- Location: `~/.openclaw/state/session.json`
- Developer-friendly: Human-readable JSON format

---

## Memory System

### Native Memory Options

OpenClaw has built-in memory plugins that handle persistence:

| Plugin | Purpose | Storage |
|--------|---------|---------|
| **memory-core** | Default session memory | JSONL transcripts |
| **memory-lancedb** | Long-term semantic recall | LanceDB vector store |

Select via `plugins.slots.memory` in config.

### ⚠️ Avoid Creating Memory-Type Plugins

**Bad Practice**: Building standalone "memory plugins" (type `kind: "memory"`) that try to replace or duplicate native memory.

**Why It's Problematic**:
- Competes with built-in memory plugins for the exclusive slot
- Requires users to choose between your plugin and native features
- Duplicates functionality already provided by memory-core/LanceDB
- Creates configuration burden for users

### ✅ Better Approach: Enhance Native Memory

Instead of creating memory plugins, build **skills and hooks** that enhance the existing native memory:

1. **Skills** that teach agents how to use memory effectively:
   - Semantic search patterns
   - Memory organization strategies
   - Context injection from memory

2. **Hooks** that interact with memory at the right moments:
   - Extract key info before session end
   - Suggest relevant memories at session start
   - Tag/categorize important decisions

3. **Tools** that extend memory capabilities:
   - Advanced search interfaces
   - Memory visualization
   - Export/import functionality

This approach:
- Works with whatever memory plugin the user has configured
- Adds value without competing for slots
- Requires zero configuration from users
- Leverages the existing memory infrastructure

### Lifecycle Events

1. **session_start** - Gateway session initialized
2. **session_end** - Gateway session terminates
3. **before_command** - Before any command executes
4. **after_command** - After command completes

### Session Optimization

The gateway optimizes the system prompt by injecting only the most relevant skill frontmatter, avoiding context bloat.

---

## Integration Patterns

### Hybrid Setup

When using OpenClaw alongside other platforms:

1. **Naming Convention**: Use unique prefixes for skills
   - OpenClaw skills: `openclaw-<skill-name>`
   - Other platform skills: `<platform>-<skill-name>`

2. **Hook Order**: OpenClaw hooks typically run before other platform hooks to ensure global context is loaded first

3. **Memory Coordination**: Use OpenClaw for persistent "Vault" memory; use native platform features for ephemeral session notes

### Multi-Channel Access

OpenClaw supports:
- Desktop CLI access
- Mobile/Telegram integration
- Webhook triggers

### Best Practices

- Leverage feature gating for OS/environment-specific skills
- Use workspace hooks for project-specific automation
- Keep state files in `~/.openclaw/` for persistence
- Version skills using git for distribution

---

## Summary

| Aspect | OpenClaw Behavior |
|--------|-------------------|
| **Skill Discovery** | Dynamic scan with feature gating |
| **Skill Loading** | Metadata-first, gated by `requires` |
| **Hook Events** | session_start/end, before/after_command, webhook, cron, heartbeat |
| **Hook Format** | `HOOK.md` + `handler.ts` |
| **Manifest** | `openclaw.plugin.json` |
| **Session** | Gateway-managed, persistent |
| **Persistence** | JSONL transcripts + state file |
| **Multi-channel** | Desktop, Mobile, Webhook |

