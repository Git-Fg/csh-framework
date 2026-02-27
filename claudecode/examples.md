---
title: "Real Examples: ClawVault"
summary: "Reference implementation of CSH Framework patterns in ClawVault - CLI structure, skills, and plugins."
read_when:
  - You want to see CSH patterns in action
  - You need a working plugin example
  - You are designing your own CLI + skills + hooks
---

# Real Examples: ClawVault

ClawVault (`/root/clawvault/`) is a production-ready implementation of CSH Framework patterns.

---

## Project Structure

```
clawvault/
├── bin/                          # CLI entry points
│   ├── clawvault.js             # Main CLI
│   └── register-*.js            # Command registration
├── src/                         # TypeScript source
│   ├── commands/                # Command implementations
│   └── lib/                    # Core library
├── extension-openclaw/          # OpenClaw plugin
│   ├── skills/                 # Bundled skills
│   ├── hooks/                  # Hook handlers
│   └── plugin.json             # Manifest
├── extension-claudecode/        # Claude Code plugin
│   ├── skills/                 # Bundled skills
│   ├── hooks/                  # Hook handlers
│   └── plugin.json             # Manifest
└── package.json
```

---

## CLI Structure (5-10 Pattern)

```bash
# Top-level commands (9 total)
clawvault --help

# Core commands
clawvault remember <type> <content>  # Create memory
clawvault search <query>             # Search memories
clawvault get <id>                   # Get memory by ID
clawvault list                       # List memories

# Session commands
clawvault wake                       # Start session
clawvault sleep                      # End session
clawvault checkpoint                 # Save state

# Task commands
clawvault task <name>                # Create task
clawvault backlog                    # List backlog

# Config commands
clawvault config get <key>
clawvault config set <key> <value>
```

---

## Skill Example: Memory Remember

Location: `/root/clawvault/extension-openclaw/skills/remember/SKILL.md`

```markdown
---
name: clawvault-remember
description: Remember important information with rich metadata

## When to use
- User says "remember", "note", "save"
- Capturing decisions, lessons, facts

## Prerequisites
- ClawVault CLI installed
- OpenClaw plugin installed

## Workflow
1. Parse user input for memory content
2. Determine memory type (fact, decision, lesson, etc.)
3. Use CLI to store: clawvault remember <type> "<content>"
4. Confirm storage location

## Success Criteria
- Memory stored in correct location
- Metadata captured (type, timestamp)
- Confirmation shown to user
```

---

## Hook Example: Session Start

Location: `/root/clawvault/extension-openclaw/hooks/session-start.sh`

```bash
#!/bin/bash
# Injects skill menu at session start

read -r event

# Output skill guidance for agent
echo "Available skills:"
echo "- clawvault-remember: when user says 'remember'"
echo "- clawvault-search: when searching past context"
echo ""
echo "PROACTIVELY use clawvault-search before reading files."
```

---

## Plugin Manifest

Location: `/root/clawvault/extension-openclaw/plugin.json`

```json
{
  "name": "clawvault-extension",
  "version": "3.0.0",
  "description": "ClawVault memory system for OpenClaw",
  "skills": ["./skills"],
  "hooks": ["./hooks/hooks.json"]
}
```

---

## Hooks Config

Location: `/root/clawvault/extension-openclaw/hooks/hooks.json`

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/session-start.sh"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/pre-read.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Key Patterns Demonstrated

| Pattern | How ClawVault Does It |
|---------|----------------------|
| **5-10 Commands** | 9 top-level commands with subcommands |
| **Hub-and-Spoke** | Main skill routes to sub-skills |
| **Progressive Disclosure** | Minimal skill loading, expand on use |
| **CLI First** | 40+ CLI commands before plugins |
| **Hook Automation** | Session start, pre-tool, post-tool |
| **Self-Contained** | No workspace config needed |

---

## Using as Reference

1. **CLI Design**: Study `bin/clawvault.js` for command structure
2. **Skills**: Study `extension-openclaw/skills/` for SKILL.md format
3. **Hooks**: Study `extension-openclaw/hooks/` for hook handlers
4. **Plugins**: Study `extension-openclaw/plugin.json` for manifest

---

## References

- [Skills Implementation](../core/skills.md)
- [Hook Patterns](../core/hooks.md)
- [Best Practices](../core/best-practices.md)
- [ClawVault Repo](https://github.com/example/clawvault)
