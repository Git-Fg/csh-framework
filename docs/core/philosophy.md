---
title: "Philosophy"
summary: "Why CLI + Skills + Hooks - the core problem CSH solves and the design principles behind it."
read_when:
  - You want to understand why CSH exists
  - You are designing agent architectures
  - You need to justify CSH to stakeholders
---

# Why CLI + Skills + Hooks?

The CSH Framework (CLI-Skills-Hooks) provides a Unix-like architecture for building AI agent extensions through CLI primitives, skills for guidance, and hooks for context injection.

---

## The Core Problem

### AI Agents Fail to Use Skills Correctly

Current AI agents struggle to invoke and use skills, even with perfect descriptions:

- **Task-skill match**: ~92% (agents recognize when a skill is relevant)
- **Actual skill invocation**: ~30% (agents fail to actually use the skill)
- **Gap**: ~65 percentage points lost to mode-switching resistance

Agents know *what* to do but fail to *execute* correctly with skills.

---

## CSH Framework: Three Layers

### Layer 1: CLI Primitives (Foundation)

**Purpose**: Single, focused Unix-like operations

**Design Principles**:
- Self-documenting (`--help` provides complete manual)
- Absolute paths for predictable output
- JSON output for machine parsing
- Exit codes for error handling
- No server required — just CLI tools

**What This Provides**:
- Universal tool access (git, aws, docker, curl, jq, etc.)
- Battle-tested reliability (decades of Unix tooling)
- Text-based interfaces (composable via pipes)
- Direct filesystem access for state persistence

**Key Insight**: LLMs have been trained on vast amounts of shell scripts. They "know" CLI tools naturally.

---

### Layer 2: Skills (Guidance Format)

**Purpose**: Format how to teach agents correct patterns + minimize context cost

**Problem Solved**: Agents fail to follow correct workflows + context compression loses instructions.

**Skills Provide**:
- Step-by-step workflow instructions
- Clear success criteria
- Error handling patterns
- Best practice reminders

**Session Lifetime Benefits**:
- **Reference, not embed**: Hooks inject "use 'skill-name'" not full instructions
- **Counter compression**: Agent knows where to find guidance when context compresses
- **Minimize context cost**: Small reference vs 1000s of tokens of instructions
- **"Lost in middle" resilience**: Skills are external, not in context window

**Design Principles**:
- Skills format guidance, don't enforce
- Agent can ignore skills (freedom preserved)
- Skills make the right choice obvious
- File-based: SKILL.md with YAML metadata + markdown
- Reference by name, load on demand

**What This Provides**:
- Domain expertise encoding
- Multi-tool orchestration templates
- Best practice reminders
- Progressive disclosure (load only when needed)
- Compression-resistant: external references survive context trimming

**Critical Rule**: Skills must NEVER abstract away CLI primitives. They teach patterns, not magic wrappers.

---

### Layer 3: Hooks (Context + Control)

**Purpose**: Orchestrate agent behavior through context injection and automated reinforcement.

**Core Capacity: Elicitation**
The most powerful use of hooks is to guide the agent toward the correct **Skill** (Layer 2) at the right moment. Hooks counter the "Mode-Switching Problem" by injecting instructions that make skill invocation obvious.

**Capacity: Integrated Automation**
Hooks automate background tasks (side effects) while simultaneously providing guidance.

**Two Types of Hooks**:

#### Guidance Hooks (Instruction Injection)
- Tell agent WHICH skill to use WHEN - not how to use it
- Inject reminders, environment state, and best practices
- Use **MANDATORY language** to counter AI laziness
- Example: *"YOU MUST use the 'db-query' skill for current events"*

#### Enforcement Hooks (Hard Hooks)
- Deterministically block or allow specific tool uses
- Enforce project-level safety and compliance rules
- Example: Block `rm -rf` on critical directories

