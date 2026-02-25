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

1. **Quick Start**: Get up and running in 5 minutes → [Quickstart Guide](./02-quickstart.md)
2. **Philosophy**: Understand the design principles → [Philosophy](./03-philosophy.md)
3. **Core Concepts**: Deep dive into CLI, Skills, and Hooks → [Core Concepts](../02-core-concepts/)
4. **Platform Integration**: Learn platform-specific details → [OpenClaw](../03-openclaw/) or [Claude Code](../04-claude-code/)

---

## Key Concepts at a Glance

| Concept | Description | Learn More |
|----------|-------------|-------------|
| **CLI Tools** | Executable commands with natural output | [Development Patterns](../02-core-concepts/02-development.md) |
| **Skills** | Agent guidance files (SKILL.md) | [Skill Implementation](../02-core-concepts/03-skills.md) |
| **Hooks** | Event-driven automation + context injection | [Hook Patterns](../02-core-concepts/04-hooks.md) |
| **OpenClaw** | Multi-platform gateway with native plugins | [OpenClaw Integration](../03-openclaw/) |
| **Claude Code** | Desktop AI coding assistant with plugins | [Claude Code Integration](../04-claude-code/) |
