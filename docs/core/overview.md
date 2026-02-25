---
title: "CSH Framework Overview"
summary: "The CSH (CLI + Skills + Hooks) Framework is a portable architecture for building AI agent systems that can discover, understand, and effectively use CLI tools."
read_when:
  - You are new to CSH Framework
  - You want to understand the high-level architecture
  - You need to decide if CSH is right for your project
---

# CSH Framework - Overview

The CSH (CLI + Skills + Hooks) Framework is a portable, composable architecture for building AI agent systems that can discover, understand, and effectively use CLI tools.

---

## What is CSH?

CSH Framework provides three core primitives that work together:

| Primitive | Purpose | Example |
|-----------|---------|---------|
| **CLI Tools** | Execute commands and return natural output | `mytool --help` |
| **Skills** | Guide agents on when/how to use tools | SKILL.md with workflows |
| **Hooks** | Automate tasks and inject context | Event-driven scripts |

### The Core Problem

AI agents struggle with two critical issues:

1. **Discovery**: Finding the right tool for the task
2. **Invocation**: Actually using that tool (mode-switching resistance)

CSH Framework solves both problems through progressive disclosure and contextual reinforcement.

---

## Key Principles

### 1. Zero-Transformation Output
CLI tools return data exactly as-is, not wrapped in JSON unless requested. This gives agents flexibility to process output as needed.

### 2. Self-Teaching Documentation
Every tool provides `--help` with complete documentation. Agents learn tool capabilities by reading help output.

### 3. Context-Aware Skills
Skills don't contain tool commands—they contain guidance on when to use tools. The actual commands live in help documentation.

### 4. Composable Hooks
Hooks automate background tasks AND inject guidance at key moments, ensuring agents use the right tools at the right time.

### 5. Self-Contained Plugins
Everything (skills, hooks, logic) lives in the plugin. Users install and it just works—no editing CLAUDE.md, AGENTS.md, or workspace instructions.

---

## Self-Contained: No Manual Config Required

Unlike traditional plugins, **CSH plugins are fully self-contained**:

| Traditional Plugins | CSH Plugins |
|--------------------|--------------|
| Requires editing CLAUDE.md | ❌ No editing needed |
| Requires workspace hooks | ❌ Hooks bundled in plugin |
| Requires AGENTS.md updates | ❌ Soft hooks inject context |
| Manual skill installation | ❌ Skills auto-discovered |

**How it works:**
- **Soft hooks** inject context at session start, tool calls, etc.
- **Hard hooks** can enforce safety rules (block execution if needed)
- Skills are bundled and loaded when relevant
- No user intervention required after install

**Install command:**
```bash
openclaw plugins install ./my-plugin  # OpenClaw
claude plugin install ./my-plugin     # Claude Code
```

That's it. Plugin self-registers skills and hooks automatically.

---

## Architecture

CSH Framework is a three-layer architecture:

```
Layer 1: CLI Primitives (C1)
  └─> Tools that execute commands (Bash, Exec, Read, Write)

Layer 2: Skills (C2)
  └─> Guidance on when/how to use tools (SKILL.md files)

Layer 3: Hooks (C3)
  └─> Automation + context injection (event-driven)
```

**How It Works:**
1. Agent needs to perform a task
2. Agent discovers available skills
3. Skill guidance tells agent which CLI tool to use
4. Agent reads tool's `--help` to learn commands
5. Agent executes tool via Layer 1 primitive
6. Hook provides context or automation at key events

---

## Platform Support

CSH Framework works with multiple AI agent platforms:

| Platform | Integration | Status |
|----------|-------------|--------|
| **OpenClaw** | Native plugin + skill system | ✅ Production-ready |
| **Claude Code** | Plugin with skills + hooks | ✅ Supported |

All platforms support the same core CSH primitives while providing platform-specific tooling for discovery and management.

---

## When to Use CSH Framework

Use CSH Framework when you need to:

- ✅ Build AI agent tools that are easily discoverable
- ✅ Guide agents through complex workflows
- ✅ Automate tasks without agent intervention
- ✅ Maintain consistent behavior across multiple agents
- ✅ Scale from personal automation to team collaboration

---

## Next Steps

1. **Quick Start**: Get up and running in 5 minutes → [Quickstart Guide](./quickstart.md)
2. **Philosophy**: Understand the design principles → [Philosophy](./philosophy.md)
3. **Core Concepts**: Deep dive into CLI, Skills, and Hooks → [Core Concepts](./)
4. **Platform Integration**: Learn platform-specific details → [OpenClaw](../openclaw/) or [Claude Code](../claudecode/)

---

## Key Concepts at a Glance

| Concept | Description | Learn More |
|----------|-------------|-------------|
| **CLI Tools** | Executable commands with natural output | [Development Patterns](./development.md) |
| **Skills** | Agent guidance files (SKILL.md) | [Skill Implementation](./skills.md) |
| **Hooks** | Event-driven automation + context injection | [Hook Patterns](./hooks.md) |
| **OpenClaw** | Multi-platform gateway with native plugins | [OpenClaw Integration](../openclaw/) |
| **Claude Code** | Desktop AI coding assistant with plugins | [Claude Code Integration](../claudecode/) |
