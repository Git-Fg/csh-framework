# CSH Framework Documentation

Complete documentation for the CSH (CLI + Skills + Hooks) Framework.

---

## Start Here

| Guide | Purpose |
|-------|---------|
| **[USER_README.md](../USER_README.md)** | Complete human overview — start here |
| **[Overview](./01-getting-started/01-overview.md)** | High-level introduction to CSH Framework |
| **[Quickstart](./01-getting-started/02-quickstart.md)** | Get up and running in 5 minutes |
| **[Philosophy](./01-getting-started/03-philosophy.md)** | Why CSH exists, the core problem |

---

## Documentation Structure

### 01 - Getting Started
- [Overview](./01-getting-started/01-overview.md) — What is CSH Framework?
- [Quickstart](./01-getting-started/02-quickstart.md) — Get up and running in 5 minutes
- [Philosophy](./01-getting-started/03-philosophy.md) — Design principles and core problem

### 02 - Core Concepts
- [Architecture](./02-core-concepts/01-architecture.md) — Three-layer architecture visual
- [Development](./02-core-concepts/02-development.md) — CLI tool patterns
- [Skills](./02-core-concepts/03-skills.md) — Skill implementation
- [Hooks](./02-core-concepts/04-hooks.md) — Hook patterns & orchestration
- [Cross-Platform](./02-core-concepts/05-cross-platform.md) — Multi-platform skills
- [Best Practices](./02-core-concepts/06-best-practices.md) — Skill descriptions + testing validation
- [Troubleshooting](./02-core-concepts/07-troubleshooting.md) — Common issues & solutions
- [CLI-First Architecture](./02-core-concepts/08-cli-first-architecture.md) — Build CLI then wrap with plugins
- [Cognitive Load](./02-core-concepts/09-cognitive-load-optimization.md) — 5-10-5 command pattern
- [Project Structure](./02-core-concepts/11-project-structure.md) — Monorepo organization

### 03 - OpenClaw
- [Overview](./03-openclaw/01-overview.md) — OpenClaw integration overview
- [Skills](./03-openclaw/02-skills.md) — OpenClaw-specific skill discovery/loading
- [Hooks](./03-openclaw/03-hooks.md) — OpenClaw hook system (handler.ts, events)
- [Testing](./03-openclaw/04-testing.md) — OpenClaw testing guide
- [Reference](./03-openclaw/05-reference.md) — OpenClaw CLI commands

### 04 - Claude Code
- [Overview](./04-claude-code/01-overview.md) — Claude Code integration overview
- [Skills](./04-claude-code/02-skills.md) — Claude Code skill system
- [Hooks](./04-claude-code/03-hooks.md) — Claude Code hooks (17 events, types)
- [Reference](./04-claude-code/04-reference.md) — Claude Code CLI commands

### 05 - Testing Framework
- [Behavioral Testing Approach](./06-testing/01-behavioral-testing-approach.md) — Agent behavior validation methodology
- [Claude Code Testbed](./06-testing/02-claude-code-testbed.md) — Using Claude Code as live development testbed
- [Testing Workflows](./06-testing/03-test-workflows.md) — Practical testing workflows and patterns
- [Test Cases & Templates](./06-testing/04-test-cases-templates.md) — Ready-to-use behavioral test cases

*Note: The testing framework focuses on **agent behavioral validation** using Claude Code as a controlled testbed, not code-level unit testing.*

### 05 - Reference
- [Skill Format](./05-reference/01-skill-format.md) — SKILL.md specification
- [Hook Events](./05-reference/02-hook-events.md) — Complete hook event reference
- [Platform Comparison](./05-reference/03-platform-comparison.md) — Platform decision matrix
- [CLI Reference](./05-reference/04-cli-reference.md) — CLI design principles
- [Terminology](./05-reference/05-terminology.md) — Glossary of terms
- [Exit Codes](./05-reference/06-exit-codes.md) — Standard exit codes

### Examples
- [ClawVault Case Study](../examples/clawvault-case-study.md) — Real-world implementation

