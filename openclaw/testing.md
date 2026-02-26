---
title: "OpenClaw Testing Guide"
summary: "Professional dev workflow for testing CSH Framework plugins in OpenClaw - Gateway, CLI, HTTP, and log-based verification."
read_when:
  - You are testing OpenClaw plugins
  - You want to verify skill/hook loading
  - You need diagnostic commands
---

# OpenClaw Testing Guide for CSH Framework

> **The Professional Workflow**: Verify Gateway -> Preliminary CLI Checks -> Real-World HTTP Testing -> Behavioral Log Audit -> Iterative Refinement

---

## Mission: The Professional Dev Workflow

This guide establishes the diagnostic-first approach to CSH Framework development. Instead of sending messages and hoping they work, follow a structured, non-interactive path to ensure the control plane is healthy and the logic is sound.

1.  **Verify Gateway**: Ensure the engine is running before you drive.
2.  **Preliminary CLI Check**: Confirm skills and hooks are loaded and visible.
3.  **Real Interaction (HTTP)**: Send programmatic prompts via CLI to see how the agent handles them.
4.  **Behavioral Audit (Logs)**: Verify internal behavior via system logs.
5.  **Iterative Refinement**: Compare logs to intent and refine code.

---

## Phase 1: Verify Gateway Health

Before testing any skill, ensure the OpenClaw Gateway is operational.

```bash
# Check the immediate status
openclaw gateway status

# If state is "stopped" or "error", restart:
openclaw gateway restart

# Deep health check for model availability
openclaw health
```

**Success Criteria**:
- Service state: running
- Control plane: active
- RPC probe: ok

---

## Phase 2: Preliminary Configuration Check (CLI)

Use the CLI to verify that OpenClaw "sees" your CSH changes.

### Verification Commands
```bash
# List all loaded skills
openclaw skills list | grep "csh"

# List enabled plugins and hooks
openclaw plugins list

# Verify your specific skill is active
openclaw skills list | grep -i "memory-search"
```

### Hot Reload in Development
Install your plugin using the link mode for instant updates:
```bash
openclaw plugins install -l /path/to/csh-framework
```

---

## Phase 3: Real Interaction via HTTP (Non-Interactive)

Use the HTTP endpoint to simulate how sessions behave in production.

### Base Configuration
- **Endpoint**: POST http://127.0.0.1:18789/v1/chat/completions
- **Auth**: Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN

### Scenario: Generic Discovery Prompt
```bash
curl -s -X POST http://127.0.0.1:18789/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN" \
  -d '{
    "model": "agent:main",
    "messages": [
      { "role": "user", "content": "I need to find all information about Project X in my memory." }
    ]
  }' | jq '.'
```

---

## Phase 4: High-Fidelity CLI Turn Testing

For faster verification without manual chat, use the `openclaw agent` command.

```bash
# Test a direct tool call
openclaw agent -m "I need to find files related to project X"

# Verify JSON output for integration tests
openclaw agent -m "List my active skills" --json | jq '.'
```

---

## Phase 5: Behavioral Audit (The "Step Back")

Do not trust the final text output alone. Verify internal behavior via logs.

> [!CAUTION]
> **NEVER trust an agent's response** when it claims it used a skill. Agents often hallucinate tool usage. **Always verify** execution via logs.

### Analyzing Logs
Keep logs streaming in a separate shell or capture them:
```bash
openclaw logs --follow | grep -E "tool|hook|error"
```

**What to look for**:
- **Tool Calls**: `[info] tool_call: memory-search { query: "Project X" }`
- **Hallucinations**: Did the agent pretend to search without a matching `tool_call` entry?
- **Hooks**: Did `session-start` or `pre-tool-use` hooks fire correctly?

---

## Advanced Diagnostic Workflows

Leverage OpenClaw's own intelligence for debugging.

### 1. Self-Diagnostic Reflection
Ask the agent to reflect on its own configuration (requires Dev mode).
```bash
openclaw agent -m "Diagnose the tool registration for 'memory-search'. Check if hooks are blocking it."
```

### 2. Absolute Repo Review
Reference the absolute path to the repository for source-of-truth analysis.
```bash
openclaw agent -m "Review source code in /root/csh-dev/csh-project against recent logs and identify why hooks are failing."
```

### 3. Read-Only Audits
Request a descriptive analysis without modifications.
```bash
openclaw agent -m "Analyze /root/csh-dev/csh-framework/src/hooks/session-start.ts for race conditions. DO NOT use edit tools."
```

---

## Success Criteria
- [x] Gateway is verified active via CLI.
- [x] Skills/Hooks are visible in `openclaw skills list`.
- [x] CLI Agent turns are tested and audited.
- [x] Logs verify tool calls (no hallucinations).

---
*Last Updated: 2026-02-24*
