---
title: "Quickstart Guide"
summary: "Install a CSH plugin in 2 commands - CLI first, then plugin. No manual configuration needed."
read_when:
  - You want to install a CSH plugin quickly
  - You need a working plugin in minutes
  - You are evaluating CSH for your project
---

# Quickstart Guide

Goal: Install a CSH plugin in 2 commands, then it just works. No manual configuration, no editing CLAUDE.md or AGENTS.md.

---

## Prereqs

- An AI agent platform: **OpenClaw** or **Claude Code**

---

## Quick setup

### For Developers (Recommended)

Iterate faster with symlinks and hot reload:

```bash
# 1. Link your CLI locally (for iterative development)
cd /path/to/my-cli && npm link

# 2. Install plugin with -l (creates SYMLINK, not copy!)
openclaw plugins install -l /path/to/my-plugin   # OpenClaw

# 3. Edit code → changes auto-reload (no restart!)
```

### For Users (Production)

```bash
# 1. Install CLI globally
npm install -g clawvault

# 2. Install plugin (copies files)
openclaw plugins install /path/to/my-plugin
openclaw gateway restart
```

---

**Why use `-l` for development?**
- Creates **symlink** → edits take effect immediately
- **Hot reload** → no gateway restart needed
- Test changes instantly → iterate faster

That's it! The plugin is now active with:

- **Skills** → auto-discovered and loaded when relevant
- **Hooks** → inject context automatically (soft hooks)
- **Hard hooks** → enforce safety rules if needed

---

## What you get (no manual work)

| Feature | How it works |
|---------|--------------|
| **Skills** | Bundled in plugin, loaded automatically when relevant |
| **Soft hooks** | Inject context at session start, tool calls, etc. |
| **Hard hooks** | Enforce safety rules (can block execution) |
| **No config** | Self-contained - no workspace instructions needed |

**Unlike other plugins**, CSH plugins don't require you to:
- ❌ Edit CLAUDE.md
- ❌ Edit AGENTS.md
- ❌ Create workspace hooks
- ❌ Configure anything manually

Everything is bundled in the plugin itself.

---

## Verify it works

```bash
# OpenClaw
openclaw skills list | grep my-plugin
openclaw plugins list

# Claude Code
claude plugin list
```

---

## Troubleshooting

**Skill not loading?**
- Verify plugin is installed: `openclaw plugins list`
- Check skill is in `skills/` subdirectory with `SKILL.md`

**Hooks not firing?**
- Check gateway logs: `openclaw logs | grep my-plugin`
- Verify hooks directory exists in plugin

---

## Next Steps

| Goal | Doc |
|------|-----|
| Create new plugin | [Development Patterns](./development.md) |
| Understand skills | [Skills Implementation](./skills.md) |
| Understand hooks | [Hook Patterns](./hooks.md) |
| Platform details | [OpenClaw](../openclaw/overview.md), [Claude Code](../claudecode/overview.md) |