**Infinite Tunability**:
- Hooks tailored to YOUR project context
- Each project has different needs: memory, web search, database, etc.
- Fine-tune hooks based on YOUR agent's behavior
- Keep hooks concise - reference skills for details, don't embed instructions

**What Hooks Can Do** (project-specific):
- Remind about YOUR project's skills
- Add constraints relevant to YOUR workflow
- Inject project context at session start
- Block operations specific to YOUR project
- Log and notify based on YOUR needs

**Example Hooks** (skill selection, not CLI commands):
```bash
# Session start - proactive skill usage
"For this session you have access to web-search skill.
PROACTIVELY use it anytime a web search could be relevant or unlock you."

# Session start - skill menu
"Available skills:
- clawvault-remember: when user says 'remember' or 'note this'
- clawvault-search: when searching for past context
PROACTIVELY use clawvault-search before reading files"

# Skill reminder - when to use
"YOU MUST use web-search for current events"
```

**Principles**:
- Hooks → "which skill to use" (skill selection)
- Skills → "how to use it" (CLI commands)
- Session start → skill menu with "when to use" guidance
- Keep hooks concise - reference skills, don't embed instructions

---

## Hooks: Soft vs Hard

Hooks serve **two purposes**:

### Soft Hooks (Context Injection)
- Inject reminders, context, best practices
- Agent receives guidance but can ignore
- Always runs, non-blocking

### Hard Hooks (Enforcement)
- Can block tool execution
- Enforce safety rules
- Require explicit approval

---

## Why This Works

### Hooks vs Skills

| Aspect | Skills | Hooks |
|--------|--------|-------|
| **When** | Agent chooses to load | Automatic at events |
| **Type** | Guidance format | Context + enforcement |
| **Soft** | Can ignore | Inject context |
| **Hard** | N/A | Can block execution |
| **Reliability** | ~30% invocation | Always runs |

### The Feedback Loop

```
Agent works
     │
     ▼
Hook injects reminder ──► Agent receives context
     │
     ▼
Skill available ──► Agent uses skill correctly
     │
     ▼
Task completes
```

---

## Comparison with MCP

| Aspect | CSH Framework | MCP |
|--------|--------------|-----|
| **Architecture** | CLI + files (Unix-like) | Server-based |
| **Complexity** | Simple, portable | Requires server |
| **Token Efficiency** | Progressive disclosure | All tools loaded |
| **Context** | Hooks inject reminders | Limited |
| **Agent Freedom** | High (skills are suggestions) | Schema-bound |
| **Setup** | Just CLI tools | Server infrastructure |

---

## Research: Why Agents Fail at Skills

### The Mode-Switching Problem

1. **Pattern Matching** (works): Agents recognize when a skill is relevant (~92%)
2. **Mode Switching** (fails): Agents actually invoke and use the skill (~30%)

### Root Cause

Current LLMs are trained on pattern matching, not tool orchestration. They excel at:
- Understanding what a skill does
- Recognizing when to use it

But struggle with:
- Actually invoking the skill
- Following the workflow correctly
- Handling edge cases

### How CSH Helps

**Hooks** inject context to nudge agents toward correct skill usage:
- At session start: Remind what skills exist
- Before tasks: Suggest relevant skills with **mandatory language**
- After errors: Offer retry patterns

**Why Mandatory Language?**
- AI agents are lazy about reading skills (~30% invocation rate)
- "Consider using X" → ignored
- "YOU MUST use X skill" → higher compliance
- Hooks use strong directives to counter this

---

## When to Use CSH Framework

### Best For
- **Context-aware agents**: Where session memory matters
- **Workflow guidance**: Where agents need reminders
- **Unix philosophy**: Simple, portable, CLI-based
- **Privacy-sensitive**: No server required, data stays local

### Not For
- **Simple one-off tools**: Overhead not worth it
- **Enterprise MCP compliance**: Use MCP if required

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

*Next: [Three-Layer Architecture](./02-architecture.md)*
