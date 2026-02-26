---
title: "Terminology Glossary"
summary: "Common terms and concepts used throughout the CSH Framework documentation - CLI Primitives, Skills, Hooks, and more."
read_when:
  - You are new to CSH Framework
  - You need to understand terminology
  - You want to read consistent documentation
---

# Terminology Glossary

Common terms and concepts used throughout the CSH Framework documentation.

---

## Core Concepts

### CLI Primitives (C1)
The base layer of CSH Framework consisting of executable command-line tools that perform actual operations.

**Example**: `mytool list` returns clear, readable output.

**See Also**: [Development Patterns](../core/development.md)

### Skills (C2)
Agent guidance files (SKILL.md) that teach agents when and how to use CLI tools. Skills don't contain tool commands—they contain workflow guidance.

**Example**: A skill might guide an agent to "use `memory-search` when the user asks about past decisions."

**See Also**: [Skill Implementation](../core/skills.md)

### Hooks (C3)
Event-driven automation and context injection system. Hooks perform side effects (automation) and inject guidance (context) at specific events.

**Example**: A `session_start` hook might load project context and remind the agent about available skills.

**See Also**: [Hook Patterns](../core/hooks.md)

---

## Framework Architecture

### CSH Framework
The **CLI + Skills + Hooks** Framework: a portable, composable architecture for building AI agent systems.

**Components**: CLI Tools (C1), Skills (C2), Hooks (C3)

**Purpose**: Enable agents to discover, understand, and effectively use CLI tools.

### Three-Layer Architecture
The structural organization of CSH Framework:
1. **Layer 1 (C1)**: CLI Primitives - Execute commands
2. **Layer 2 (C2)**: Skills - Guide usage
3. **Layer 3 (C3)**: Hooks - Automate + inject context

### Mode-Switching Problem
The tendency of AI agents to recognize when a tool is relevant (~92% accuracy) but fail to actually invoke it (~30% rate).

**Solution**: Hooks inject guidance at key moments to overcome this resistance.

---

## Plugin Concepts

### Plugin
A packaged bundle containing skills, hooks, and optional native tooling for a specific platform.

**Structure**:
- OpenClaw: `openclaw.plugin.json`
- Claude Code: `.claude-plugin/plugin.json`

### Manifest
Configuration file defining plugin metadata, skills, and hooks.

**Formats**:
- OpenClaw: `openclaw.plugin.json`
- Claude Code: `.claude-plugin/plugin.json`

### Plugin-Bundled Skills
Skills shipped within a plugin directory (`<plugin>/skills/<name>/SKILL.md`).

**Contrast**: Global skills in `~/.openclaw/skills/` or `~/.claude/skills/`.

---

## Skill Concepts

### SKILL.md Format
The markdown-based skill file format with YAML frontmatter.

**Structure**:
```yaml
---
name: skill-name
description: Brief description
---

# Skill Content
Markdown instructions...
```

### YAML Frontmatter
Metadata at the top of SKILL.md files.

**Required Fields**: `name`, `description`
**Optional Fields**: `version`, `author`, `tags`, `metadata`

### Skill Discovery
The process by which agents learn about available skills.

**OpenClaw**: Dynamic with feature gating
**Claude Code**: Automatic directory scanning

### Skill Gating
Conditional loading of skills based on runtime requirements.

**Requirements**: `bins` (binaries), `env` (environment variables), `os` (operating system)

### Skill Orchestration
Coordinating multiple skills to accomplish complex tasks.

**Pattern**: Hooks guide agents to use the right skills at the right time.

---

## Hook Concepts

### Hook Event
A specific moment in the session lifecycle or system operation that triggers hooks.

**Examples**: `session_start`, `command:new`, `webhook`, `cron`

### Hook Handler
Executable code that responds to hook events.

