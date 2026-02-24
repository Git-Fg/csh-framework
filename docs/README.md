# CSH Framework Documentation

**The progressive, zero-touch extension framework for terminal-native AI agents.**

---

## What is the CSH Framework?

The CSH Framework (CLI-Skills-Hooks) provides a architecture for building AI agent extensions that:

- **Load on demand** via progressive disclosure
- **Execute anywhere** using CLI primitives agents already understand
- **Automate transparently** through hooks without agent intervention

### Core Philosophy

| Layer | Purpose | Example |
|-------|---------|---------|
| **CLI Primitives** | Single, focused operations | `git`, `curl`, `jq` |
| **Skills** | High-level workflows | "resilient-web-scrape" |
| **Hooks** | Automation & guardrails | Session lifecycle |

---

## Documentation Structure

### Core Concepts
- [Philosophy](./core/01-philosophy.md) — Why CLI + Skills + Hooks
- [Architecture](./core/02-architecture.md) — Three-layer architecture
- [Trade-offs](./core/03-tradeoffs.md) — Honest assessment vs alternatives

### Platform Guides
- [Claude Code](./platforms/01-claude-code.md) — Integration with Claude Code
- [OpenClaw](./platforms/02-openclaw.md) — Integration with OpenClaw
- [Comparison](./platforms/03-comparison.md) — Platform comparison matrix

### Patterns & Guides
- [Development](./patterns/01-development.md) — CLI tool design, 5-10-5 pattern
- [Skills](./patterns/02-skills.md) — Skill implementation patterns
- [Hooks](./patterns/03-hooks.md) — Hook automation patterns
- [Cross-Platform](./patterns/04-cross-platform.md) — Cross-platform skills

### Reference
- [CLI Reference](./reference/01-cli-reference.md) — Framework CLI commands
- [Skill Format](./reference/02-skill-format.md) — SKILL.md specification
- [Hook Events](./reference/03-hook-events.md) — Available hook events

### Guides
- [Quickstart](./guides/01-quickstart.md) — Getting started
- [Plugin Development](./guides/02-plugin-development.md) — Creating plugins

---

## Quick Start

**Step 1: Install the framework**
```bash
# Platform-specific installation
npm install -g csh-cli    # or your framework's package
```

**Step 2: Initialize a plugin**
```bash
csh init my-plugin
cd my-plugin
```

**Step 3: Create your first skill**
```markdown
# SKILL.md
name: my-skill
description: Does something useful

When I need to do X, use this approach:
1. Run `csh do-x`
2. Parse the JSON output
3. Handle errors with retry logic
```

---

## Best Practices

1. **5-10-5 Pattern**: Aim for 5-10 command families with matching skills
2. **Progressive Disclosure**: Return minimal summaries by default, verbose on request
3. **Hooks for Safety**: Use hooks for guardrails that always apply
4. **Self-Documenting CLI**: Every command supports `--help`

---

## Research Validation

This architecture is based on research showing:
- **87-98% token reduction** via progressive disclosure (Hive Research)
- **Unlimited tool scalability** via filesystem-based skill discovery
- **Maximum agent autonomy** via CLI primitives

---

## Why This Approach?

Traditional MCP loads all tools into context (50,000+ tokens). The CSH Framework uses:

- **Progressive Disclosure**: Load only what's needed when it's needed
- **CLI as Contract**: Leverage tools agents already understand
- **Hooks as Safety Net**: Automate guardrails without restricting freedom

---

*Learn more: [Architecture Overview](./core/02-architecture.md)*
