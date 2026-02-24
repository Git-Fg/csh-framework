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

## Plugin Manifest Details

### OpenClaw Manifest Full Format

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "main": "./index.js",
  "configSchema": {
    "type": "object",
    "properties": {
      "apiKey": {
        "type": "string",
        "description": "API key for external service"
      },
      "enabled": {
        "type": "boolean",
        "default": true
      }
    }
  },
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

### Skill Gating Options

Skills can require specific binaries or environment variables:

```yaml
---
name: my-skill
description: Requires curl and API key
requires:
  bins: ["curl", "jq"]
  env: ["API_KEY"]
---

# My Skill

This skill only loads when curl and jq are available.
```

### Skill Precedence (OpenClaw)

Skills load in this order (later overrides earlier):

1. **Workspace** - `~/.openclaw/workspace/skills/`
2. **Managed** - `~/.openclaw/skills/`
3. **Bundled** - Plugin-provided (lowest priority)

Workspace skills override bundled ones with same name.

---

## Hook Handler Patterns

### CRITICAL: Use require() Inside Handler

Hook handlers run in a different context. Always use `require()` inside the handler, not at module level:

```javascript
// ❌ Wrong - may fail in hook context
const { execFileSync } = require('child_process');
function handler(event) { ... }

// ✅ Correct - require inside handler
function handler(event) {
  const { execFileSync } = require('child_process');
  const result = execFileSync('my-cli', ['arg'], { encoding: 'utf8' });
  return result;
}
```

### JavaScript Handler Example

```javascript
// hooks/my-hook/handler.js
function handler(event) {
  const { execFileSync } = require('child_process');
  const { type, payload } = event;

  switch (type) {
    case 'session_start':
      // Run CLI to get context
      const context = execFileSync('my-cli', ['context'], { encoding: 'utf8' });
      return { instructions: context };

    case 'after_command':
      // Log command usage
      console.log('Command executed:', payload.tool);
      return { continue: true };

    default:
      return { continue: true };
  }
}

module.exports = { handler };
```

### Shell Handler Example

```bash
#!/bin/bash
# Read event from stdin
EVENT=$(jq -r '.type' /dev/stdin)

case "$EVENT" in
  "session_start")
    echo "Session started - inject plugin context"
    ;;
  "after_command")
    TOOL=$(jq -r '.payload.tool' /dev/stdin)
    echo "Used tool: $TOOL"
    ;;
esac
```

---

## CLI-as-Template-Generator Pattern

A key pattern for complex operations: CLI creates templates, agent uses Read/Edit tools.

### Why?

Passing complex data through CLI arguments is error-prone. Better to:
1. CLI scaffolds a template file
2. Agent uses native Edit tool to modify it
3. CLI processes the completed file

### Example: Search Workflow

```bash
# 1. CLI returns file paths (not data)
$ mytool search "database"
[
  {"path": "/vault/decisions/2024-postgres.md", "relevance": 0.95},
  {"path": "/vault/lessons/scaling-lessons.md", "relevance": 0.72}
]

# 2. Agent reads files iteratively with Read tool
# 3. Agent finds the relevant one
```

### Example: Create Workflow

```bash
# 1. CLI creates template, returns path
$ mytool template decision "Choose Postgres"
Created: /vault/decisions/2024-choose-postgres.md

# 2. Agent uses Edit tool to fill in:
# - context
# - decision
# - tradeoffs
# - related

# 3. File is ready
```

### Hook Instructions Template

Inject this at session_start to teach agents the pattern:

```markdown
CRITICAL: USE MYPLUGIN CLI FOR ALL OPERATIONS

## SEARCH WORKFLOW
1. Run: mytool search "<keywords>"
2. Get PATH from output
3. Use Read tool on the PATH
4. If not relevant, read next result - ITERATE

## CREATE WORKFLOW
1. Run: mytool template <type> "<title>"
2. Get PATH from output
3. Use Edit tool to fill in

Remember: CLI creates templates/paths, YOU use Read/Edit tools!
```

---

## Troubleshooting

### Plugin loads but skills don't appear

**Cause:** `skills.allowBundled` set to empty array `[]` in openclaw.json

**Fix:** Remove `allowBundled` entirely (no config = all bundled allowed)

### Hook instructions not followed

**Cause:** Agent follows AGENTS.md instead of hook instructions

**Fix:** Add explicit rule in AGENTS.md, or ensure skills are loaded

### require() fails in handler

**Cause:** Using ESM imports at module level

**Fix:** Use require() inside handler function, not at module level

### Skills manifest wrong format

**Cause:** Listed individual skill paths instead of directory

**Fix:** Use `"skills": ["./skills"]` pointing to directory

---

## Testing Commands

```bash
# Verify plugin loads (OpenClaw)
openclaw plugins list | grep my-plugin

# Verify skills available
openclaw skills list | grep my-plugin

# Reload plugin
openclaw plugins install -l /path/to/my-plugin
openclaw gateway restart

# Test via gateway API
curl -X POST http://127.0.0.1:18789/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Use my-plugin skill"}]}'
```

---

## Related Docs

- [SKILL.md Format](../reference/02-skill-format.md)
- [Hook Events](../reference/03-hook-events.md)
- [Cross-Platform Skills](../patterns/04-cross-platform.md)