**Types**:
- **Shell Script**: Bash/sh scripts
- **TypeScript**: Node.js handlers (OpenClaw)
- **Prompt Hook**: LLM evaluation (Claude Code)
- **Agent Hook**: Sub-agent verification (Claude Code)

### Hook Automation
Side effects performed by hooks without returning text to the agent.

**Examples**: Logging, cleanup, reindexing, file operations.

### Hook Injection
Text returned by hooks that is injected into the agent's context.

**Purpose**: Remind agents about available skills, warn about constraints, provide context.

### Automation + Elicitation Pattern
The dual purpose of hooks:
1. **Automate**: Perform technical tasks silently
2. **Elicitate**: Inject guidance to trigger correct skill usage

---

## Platform-Specific Terms

### OpenClaw
Multi-channel AI agent gateway with native plugin support.

**Features**: Persistent sessions, cron/webhook automation, multi-platform support.

### Claude Code
Desktop AI coding assistant by Anthropic with plugin system.

**Features**: Project-based work, integrated editor, local execution.

### Native Plugin
Plugin implementation using platform's native API.

**OpenClaw**: `openclaw.plugin.json` with native tool registration
**Claude Code**: `.claude-plugin/plugin.json` with hooks

### Link Mode (OpenClaw)
Development installation mode that enables hot reload.

**Command**: `openclaw plugins install -l /path/to/plugin`

### Gateway (OpenClaw)
Background service that manages AI agents, sessions, and plugins.

**Status Commands**: `openclaw gateway status`, `openclaw health`

---

## Testing & Validation

### Independent Verification
The practice of not trusting agent claims about skill/tool usage, but verifying independently.

**Methods**: Check logs, verify state, examine files, test commands.

### False Positive
Incorrect reporting of success when failure actually occurred.

**Common**: Agent claims "skill was used" but it wasn't.

### Validation Gap
Missing verification steps in testing that allow false positives to go undetected.

**Prevention**: Always verify independently, check logs, test edge cases.

---

## Development Concepts

### Zero-Transformation Output
Principle that CLI tools should return data exactly as-is, not wrapped in JSON unless requested.

**Benefit**: Gives agents flexibility to process output as needed.

### Self-Teaching Documentation
CLI tools providing complete documentation via `--help` that agents can read to learn capabilities.

**Principle**: Don't embed commands in skills—embed them in tool help documentation.

### Progressive Disclosure
Loading minimal information first (metadata), then full content only when needed.

**Benefit**: Reduces token consumption, improves performance.

### Semantic Density
Prioritizing concise text over verbose JSON outputs for better agent understanding.

**Principle**: Text is often better for agents than structured data.

---

## Common Acronyms

| Acronym | Full Term | Description |
|----------|-------------|-------------|
| CSH | CLI + Skills + Hooks | The CSH Framework architecture |
| CLI | Command Line Interface | Executable command-line tools |
| LLM | Large Language Model | AI model powering agents |
| API | Application Programming Interface | Interface for software communication |
| JSON | JavaScript Object Notation | Structured data format |
| YAML | YAML Ain't Markup Language | Configuration file format |
| MCP | Model Context Protocol | Tool server protocol |

---

## Cross-Reference

By Topic:
- **CLI Development**: [Development Patterns](../core/development.md)
- **Skills**: [Skill Implementation](../core/skills.md)
- **Hooks**: [Hook Patterns](../core/hooks.md)
- **OpenClaw**: [OpenClaw Integration](./overview.md)
- **Claude Code**: [Claude Code Integration](../claudecode/overview.md)

By Platform:
- **OpenClaw Skills**: [OpenClaw Skills](./skills.md)
- **OpenClaw Hooks**: [OpenClaw Hooks](./hooks.md)
- **Claude Code Skills**: [Claude Code Skills](../claudecode/skills.md)
- **Claude Code Hooks**: [Claude Code Hooks](../claudecode/hooks.md)

---

*Created by: Claw 🦞*
*Date: 2026-02-24*
