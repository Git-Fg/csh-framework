---
title: "CSH Framework Documentation"
summary: "Complete documentation for the CSH (CLI + Skills + Hooks) Framework - core concepts, platform guides, testing."
read_when:
  - You are new to CSH Framework
  - You want to find specific documentation
  - You need to understand the structure
---

# CSH Framework Documentation

Complete documentation for the CSH (CLI + Skills + Hooks) Framework.

---

## Start Here

| Guide | Purpose |
|-------|---------|
| **[USER_README.md](../USER_README.md)** | Complete human overview — start here |
| **[Overview](./core/overview.md)** | High-level introduction to CSH Framework |
| **[Quickstart](./core/quickstart.md)** | Get up and running in 5 minutes |
| **[Philosophy](./core/philosophy.md)** | Why CSH exists, the core problem |

---

## Documentation Structure

### Core (Framework-Agnostic)
- [Overview](./core/overview.md) — What is CSH Framework?
- [Quickstart](./core/quickstart.md) — Get up and running in 5 minutes
- [Philosophy](./core/philosophy.md) — Design principles and core problem
- [Architecture](./core/architecture.md) — Three-layer architecture visual
- [Development](./core/development.md) — CLI tool patterns
- [Skills](./core/skills.md) — Skill implementation
- [Hooks](./core/hooks.md) — Hook patterns & orchestration
- [Cross-Platform](./core/cross-platform.md) — Multi-platform skills
- [Best Practices](./core/best-practices.md) — Skill descriptions + testing validation
- [Troubleshooting](./core/troubleshooting.md) — Common issues & solutions
- [CLI-First Architecture](./core/cli-first-architecture.md) — Build CLI then wrap with plugins
- [Cognitive Load](./core/cognitive-load-optimization.md) — 5-10-5 command pattern
- [Project Structure](./core/project-structure.md) — Monorepo organization

### OpenClaw
- [Overview](./openclaw/overview.md) — OpenClaw integration overview
- [Skills](./openclaw/skills.md) — OpenClaw-specific skill discovery/loading
- [Hooks](./openclaw/hooks.md) — OpenClaw hook system (handler.ts, events)
- [Testing](./openclaw/testing.md) — OpenClaw testing guide
- [Reference](./openclaw/reference.md) — OpenClaw CLI commands
- [Hook Events](./openclaw/hook-events.md) — Complete hook event reference
- [Platform Comparison](./openclaw/platform-comparison.md) — Platform decision matrix
- [Skill Format](./openclaw/skill-format.md) — SKILL.md specification
- [CLI Reference](./openclaw/cli-reference.md) — CLI design principles
- [Terminology](./openclaw/terminology.md) — Glossary of terms
- [Exit Codes](./openclaw/exit-codes.md) — Standard exit codes

### Claude Code
- [Overview](./claudecode/overview.md) — Claude Code integration overview
- [Skills](./claudecode/skills.md) — Claude Code skill system
- [Hooks](./claudecode/hooks.md) — Claude Code hooks (14 events, types)
- [CLI Reference](./claudecode/cli-reference.md) — Plugin commands
- [Testing](./claudecode/testing.md) — Plugin testing guide
- [Real Examples](./claudecode/examples.md) — ClawVault reference implementation

### Testing (OpenClaw-focused)
- [Behavioral Testing Approach](./openclaw/behavioral-testing-approach.md) — Agent behavior validation methodology
- [Claude Code Testbed](./openclaw/claude-code-testbed.md) — Using Claude Code as live development testbed
- [Testing Workflows](./openclaw/test-workflows.md) — Practical testing workflows and patterns
- [Test Cases & Templates](./openclaw/test-cases-templates.md) — Ready-to-use behavioral test cases
- [Testing Guide](./openclaw/testing-guide.md) — Comprehensive testing guide

*Note: The testing framework focuses on **agent behavioral validation** using Claude Code as a controlled testbed, not code-level unit testing.*

---

## Key Topics

### By Interest
- **New to CSH?** Start with [Overview](./core/overview.md) → [Quickstart](./core/quickstart.md)
- **Building CLI Tools?** [Development Patterns](./core/development.md) + [CLI Reference](./openclaw/cli-reference.md)
- **Creating Skills?** [Skill Implementation](./core/skills.md) + [Skill Format](./openclaw/skill-format.md)
- **Implementing Hooks?** [Hook Patterns](./core/hooks.md) + [Hook Events](./openclaw/hook-events.md)
- **OpenClaw Integration?** [OpenClaw Overview](./openclaw/overview.md) + [Skills](./openclaw/skills.md) + [Hooks](./openclaw/hooks.md)
- **Claude Code Integration?** [Claude Code Overview](./claudecode/overview.md) + [Skills](./claudecode/skills.md) + [Hooks](./claudecode/hooks.md)
- **Choosing a Platform?** [Platform Comparison](./openclaw/platform-comparison.md)
- **Troubleshooting?** [Troubleshooting Guide](./core/troubleshooting.md)

### By Platform
- **OpenClaw**: [OpenClaw Integration](./openclaw/)
- **Claude Code**: [Claude Code Integration](./claudecode/)

---

## Structure Overview

```
docs/
├── core/         # Framework-agnostic concepts
├── openclaw/     # OpenClaw-specific integration
├── claudecode/   # Claude Code-specific integration
└── archive/      # Old documentation (pre-refactor)
```

**Key Principles**:
- No subfolders - all docs one level deep
- Clear separation: Core concepts vs platform-specific
- No information lost from original structure

---

## Recent Updates

**2026-02-25**: Documentation reorganized into 3 folders
- Reorganized from numbered subfolders to: core/, openclaw/, claudecode/
- No subfolders - all docs one level deep
- Testing docs moved to openclaw/ (gateway-focused)
- Reference docs distributed to appropriate platform folders

**2026-02-25**: New architecture and pattern docs
- Added CLI-First Architecture - Build CLI then wrap with plugins
- Added Cognitive Load Optimization - 5-10-5 command pattern
- Added Project Structure - Monorepo organization
- Added Exit Codes - Standard exit codes
- Updated Hooks - Added mandatory language guidelines

**2026-02-24**: Testing framework completely reimagined
- **Removed**: Code-centric testing (unit tests, integration scripts, CI/CD)
- **Added**: Behavioral testing methodology using Claude Code as testbed
- **Rationale**: The most valuable testing is observing agent behavior, not code coverage

---

## Reference Implementation

**ClawVault** is a production-ready example of CSH Framework in action:
- **Location**: `/root/clawvault/`
- **CLI**: 40+ commands (memory, session, task, project, config, etc.)
- **Skills**: 10+ skills for memory, learning, planning
- **Plugins**: OpenClaude Code and Claude Code extensions
- **Pattern**: CLI-first → Skills wrapping → Hooks for injection

Use ClawVault as a reference when implementing CSH patterns.

---

## Quick Links

**For Developers**:
- [Create Your First Skill](./core/quickstart.md)
- [Design CLI Tools](./core/development.md)
- [Implement Hooks](./core/hooks.md)

**For Integrators**:
- [OpenClaw Integration](./openclaw/overview.md)
- [Claude Code Integration](./claudecode/overview.md)
- [Platform Decision](./openclaw/platform-comparison.md)

**For Troubleshooting**:
- [Common Issues](./core/troubleshooting.md)
- [Best Practices](./core/best-practices.md)
- [Hook Events](./openclaw/hook-events.md)