---

## Key Topics

### By Interest
- **New to CSH?** Start with [Overview](./01-getting-started/01-overview.md) → [Quickstart](./01-getting-started/02-quickstart.md)
- **Building CLI Tools?** [Development Patterns](./02-core-concepts/02-development.md) + [CLI Reference](./05-reference/04-cli-reference.md)
- **Creating Skills?** [Skill Implementation](./02-core-concepts/03-skills.md) + [Skill Format](./05-reference/01-skill-format.md)
- **Implementing Hooks?** [Hook Patterns](./02-core-concepts/04-hooks.md) + [Hook Events](./05-reference/02-hook-events.md)
- **OpenClaw Integration?** [OpenClaw Overview](./03-openclaw/01-overview.md) + [Skills](./03-openclaw/02-skills.md) + [Hooks](./03-openclaw/03-hooks.md)
- **Claude Code Integration?** [Claude Code Overview](./04-claude-code/01-overview.md) + [Skills](./04-claude-code/02-skills.md) + [Hooks](./04-claude-code/03-hooks.md)
- **Choosing a Platform?** [Platform Comparison](./05-reference/03-platform-comparison.md)
- **Troubleshooting?** [Troubleshooting Guide](./02-core-concepts/07-troubleshooting.md)

### By Platform
- **OpenClaw**: [OpenClaw Integration](./03-openclaw/)
- **Claude Code**: [Claude Code Integration](./04-claude-code/)

---

## Structure Overview

```
docs/
├── 01-getting-started/       # Overview, quickstart, philosophy
├── 02-core-concepts/         # Framework-agnostic concepts
├── 03-openclaw/              # OpenClaw-specific integration
├── 04-claude-code/           # Claude Code-specific integration
├── 05-reference/             # Specs, comparisons, terminology
└── archive/                  # Old documentation (pre-refactor)
```

**Key Principles**:
- One level deep maximum
- Consistent prefix numbering (01-, 02-, 03-)
- Clear separation: Core concepts vs platform-specific
- No information lost from original structure

---

## Recent Updates

**2026-02-25**: New architecture and pattern docs
- Added [CLI-First Architecture](./02-core-concepts/08-cli-first-architecture.md) - Build CLI then wrap with plugins
- Added [Cognitive Load Optimization](./02-core-concepts/09-cognitive-load-optimization.md) - 5-10-5 command pattern
- Added [Project Structure](./02-core-concepts/11-project-structure.md) - Monorepo organization
- Added [Exit Codes](./05-reference/06-exit-codes.md) - Standard exit codes
- Updated [Hooks](./02-core-concepts/04-hooks.md) - Added mandatory language guidelines

**2026-02-24**: Documentation refactored into hierarchical structure
- Separated core concepts from platform-specific content
- Consistent numbering across all documentation
- Clear navigation for cross-platform development
- Added comprehensive troubleshooting guide and terminology glossary

**2026-02-24 (later)**: Testing framework completely reimagined
- **Removed**: Code-centric testing (unit tests, integration scripts, CI/CD)
- **Added**: Behavioral testing methodology using Claude Code as testbed
- **Rationale**: The most valuable testing is observing agent behavior, not code coverage
- **Old documentation**: Archived in `docs/06-testing/archive/` (deprecated)

---

## Quick Links

**For Developers**:
- [Create Your First Skill](./01-getting-started/02-quickstart.md)
- [Design CLI Tools](./02-core-concepts/02-development.md)
- [Implement Hooks](./02-core-concepts/04-hooks.md)

**For Integrators**:
- [OpenClaw Integration](./03-openclaw/01-overview.md)
- [Claude Code Integration](./04-claude-code/01-overview.md)
- [Platform Decision](./05-reference/03-platform-comparison.md)

**For Troubleshooting**:
- [Common Issues](./02-core-concepts/07-troubleshooting.md)
- [Best Practices](./02-core-concepts/06-best-practices.md)
- [Hook Events](./05-reference/02-hook-events.md)
