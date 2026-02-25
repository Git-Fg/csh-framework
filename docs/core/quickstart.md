---
title: "Quickstart Guide"
summary: "Get started with the CSH Framework methodology in 5 minutes."
read_when:
  - You want to create your first CSH plugin
  - You need a step-by-step tutorial
  - You are evaluating CSH for your project
---

# Quickstart Guide

Get started with the CSH Framework methodology in 5 minutes.

---

## Step 1: Initialize Your Plugin Structure

Since CSH is a "zero-touch" methodology, you don't need to install a specific "CSH CLI." Instead, you create the directory structure that agent platforms like Claude Code and OpenClaw expect for plugins.

```bash
mkdir -p my-plugin/skills
mkdir -p my-plugin/hooks
```

This establishes the baseline:

```
my-plugin/
├── skills/            # Folder for your SKILL.md files
├── hooks/             # Folder for your hook handlers
└── openclaw.plugin.json # Manifest for OpenClaw
```

---

## Step 2: Create Your First Skill

Create a directory for your skill and add a `SKILL.md` file:

```bash
mkdir -p my-plugin/skills/hello-world
touch my-plugin/skills/hello-world/SKILL.md
```

Edit `my-plugin/skills/hello-world/SKILL.md`:

```yaml
---
name: hello-world
description: Prints a greeting. Use when you need to test the setup.
---

# Hello World Skill

A simple skill that demonstrates the CSH pattern.

## Workflow

1. Run: `echo "Hello from my portable plugin!"`
```

---

## Step 3: Test Your Plugin

### For OpenClaw

```bash
# Standard install
openclaw plugins install ./my-plugin

# For development/iteration (link mode for hot reload)
openclaw plugins install -l ./my-plugin
```

### For Claude Code

```bash
# Install the local plugin directory
claude plugin install ./my-plugin
```

---

## Step 4: Add a Hook (Optional)

Create `hooks/start.sh`:

```bash
#!/bin/bash
echo "Session starting at $(date)"
```

Add to `openclaw.plugin.json` (OpenClaw):

```json
{
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/start.sh"
    }
  ]
}
```

Or `.claude-plugin/plugin.json` (Claude Code):

```json
{
  "hooks": {
    "events": {
      "SessionStart": {
        "command": "./hooks/start.sh"
      }
    }
  }
}
```

---

## What Next?

| Goal | Doc |
|------|-----|
| More skill patterns | [Skill Patterns](../patterns/02-skills.md) |
| Hook automation | [Hooks](../patterns/03-hooks.md) |
| CLI design | [Development](../patterns/01-development.md) |
| Platform specifics | [Claude Code](../platforms/01-claude-code.md), [OpenClaw](../platforms/02-openclaw.md) |

---

## Troubleshooting

**Skill not loading?**
- Check file is named `SKILL.md` (uppercase)
- Verify YAML frontmatter has `---` delimiters
- Check skill is in a subdirectory of `skills/`

**Hook not firing?**
- Verify event name in manifest
- Ensure handler script is executable: `chmod +x hooks/*.sh`
- Check gateway logs

---

*Ready to build? See [Plugin Development](./02-plugin-development.md)*
