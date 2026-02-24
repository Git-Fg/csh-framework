# Three-Layer Architecture

The CSH Framework uses a three-layer architecture that balances capability with efficiency.

---

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      AGENT CONTEXT                          │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Hooks     │ Automation, Safety, Context          │
├─────────────────────┼───────────────────────────────────────┤
│  Layer 2: Skills    │ Workflows, Patterns, Best Practices    │
├─────────────────────┼───────────────────────────────────────┤
│  Layer 1: CLI      │ Primitives (git, curl, jq, etc.)     │
└─────────────────────┴───────────────────────────────────────┘
```

---

## Layer 1: CLI Primitives

**Purpose**: Single, focused operations that agents can invoke directly.

### What Qualifies as a Primitive?
- `git`, `docker`, `kubectl` — Version control & containers
- `curl`, `wget` — Network requests
- `jq`, `yq` — Data processing
- `find`, `grep`, `awk` — File operations
- Any CLI tool with `--help` documentation

### Design Requirements

| Requirement | Description | Why It Matters |
|-------------|-------------|----------------|
| **Self-documenting** | `--help` provides complete usage | Agents discover capability |
| **Absolute paths** | Always use `/abs/path`, never `./rel` | Predictable output |
| **JSON output** | `--json` flag for machine parsing | Structured results |
| **Exit codes** | 0 = success, non-zero = error | Error handling |
| **Progressive disclosure** | `--verbose` for full output | Token efficiency |

### Example Primitive Interface

```bash
# Self-documenting
$ mytool --help
Usage: mytool <command> [options]

Commands:
  list    List items
  get     Get item by ID
  create  Create new item

# JSON output
$ mytool list --json
[{"id": "1", "name": "A"}, {"id": "2", "name": "B"}]

# Error handling
$ mytool get missing-id
Error: not found (404)
$ echo $?
1
```

---

## Layer 2: Skills

**Purpose**: Encode complex workflows as reusable patterns.

### What a Skill Provides

```
SKILL.md
├── name: workflow-name
├── description: What this skill does
├── triggers: When to load (keywords, file patterns)
└── body: Markdown with instructions + examples
```

### Skill Structure

```yaml
---
name: resilient-web-scrape
description: Scrape websites with retry logic and rate limiting
triggers:
  - web scraping
  - fetch HTML
  - scrape page
platforms:
  - claude-code
  - openclaw
---

# Resilient Web Scraping

When you need to scrape a website, follow this pattern:

## Step 1: Checkrobots.txt
Before scraping, check if allowed:
```bash
curl -s https://example.com/robots.txt | grep -q "Disallow" && echo "blocked"
```

## Step 2: Rate Limit
Add delays between requests:
```bash
sleep $((RANDOM % 3 + 1))
```

## Step 3: Retry Logic
Wrap in retry loop:
```bash
for i in 1 2 3; do
  curl -s "$url" && break
  sleep 5
done
```

## Anti-Patterns
- Don't scrape without checking robots.txt
- Don't fire requests without rate limiting
- Don't give up after single failure
```

### Skill Loading Behavior

| Platform | When Loaded | How |
|----------|-------------|-----|
| **Claude Code** | Agent invokes skill tool | On-demand via skill name |
| **OpenClaw** | Session start + explicit load | Bootstrap or `/skill read` |

---

## Layer 3: Hooks

**Purpose**: Event-driven automation that runs without agent invocation.

### Hook Types

| Event | When Fires | Common Use Cases |
|-------|------------|------------------|
| `session:start` | New session begins | Load context, set variables |
| `session:end` | Session completes | Save state, cleanup |
| `before:command` | Before tool execution | Validation, warnings |
| `after:command` | After tool execution | Logging, notifications |
| `prompt:inject` | Before response | Context reinforcement |

### Hook Manifest

```json
{
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/startup.sh",
      "description": "Load project context"
    },
    {
      "event": "before:command",
      "handler": "./hooks/dry-run.sh",
      "description": "Warn on destructive commands"
    }
  ]
}
```

### Hook Execution Model

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Trigger    │ ──▶  │   Handler    │ ──▶  │   Result     │
│  (event)     │      │ (CLI/script) │      │ (injected)   │
└──────────────┘      └──────────────┘      └──────────────┘
```

---

## Token Efficiency Mechanics

### Progressive Disclosure Flow

```
Task arrives
     │
     ▼
┌─────────────────┐
│ Context Window  │
│ (minimal)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     No      ┌─────────────────┐
│ Skill Match?    │───────────▶│ Use Primitives  │
└────────┬────────┘            └─────────────────┘
         │ Yes
         ▼
┌─────────────────┐
│ Load Skill     │
│ (50-200 lines) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Execute with   │
│ Skill Guidance │
└─────────────────┘
```

### Token Comparison

| Scenario | MCP | CSH Framework |
|----------|-----|---------------|
| Simple task | 50,000 tokens | 4,000 tokens |
| Medium task | 80,000 tokens | 12,000 tokens |
| Complex task | 120,000 tokens | 25,000 tokens |

**Key Insight**: CSH Framework processes data in the execution environment, not in the reasoning context.

---

## Architectural Principles

1. **Primitives First**: Always prefer CLI tools that agents already understand
2. **Skills Encode Patterns**: Don't abstract commands, teach correct usage
3. **Hooks Run Automatically**: Don't require agent invocation
4. **Progressive Disclosure**: Load more context only when needed
5. **Privacy by Default**: Keep intermediate results in execution environment

---

## Platform-Specific Implementation

See [Platform Comparison](../platforms/03-comparison.md) for how each layer works differently on:
- Claude Code
- OpenClaw

---

*Next: [Trade-offs Assessment](./03-tradeoffs.md)*
