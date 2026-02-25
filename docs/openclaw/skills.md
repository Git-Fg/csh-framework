---
title: "OpenClaw Skills"
summary: "How skills are discovered, loaded, and managed in OpenClaw - including plugin-bundled skills."
read_when:
  - You are creating skills for OpenClaw
  - You want to understand skill discovery
  - You need to configure skill allowlists
---

# OpenClaw Skills

How skills are discovered, loaded, and managed in OpenClaw.

---

## Skill Discovery Mechanism

OpenClaw uses a dynamic skill discovery system with feature gating.

> **NOTE**: For CSH Framework, skills are typically **plugin-bundled** (shipped with plugins) rather than using the global skill discovery paths below. Plugin skills live in `<plugin>/skills/<name>/SKILL.md` and are referenced via `"skills": ["./skills"]` in the plugin manifest.

### Global Skill Paths (for reference)

OpenClaw scans locations in precedence order:

1. **Workspace skills**: `~/.openclaw/skills/` (or `<workspace>/skills/`)
2. **Managed skills**: `~/.openclaw/skills/`
3. **Bundled skills**: Distributed with the gateway

### Feature Gating

Skills can declare runtime requirements using nested metadata:

```yaml
---
name: my-skill
description: Does something useful
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["curl", "jq"], "env": ["API_KEY"] },
      },
  }
---
```

**Requirements Fields**:
- `bins`: Array of required binary names (must be in PATH)
- `env`: Array of required environment variable names
- `os`: Array of supported operating systems (`linux`, `darwin`, `win32`)

### Metadata-First Loading

OpenClaw loads YAML frontmatter first to optimize prompt density before loading full skill instructions.

**Why This Matters**:
- Reduces token consumption (only metadata in context)
- Faster skill discovery (doesn't need to read full files)
- Progressive disclosure (load full skill only when needed)

---

## Loading Behavior

Skills are discovered and managed via CLI:

```bash
# List all discovered skills
openclaw skills list

# Show skill metadata
openclaw skills show <skill-name>

# Enable a specific skill
openclaw skills enable <skill-name>

# Disable a specific skill
openclaw skills disable <skill-name>
```

### Permission Control

Skills can be explicitly enabled or disabled:

```bash
# Enable skill
openclaw skills enable memory-search

# Disable skill
openclaw skills disable memory-search

# List enabled skills only
openclaw skills list --enabled
```

---

## Skill Manifest (Plugin-Based)

For CSH Framework plugins, skills are defined in the plugin manifest:

```json
{
  "id": "my-csh-plugin",
  "skills": [
    "./skills/memory-search",
    "./skills/memory-remember",
    "./skills/memory-forget"
  ]
}
```

### Directory Structure

```
my-plugin/
├── openclaw.plugin.json
└── skills/
    ├── memory-search/
    │   └── SKILL.md
    ├── memory-remember/
    │   └── SKILL.md
    └── memory-forget/
        └── SKILL.md
```

---

## Skill Requirements Validation

OpenClaw validates skill requirements before making them available to agents:

### Binary Validation

Checks if required binaries are installed:

```yaml
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["curl", "jq", "git"] },
      },
  }
```

**Validation**:
```bash
# Internally, OpenClaw runs:
which curl && which jq && which git
# Only makes skill available if all binaries found
```

### Environment Variable Validation

Checks if required environment variables are set:

```yaml
metadata:
  {
    "openclaw":
      {
        "requires": { "env": ["API_KEY", "DATABASE_URL"] },
      },
  }
```

**Validation**:
```bash
# Internally, OpenClaw checks:
[ -n "$API_KEY" ] && [ -n "$DATABASE_URL" ]
# Only makes skill available if all variables set
```

---

## Skill Loading Order

Skills are loaded in this order:

1. **Bundled Skills**: Gateway default skills
2. **Managed Skills**: Skills installed via `openclaw skills install`
3. **Workspace Skills**: Skills in local `skills/` directory
4. **Plugin Skills**: Skills bundled with plugins

**Priority**: Later entries override earlier ones (if conflicts exist).

---

## Skill Metadata Reference

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique skill identifier (lowercase, hyphens) |
| `description` | string | One-line description for agent understanding |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | string | - | Semantic version of skill |
| `author` | string | - | Skill author or team |
| `tags` | array | `[]` | Searchable tags for discovery |
| `disable-model-invocation` | boolean | `false` | If true, only user can invoke |
| `user-invokable` | boolean | `true` | If false, hidden from `/` menu |
| `allowed-tools` | array | all | List of tools available during execution |
| `metadata` | object | `{}` | Platform-specific configuration |

---

## Best Practices

### 1. Feature Gating

Always gate skills on their actual requirements:

```yaml
# Good: Gated on actual dependencies
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["ffmpeg"], "env": ["FFMPEG_PATH"] },
      },
  }

# Bad: No gating
metadata:
  {}
```

### 2. Clear Descriptions

Use single-line descriptions when possible:

```yaml
# Good: Clear, concise
description: "Search across all memories using semantic matching."

# Bad: Vague, verbose
description: |
  This skill allows you to search across all memories that have been
  stored in the system using semantic matching algorithms that
  understand the meaning of your query...
```

### 3. Minimal Dependencies

Only declare requirements that are truly necessary:

```yaml
# Good: Only necessary deps
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["curl"] },
      },
  }

# Bad: Unnecessary deps
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["curl", "jq", "git", "docker", "npm"] },
      },
  }
```

---

## Troubleshooting

### Skill Not Showing Up

**Symptom**: Skill doesn't appear in `openclaw skills list`

**Check**:
```bash
# Verify SKILL.md exists
ls -la my-plugin/skills/my-skill/SKILL.md

# Check YAML syntax
head -20 my-plugin/skills/my-skill/SKILL.md | yamllint

# Verify plugin manifest
cat my-plugin/openclaw.plugin.json | jq '.skills'

# Check requirements met
which <required-binary>
echo $<required-env-var>
```

### Skill Disabled Due to Requirements

**Symptom**: Skill listed but marked as "unavailable"

**Check**:
```bash
# Show skill status
openclaw skills show my-skill

# Check requirements
cat my-plugin/skills/my-skill/SKILL.md | grep -A 10 "requires:"

# Install missing deps
sudo apt-get install <missing-binary>
export MISSING_ENV="value"
```

---

## References

- **Skill Format**: [SKILL.md Format Specification](./skill-format.md)
- **Skill Patterns**: [Skill Implementation Guide](../core/skills.md)
- **Best Practices**: [Skill Description Best Practices](../core/best-practices.md)
- **Hook System**: [OpenClaw Hooks](./hooks.md)

---

*See Also*: [OpenClaw Overview](./01-overview.md) for general platform information
