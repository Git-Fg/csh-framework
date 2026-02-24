# Plugin Development Guide

Create reusable plugins that work with Claude Code and OpenClaw.

---

## What is a Plugin?

A plugin bundles:
- **Skills** — Workflow definitions
- **Hooks** — Automation handlers
- **Configuration** — Manifest and metadata

---

## Plugin Structure

```
my-plugin/
├── SKILL.md              # Skill definition
├── openclaw.plugin.json  # OpenClaw manifest
├── .claude-plugin/      # Claude Code manifest
│   └── plugin.json
├── hooks/               # Hook handlers
│   ├── start.sh
│   └── before-command.sh
└── skills/              # Additional skills (optional)
    └── advanced/
        └── SKILL.md
```

---

## Manifest Formats

### OpenClaw (`openclaw.plugin.json`)

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "main": "./index.js",
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/start.sh",
      "description": "Run on session start"
    }
  ],
  "skills": ["./skills"]
}
```

### Claude Code (`.claude-plugin/plugin.json`)

```json
{
  "id": "my-plugin",
  "version": "1.0.0",
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

## Development Workflow

### 1. Create Plugin Structure

```bash
mkdir -p my-plugin/hooks
touch my-plugin/SKILL.md
```

### 2. Define Manifest

Choose your target platform(s) and create appropriate manifest(s).

### 3. Write Skills

Create `SKILL.md` following the [format specification](../reference/02-skill-format.md).

### 4. Add Hooks (Optional)

Create handler scripts:

```bash
#!/bin/bash
# hooks/start.sh
echo "Plugin initialized"
```

Make executable: `chmod +x hooks/*.sh`

### 5. Test Locally

```bash
# For OpenClaw
cp -r my-plugin ~/.openclaw/plugins/
openclaw gateway restart

# For Claude Code
cp -r my-plugin ~/.claude/skills/
```

### 6. Publish

```bash
# Package for npm
npm init
npm publish

# Or share as git repo
git init
git add .
git commit -m "Initial plugin"
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Single responsibility | Easier to maintain and test |
| Version semantically | Track changes clearly |
| Document prerequisites | Skills should state required tools |
| Handle errors gracefully | Hooks shouldn't crash sessions |
| Keep hooks lightweight | Fast execution, no blocking |

---

## Common Patterns

### Skill-Hook Bridge

Use hooks to prepare context that skills consume:

```
Hook: session:start → writes env vars
Skill: reads env vars → uses in workflow
```

### Context Injection

```bash
# hooks/session-start.sh
#!/bin/bash

echo "MyPlugin: Available skills"
echo "- Use 'query' for database operations"
echo "- Use 'migrate' for schema changes"
```

### Context Injection

```bash
# hooks/session-start.sh
#!/bin/bash
cat > /tmp/plugin-context.json << EOF
{
  "project": "my-app",
  "environment": "development"
}
EOF
```

---

## Testing

### Unit Test Hooks

```bash
# Test hook script
echo '{"command": "test"}' | ./hooks/before-command.sh
```

### Integration Test

```bash
# Deploy and test in session
openclaw agent -m "Use the hello-world skill" --local
```

---

## Publishing

### npm Package

```json
{
  "name": "csh-plugin-my-plugin",
  "version": "1.0.0",
  "csh": {
    "plugins": ["./"]
  }
}
```

### GitHub

Tag releases for version tracking:

```bash
git tag v1.0.0
git push --tags
```

---

## Related Docs

- [SKILL.md Format](../reference/02-skill-format.md)
- [Hook Events](../reference/03-hook-events.md)
- [Cross-Platform Skills](../patterns/04-cross-platform.md)
