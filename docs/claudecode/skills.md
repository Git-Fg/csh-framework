# Claude Code Skills

How skills work in Claude Code and the CSH Framework integration.

---

## Discovery Mechanism

Claude Code automatically discovers skills from standard directories:

### Skill Locations

Claude Code scans locations in this order:

1. **Project skills**: `.claude/skills/` in project root
2. **User skills**: `~/.claude/skills/`
3. **Plugin skills**: Skills bundled in plugins (`<plugin>/skills/`)

### Plugin-Based Skills

For CSH Framework plugins, skills are defined in the plugin structure:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    ├── memory-search/
    │   └── SKILL.md
    └── memory-remember/
        └── SKILL.md
```

### Skill Manifest

Skills are referenced in the plugin manifest:

```json
{
  "name": "my-csh-plugin",
  "skills": {
    "memory-search": {
      "description": "Search across all memories using semantic matching"
    },
    "memory-remember": {
      "description": "Store information for later retrieval"
    }
  }
}
```

---

## Configuration

### Enable/Disable Skills

Skills can be controlled via project settings:

```json
// .claude/settings.json
{
  "skills": {
    "memory-search": {
      "enabled": true
    }
  }
}
```

### Skill Namespacing

Skills in plugins are namespaced:

| Source | Invocation | Example |
|---------|-------------|----------|
| Project skill | `/skill-name` | `/my-skill` |
| Plugin skill | `/plugin:skill-name` | `/my-plugin:memory-search` |

---

## Skill Loading

### Automatic Discovery

Claude Code automatically discovers skills on startup:

1. Scans `.claude/skills/` directories
2. Parses plugin manifests
3. Loads SKILL.md files
4. Makes skills available to agent

### Loading Order

Skills are loaded in precedence order:

1. **Project skills**: Local to current project
2. **Plugin skills**: Bundled with plugins
3. **User skills**: Global user skills

**Priority**: Later entries can override earlier ones.

---

## Skill Metadata

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | One-line description for agent understanding |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | boolean | `true` | Enable/disable skill |
| `allowedTools` | array | all | Tools available during execution |

---

## Skill Usage

### Agent Invocation

Agents can invoke skills in two ways:

1. **Direct Invocation**: Agent recognizes when to use skill
2. **Slash Command**: User invokes explicitly via `/skill-name`

### Permission Control

Skills can be restricted:

```json
{
  "skills": {
    "admin-only": {
      "enabled": true,
      "userInvokable": false  // Only agent can use
    }
  }
}
```

---

## Best Practices

### 1. Clear Descriptions

```json
// Good: Clear, concise
{
  "description": "Search across all memories using semantic matching."
}

// Bad: Vague, verbose
{
  "description": "This skill enables searching across memory files that have been stored..."
}
```

### 2. Namespaced Plugins

Use plugin namespacing to avoid conflicts:

| Good | Bad |
|------|------|
| `/my-plugin:memory-search` | `/memory-search` (conflicts) |
| `/my-plugin:config-get` | `/get-config` (generic) |

### 3. Minimal Dependencies

Claude Code has limited dependency checking. Only declare skills that work reliably:

```json
// Good: Skill works independently
{
  "memory-search": {
    "description": "Search memories"
  }
}

// Bad: Skill requires external state
{
  "external-api": {
    "description": "Requires external API connection"
  }
}
```

---

## Troubleshooting

### Skill Not Discovered

**Symptom**: Skill doesn't appear in available skills

**Check**:
```bash
# Verify SKILL.md exists
ls -la .claude/skills/my-skill/SKILL.md
ls -la my-plugin/skills/my-skill/SKILL.md

# Check project settings
cat .claude/settings.json | jq '.skills'

# Restart Claude Code
# Skills reload on startup
```

### Skill Namespacing Issues

**Symptom**: Can't invoke plugin skill

**Check**:
```bash
# Use correct format
# Correct: /my-plugin:skill-name
# Incorrect: /skill-name, /my_plugin:skill_name
```

---

## References

- **Skill Format**: [SKILL.md Format Specification](../openclaw/skill-format.md)
- **Skill Patterns**: [Skill Implementation Guide](../core/skills.md)
- **Best Practices**: [Skill Description Best Practices](../core/best-practices.md)
- **Hook System**: [Claude Code Hooks](./hooks.md)

---

*See Also*: [Claude Code Overview](./01-overview.md) for general platform information
