# CSH Framework User Guide

The complete guide to understanding and using the CLI + Skills + Hooks architecture for AI agent extensions.

---

## What is the CSH Framework?

The **CSH Framework** (CLI-Skills-Hooks) is a Unix-like architecture for building AI agent extensions through three complementary layers:

- **CLI Primitives**: Reliable Unix-like tools agents already understand (git, docker, curl, jq)
- **Skills**: Guidance formats that teach agents correct patterns
- **Hooks**: Context injection + optional enforcement at lifecycle events

### The Core Problem

AI agents struggle to use skills correctly, even with perfect descriptions:

| Metric | Rate |
|--------|------|
| Task-skill match (recognizing relevance) | ~92% |
| Actual skill invocation | ~30% |
| **Gap lost to mode-switching** | ~65 percentage points |

Agents know *what* to do but fail to *execute* correctly with skills.

### How CSH Solves This

1. **Skills** format guidance as step-by-step workflows (agent can ignore)
2. **Hooks** inject context to nudge agents toward correct skill usage
3. **Mandatory language** in hooks counters AI laziness ("YOU MUST use X")

---

## The Three-Layer Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   AGENT LAYER                          │
│  • Makes autonomous decisions                           │
│  • Full context window access                           │
│  • Can bypass skills (high freedom)                    │
└──────────────────────────────────────────────────────────┘
                        │ guidance when needed
                        ▼
┌──────────────────────────────────────────────────────────┐
│                 SKILLS LAYER                            │
│  • Soft suggestions for intelligent tasks              │
│  • Step-by-step workflows                             │
│  • Agent can ignore (freedom preserved)                │
│  • Progressive disclosure                              │
└──────────────────────────────────────────────────────────┘
                        │ hard automation
                        ▼
┌──────────────────────────────────────────────────────────┐
│                  HOOKS LAYER                            │
│  • Hard hooks: 100% deterministic, cannot skip        │
│  • CLI execution, no AI involved                      │
│  • Safety, formatting, logging, checkpoints            │
└──────────────────────────────────────────────────────────┘
                        │ primitive tools
                        ▼
┌──────────────────────────────────────────────────────────┐
│                 CLI PRIMITIVES                          │
│  • git, docker, curl, jq, aws, etc.                  │
│  • Self-documenting, Unix-like                        │
│  • Battle-tested, reliable                            │
└──────────────────────────────────────────────────────────┘
```

### The Automation Spectrum

```
100% Deterministic                                    0% Deterministic
      │                                                      │
      ▼                                                      ▼
┌──────────────┐                                    ┌──────────────┐
│  HARD HOOKS │                                    │   SKILLS    │
│  (Auto-run) │                                    │ (Suggestions)│
├──────────────┤                                    ├──────────────┤
│ • Checkpoint │─────── Agent ────────►            │ • Workflows │
│ • Format     │   Always executes,               │ • Decisions │
│ • Lint       │   cannot skip                    │ • Complex   │
│ • Log        │                                    │   tasks     │
│ • Safety     │                                    │             │
└──────────────┘                                    └──────────────┘
```

---

## Layer 1: CLI Primitives

**Purpose**: Reliable Unix-like tools agents already understand

### Design Requirements

| Requirement | Description | Example |
|------------|-------------|---------|
| Self-documenting | `--help` provides complete usage | `mytool --help` |
| Absolute paths | Always use `/abs/path` | `/workspace/file` |
| Natural output | Clear, readable text | `mytool list` |
| Exit codes | 0 = success, non-zero = error | `if [ $? -eq 0 ]` |
| No server | CLI-only, no infrastructure | Just a binary |

---

## Layer 2: Skills

**Purpose**: Guide agents for tasks requiring judgment

### What Skills Provide

- Step-by-step workflow instructions
- Clear success criteria
- Error handling patterns
- Best practice reminders

### Key Characteristics

- **Suggestion, not enforcement**: Agent can ignore
- **Soft guidance**: Makes right choice obvious
- **Progressive disclosure**: Load only when relevant
- **No blocking**: Doesn't prevent agent actions
- **Reference, not embed**: Hooks inject "use X" not full instructions
- **Compression-resistant**: External references survive context trimming

### Critical Rule

> Skills must NEVER abstract away CLI primitives. They teach patterns, not magic wrappers.

---

## Layer 3: Hooks

**Purpose**: Inject relevant instructions AND optionally control agent behavior

### Two Types of Hooks

#### Soft Hooks (Context Injection)
- Tell agent WHICH skill to use WHEN - not how to use it
- Inject reminders, context, best practices
- Agent receives guidance but can ignore
- Always runs, non-blocking

#### Hard Hooks (Enforcement)
- Can block tool execution
- Enforce safety rules
- Require explicit approval

### Hook Output Examples

```bash
# Session start - proactive skill usage
"For this session you have access to web-search skill.
PROACTIVELY use it anytime a web search could be relevant or unlock you."

