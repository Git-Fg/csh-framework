---
title: "Three-Layer Architecture"
summary: "The CLI + Skills + Hooks architecture: Agent → Skills → Hooks → CLI primitives."
read_when:
  - You want to understand the CSH architecture layers
  - You are designing plugin integrations
  - You need to explain CSH to others
---

# Three-Layer Architecture

## Visual Overview

```
┌──────────────────────────────────────────────────────────┐
│                   AGENT LAYER                          │
│  • Makes autonomous decisions                           │
│  • Full context window access                           │
│  • Can bypass skills (high freedom)                    │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼ (guidance when needed)
┌──────────────────────────────────────────────────────────┐
│                 SKILLS LAYER                            │
│  • Soft suggestions for intelligent tasks              │
│  • Step-by-step workflows                             │
│  • Agent can ignore (freedom preserved)                │
│  • Progressive disclosure                              │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼ (hard automation)
┌──────────────────────────────────────────────────────────┐
│                  HOOKS LAYER                            │
│  • Hard hooks: 100% deterministic, cannot skip        │
│  • CLI execution, no AI involved                      │
│  • Safety, formatting, logging, checkpoints            │
└──────────────────────────────────────────────────────────┘
                        │
                        ▼ (primitive tools)
┌──────────────────────────────────────────────────────────┐
│                 CLI PRIMITIVES                          │
│  • git, docker, curl, jq, aws, etc.                  │
│  • Self-documenting, Unix-like                        │
│  • Battle-tested, reliable                            │
└──────────────────────────────────────────────────────────┘
```

---

## The Automation Spectrum

```
100% Deterministic                                    0% Deterministic
      │                                                      │
      ▼                                                      ▼
┌──────────────┐                                    ┌──────────────┐
│  HARD HOOKS │                                    │   SKILLS    │
│  (Auto-run) │                                    │ (Suggestions)│
├──────────────┤                                    ├──────────────┤
│ • Checkpoint │                                    │ • Workflows │
│ • Format     │─────── Agent ────────►            │ • Decisions │
│ • Lint       │   Always executes,               │ • Complex   │
│ • Log        │   cannot skip                    │   tasks     │
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

## Layer 2: Skills (Soft Suggestions)

**Purpose**: Guide agents for tasks requiring judgment

### What Skills Provide

```
SKILL.md
├── name: workflow-name
├── description: What this skill does
├── triggers: When to suggest loading
└── body: Markdown with instructions
```

### Key Characteristics

- **Suggestion, not enforcement**: Agent can ignore
- **Soft guidance**: Makes right choice obvious
- **Progressive disclosure**: Load only when relevant
- **No blocking**: Doesn't prevent agent actions
- **Reference, not embed**: Hooks inject "use X" not full instructions
- **Compression-resistant**: External references survive context trimming

### Example Skill

```yaml
---
name: git-commit-workflow
description: Conventional commits with verification
triggers:
  - commit changes
  - git commit
---

# Git Commit Workflow

Follow this pattern for commits:

## Step 1: Check Status
```bash
git status
```

## Step 2: Stage Files
```bash
git add <files>
```

## Step 3: Conventional Commit
```bash
git commit -m "type(scope): description"
```

## Step 4: Verify
```bash
git log --oneline -3
```
```

---

## Layer 3: Hooks (Context + Control)

**Purpose**: Inject any instructions AND optionally control agent behavior

### Flexibility

- Hooks tailored to YOUR project context
- Each project has different needs: memory, web search, database, etc.
- Keep hooks concise - reference skills, don't embed full instructions
- Fine-tune based on YOUR agent's behavior

### Two Types of Hooks

#### Soft Hooks (Skill Selection Guidance)
- Tell agent WHICH skill to use WHEN - not how to use it
- Let skills contain actual CLI commands
- Session start: tell agent it has access to skills, use them proactively
- Project-specific examples:
  ```
  # Session start - proactive skill usage
  "For this session you have access to web-search skill.
  PROACTIVELY use it anytime a web search could be relevant or unlock you."

  # Memory project - skill menu
  "Available skills:
  - clawvault-remember: when user says 'remember'
  - clawvault-search: when searching past context
  PROACTIVELY use clawvault-search before reading files"
  ```

#### Hard Hooks (Enforcement)
- Can block tool execution
- Enforce safety rules
- Require explicit approval

### Key Characteristics

- Run automatically at lifecycle events
- Project-specific: tailored to YOUR project context
- Keep concise: reference skills, don't embed instructions
- Fast CLI execution, no AI involved

### Example Hooks

```json
{
  "hooks": {
    "SessionStart": [
      {
        "type": "command",
        "command": "./hooks/inject-context.sh"
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/safety-check.sh"
          }
        ]
      }
    ]
  }
}
```

### Hook Output Examples (skill selection)

```bash
# Session start - skill menu
"Available skills:
- clawvault-remember: when user says 'remember'
- clawvault-search: when searching past context
- clawvault-forget: when removing memories"

# Skill reminder
"YOU MUST use web-search for current events"
```

### Hook Examples

| Hook Type | Event | Purpose |
|-----------|-------|---------|
| Soft | SessionStart | Skill menu: "which skill to use when" |
| Soft | Before task | Suggest relevant skill |
| Hard | PreToolUse(Bash) | Block dangerous commands |

### Skill Selection vs CLI Commands

- **Hooks** → "which skill to use" (skill selection)
- **Skills** → "how to use it" (CLI commands)
- Don't put CLI commands in hooks - put them in skills

---

## Hooks vs Skills

| Aspect | Hooks | Skills |
|--------|-------|--------|
| **Execution** | Automatic at events | On-demand |
| **Soft** | Inject context | Guidance format |
| **Hard** | Can block execution | Cannot block |
| **Reliability** | Always runs | ~30% invocation |

---

## Architectural Principles

1. **CLI Primitives First**: use tools agents already understand
2. **Skills Format Guidance**: teach correct patterns, agent can ignore
3. **Hooks Inject + Control**: context for guidance, enforcement when needed
4. **Preserve Freedom**: soft hooks guide, hard hooks when necessary
5. **Unix Simplicity**: No servers, just CLI + files

---

## Platform-Specific Implementation

See [Platform Comparison](./openclaw/platform-comparison.md) for how each layer works on:
- Claude Code
- OpenClaw

---

*Next: [Skill Format Reference](../reference/02-skill-format.md)*
