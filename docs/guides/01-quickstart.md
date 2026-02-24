# Quickstart Guide

Get started with the CSH Framework in 5 minutes.

---

## Prerequisites

- Node.js 18+ or Python 3.9+
- A terminal
- Access to Claude Code or OpenClaw

---

## Step 1: Install the Framework

```bash
# Option A: npm (if framework provides a CLI)
npm install -g csh-cli

# Option B: Clone and use directly
git clone https://github.com/your-org/csh-framework.git
cd csh-framework
```

---

## Step 2: Initialize a Plugin

```bash
# Create your first plugin
csh init my-first-plugin
cd my-first-plugin
```

This creates:

```
my-first-plugin/
├── SKILL.md           # Your skill definition
├── openclaw.plugin.json  # Plugin manifest (for OpenClaw)
└── hooks/             # Hook handlers (optional)
    └── start.sh
```

---

## Step 3: Create Your First Skill

Edit `SKILL.md`:

```yaml
---
name: hello-world
description: Prints a greeting. Use when you need to test the setup.
---

# Hello World Skill

A simple skill that demonstrates the basics.

## When to Use

- Testing plugin installation
- Learning the skill format

## Workflow

1. Run: `echo "Hello from my plugin!"`
2. Verify output contains greeting

## Example

```bash
echo "Hello from my plugin!"
```
```

---

## Step 4: Test Your Skill

### For OpenClaw

```bash
# Copy plugin to OpenClaw plugins directory
cp -r my-first-plugin ~/.openclaw/plugins/

# Restart gateway
openclaw gateway restart

# Use skill in session
/skill read hello-world
```

### For Claude Code

```bash
# Copy skill to Claude skills directory
cp -r my-first-plugin ~/.claude/skills/

# Reload (if needed)
# Skills auto-discover on session start
```

---

## Step 5: Add a Hook (Optional)

Create `hooks/start.sh`:

```bash
#!/bin/bash
echo "Session starting at $(date)"
```

Add to manifest:

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
- Check skill is in correct directory

**Hook not firing?**
- Verify event name in manifest
- Ensure handler script is executable: `chmod +x hooks/*.sh`
- Check gateway logs

---

*Ready to build? See [Plugin Development](./02-plugin-development.md)*