# Skill reminder - mandatory language
"YOU MUST use web-search for current events"
```

### Hooks vs Skills

| Aspect | Hooks | Skills |
|--------|-------|--------|
| **Execution** | Automatic at events | On-demand |
| **Soft** | Inject context | Guidance format |
| **Hard** | Can block execution | Cannot block |
| **Reliability** | Always runs | ~30% invocation |

### Key Insight

- **Hooks** → "which skill to use" (skill selection)
- **Skills** → "how to use it" (CLI commands)
- Don't put CLI commands in hooks - put them in skills

---

## Getting Started (Pattern Example)

### Step 1: Initialize Your Plugin Structure

Since CSH is a methodology, you start by creating the directory structure for your specific plugin (e.g., `my-plugin`):

```bash
mkdir -p my-plugin/skills
mkdir -p my-plugin/hooks
```

This establishes:

```
my-plugin/
├── skills/            # Your skill definitions
│   └── hello-world/
│       └── SKILL.md
├── openclaw.plugin.json  # Plugin manifest
└── hooks/             # Hook handlers (optional)
    └── start.sh
```

### Step 2: Create Your First Skill

Create `my-plugin/skills/hello-world/SKILL.md`:

```yaml
---
name: hello-world
description: Prints a greeting. Use when you need to test the setups.
---

# Hello World Skill

A simple skill that demonstrates the CSH pattern.

## Workflow

1. Run: `echo "Hello from my portable plugin!"`
```

### Step 3: Test Your Plugin

**For OpenClaw:**
```bash
# Register the plugin directory in ~/.openclaw/openclaw.json or copy to plugins/
cp -r my-plugin ~/.openclaw/plugins/
openclaw gateway restart
```

**For Claude Code:**
```bash
# Install the local plugin
claude plugin install ./my-plugin
```

### Step 4: Add a Hook (Optional)

Create `hooks/start.sh`:
```bash
#!/bin/bash
echo "Session starting at $(date)"
```

---

## Platform Comparison: Claude Code vs OpenClaw

### Executive Summary

| Requirement | Claude Code | OpenClaw |
|-------------|-------------|-----------|
| Personal automation | ✅ Excellent | ✅ Excellent |
| Team collaboration | ❌ Limited | ✅ Excellent |
| Persistent sessions | ❌ Ephemeral | ✅ Persistent |
| Cron/webhook automation | ❌ No | ✅ Yes |
| Quick prototyping | ✅ Excellent | ✅ Good |
| Production deployment | ⚠️ Requires effort | ✅ Production-ready |
| Multi-platform support | ❌ Claude only | ✅ Universal |

**Bottom Line**: Choose based on your use case — there is no "best" platform.

### When to Choose Claude Code

**Use When**:
- Running locally for personal use
- Need quick iteration and prototyping
- Want minimal infrastructure
- Single-user scenario
- Prefer seamless, out-of-the-box experience

**Strengths**:
- ✅ Excellent for Claude-native tasks
- ✅ Tightly integrated workflows
- ✅ Seamless, out-of-the-box experience
- ✅ Quick to set up and iterate

### When to Choose OpenClaw

**Use When**:
- Need persistent sessions across restarts
- Require gateway management and monitoring
- Multi-user or team scenario
- Need automation triggers (cron, webhooks, heartbeats)
- Building production systems

**Strengths**:
- ✅ Universality: Skills work with any agent platform
- ✅ Independence: Skills can be versioned and distributed
- ✅ Gateway ecosystem for enterprise features
- ✅ Better separation of concerns

---

## Best Practices

### 1. Agent-Usable CLI Tools

**Discovery First**: Every family of commands must provide a way to discover available options:
- Use `--types` to list valid categories
- Use `--list` to enumerate available templates

**JSON-First Output**: LLMs handle JSON better than unstructured text:
```bash
# Good: Structured JSON
$ mytool search "database" --output json

{"results": [{"file": "data.sql", "size": 1024}]}
```

### 2. Skill Descriptions

Use the **"What-When-Not" Pattern**:

1. **What/Why**: Primary value proposition
2. **WHEN**: Clear, actionable triggers
3. **NOT**: Explicit false positives

**Good (Single-Line)**:
```yaml
description: "Creates daily backups at midnight."
```

**Bad (Multi-Line)**:
```yaml
description: |
  This skill performs complex database migration...
```

### 3. Hook Language

Use **mandatory language** to counter AI laziness:
- ❌ "Consider using X" → ignored
- ✅ "YOU MUST use X skill" → higher compliance

### 4. The Skill-Hook Bridge

Because agents are inconsistent at invoking skills, use hooks at three levels:

| Level | Type | Description |
|-------|------|-------------|
| **Level 1** | Guidance | Gentle reminder |
| **Level 2** | Assertion | Explicit nudge |
| **Level 3** | Context Injection | Provide necessary context |

---

## When to Use CSH Framework

### Best For

1. **Context-aware agents**: Where session memory matters
2. **Workflow guidance**: Where agents need reminders
3. **Unix philosophy**: Simple, portable, CLI-based
4. **Privacy-sensitive**: No server required, data stays local

### Not For

1. **Simple one-off tools**: Overhead not worth it
2. **Enterprise MCP compliance**: Use MCP if required

---

## CSH vs MCP Comparison

| Aspect | CSH Framework | MCP |
|--------|--------------|-----|
| **Architecture** | CLI + files (Unix-like) | Server-based |
| **Complexity** | Simple, portable | Requires server |
| **Token Efficiency** | Progressive disclosure | All tools loaded |
| **Context** | Hooks inject reminders | Limited |
| **Agent Freedom** | High (skills are suggestions) | Schema-bound |
| **Setup** | Just CLI tools | Server infrastructure |

---

## Key Takeaways

1. **CLI primitives** are the foundation - tools agents already understand
2. **Skills format guidance** - teach correct patterns, agent can ignore
3. **Skills = external references** - minimize context cost, survive compression
4. **Hooks inject context + control** - use mandatory language for AI compliance
5. **Soft hooks**: context with strong directives ("YOU MUST")
6. **Hard hooks**: block when necessary for safety
7. **Session lifetime**: counter "lost in middle" by referencing skills
8. **Unix simplicity**: No servers, just CLI + files

---

## Next Steps

| Goal | Doc |
|------|-----|
| More skill patterns | `docs/core/skills.md` |
| Hook automation | `docs/core/hooks.md` |
| Plugin development | `docs/openclaw/overview.md` |
| Platform specifics | `docs/claudecode/overview.md`, `docs/openclaw/overview.md` |
| Cross-platform skills | `docs/core/cross-platform.md` |
| CLI design | `docs/core/development.md` |
| Testing | `docs/openclaw/testing.md` |

---

*For technical specifications (SKILL.md format, hook events, CLI reference), see the files in `docs/openclaw/`.*
