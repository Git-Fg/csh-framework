# Cross-Platform Skills

> **Hot Reload in Development**: CSH Framework (via OpenClaw) supports hot reload of skills using `openclaw -l`. See [Hot Reload Guide](../guides/hot-reload-guide.md).

---

## 1. Directory Structure

For cross-platform compatibility within a plugin, the skill must be placed in a `skills/` directory at the plugin root.

```
my-plugin/
  ├── openclaw.plugin.json        # OpenClaw manifest
  ├── .claude-plugin/             # Claude Code manifest directory
  │   └── plugin.json
  └── skills/
      └── my-skill/              # Skill folder
          └── SKILL.md            # Entry point
```

### Key Points

- **Claude Code**: Skills are prefixed with the plugin name: `/plugin-name:skill-name`
- **OpenClaw**: Skills participate in global precedence (Workspace > Managed > Bundled)
- Use `{baseDir}` (OpenClaw) or relative paths (Claude Code) to reference supporting files

---

## 2. Frontmatter Specification

To work on both platforms, frontmatter must follow the **AgentSkills** specification while observing specific parsing constraints of each engine.

### Required Fields

| Field | Requirement | Purpose |
|-------|-------------|---------|
| `name` | Lowercase/Hyphens | Display name and slash command (`/name`) |
| `description` | Single line | **Elicitation**: Must precisely describe triggers and conditions. Acts as a "search index" for agent memory. |

### Compatible Configuration (Booleans)

| Field | Compatible Value | Effect |
|-------|-----------------|--------|
| `disable-model-invocation` | `true` / `false` | If `true`, only to user can trigger to skill via slash command |
| `user-invocable` | `true` / `false` | If `false`, skill is hidden from `/` menu but available to agent |

### Tool Restrictions

Both platforms support restricting which tools are available when a skill is active.

```yaml
---
name: restricted-skill
description: Skill with limited tool access
allowed-tools: Read, Grep, Bash
---
```

---

## 3. Platform-Specific Metadata (Gating)

OpenClaw requires additional metadata to determine if a skill is eligible based on on the environment. This must be formatted as a **single-line JSON object** for compatibility with with OpenClaw parser.

```yaml
---
name: my-skill
description: Performs an action
metadata: {"openclaw": {"requires": {"bins": ["curl"], "env": ["API_KEY"]}}}
---
```

### Common OpenClaw Constraints

| Constraint | Type | Description |
|------------|------|-------------|
| `requires.bins` | Array | List of binaries that must be on the host `PATH` |
| `requires.env` | Array | List of environment variables required for to skill |
| `os` | Array | List of supported platforms (`linux`, `darwin`, `win32`) |

### Complete Metadata Example

```yaml
---
name: github
description: GitHub operations via gh CLI
metadata:
  {
    "openclaw": {
      "emoji": "🐙",
      "requires": {
        "bins": ["gh"]
      },
      "os": ["linux", "darwin", "win32"],
      "install": [
        {
          "id": "brew",
          "kind": "brew",
          "formula": "gh",
          "bins": ["gh"],
          "label": "Install GitHub CLI (brew)"
        }
      ]
    }
  }
---
```

### Additional Metadata Fields

| Field | Type | Description | Example |
|--------|------|-------------|---------|
| `homepage` | URL | URL shown in UI | `"https://github.com/cli/gh"` |
| `primaryEnv` | string | Env var for apiKey injection | `"GITHUB_TOKEN"` |
| `requires.config` | array | Config paths that must be truthy | `["github.enabled"]` |
| `requires.anyBins` | array | At least one bin must exist | `["npm", "pnpm", "yarn"]` |
| `always` | boolean | Always include skill (skip gates) | `true` |

---

## 4. Substitution and Dynamism

### String Substitutions

Both platforms support `$ARGUMENTS` (or shorthand `$0`, `$1`, etc.) to pass user input into the skill.

```markdown
## Task
Process $ARGUMENTS using ::: context above.
```

### Dynamic Context (`!command`)

- **Claude Code**: Natively supports to `!command` syntax to inject shell output at load time
- **OpenClaw**: Utilizes this primarily for pre-invocation context

```markdown
## Context
Current Date: !`date`

## Task
Process $ARGUMENTS using ::: context above.
```

---

## 5. Elicitation Strategy

### The Power of Description

The `description` is the only information that agent sees **before** making the choice to load a skill. Both systems use the description as a "compact tool" that is always present in the system prompt.

### The "What-When-Not" Pattern

Your description must elicit use of the skill. It should state:

1. **What/Why**: Primary value proposition and specific problem solved
2. **When**: Clear, actionable triggers (file types, task names, conditions)
3. **Not**: Explicit false positives or scenarios where skill should **not** be used

### Example: Weak vs Strong Descriptions

**Weak**:
> "Performs a security audit on codebase."

**Strong (Informative)**:
> "Enforces mandatory security checklist to prevent authorization bypass regressions. Use when modifying files in `src/auth` or `src/middleware`, or during formal structural reviews. Not for simple logic refactors, style-only changes, or non-sensitive documentation updates."

---

## 6. Skill-Hook Bridge Pattern

Because of agent's autonomous decision-making can be inconsistent, hooks support skill usage.

### Three Levels of Hook Support

| Level | Type | Description |
|-------|------|-------------|
| **Level 1** | Guidance | Gentle reminder: "You might want to call `/my-skill`" |
| **Level 2** | Assertion | Explicit nudge for skill usage |
| **Level 3** | Context Injection | Provide necessary context to make skill usage obvious |

### Implementation Example

**The Skill (`SKILL.md`)**:
```markdown
---
name: security-audit
description: "YOU MUST USE THIS SKILL BEFORE PROCEEDING with any changes to `src/auth` or `src/middleware`. Only authorized way to verify of security checklist. Not for documentation changes."
---
# Security Audit Workflow
[... instructions for mandatory checks ...]
```

**The Hook (Claude Code `hooks/hooks.json`)**:
```json
{
  "on_file_write": {
    "include": ["src/auth/**/*.ts", "src/middleware/**/*.ts"],
    "prompt": "You just modified a security-critical file. You MUST invoke '/security-audit' before proceeding to ensure no authorization regressions."
  }
}
```

**The Hook (OpenClaw `HOOK.md`)**:
```markdown
---
on: filesystem.write
include: ["src/auth/**/*.ts", "src/middleware/**/*.ts"]
---
# Security Audit Trigger
You just modified a security-critical file. You MUST invoke into '/security-audit' skill before finalizing this task to comply with with mandatory checklist.
```

---

## 7. Implementation Checklist

1. **Namespace Awareness**
   - In Claude Code: skill prefixed as `/plugin-name:skill-name`
   - In OpenClaw: participates in global precedence

2. **Skill-CLI Alignment**
   - Align skills with 5-10 first-layer commands of your CLI
   - Each top-level command family should have a corresponding skill

3. **Path References**
   - Use `{baseDir}` (OpenClaw) or relative paths (Claude Code) to reference supporting files

4. **Parsing Safety**
   - Keep frontmatter keys on single lines
   - Avoid complex YAML nesting

5. **Tool Names**
   - Use standard PascalCase names: `Bash`, `Read`, `Grep`

---

## 8. OpenClaw Advantages (Hot Reload)

### Development Mode: OpenClaw `-l` Flag

When running OpenClaw with the `-l` (livereload) flag in development mode:

**Automatic Detection**:
- Changes to SKILL.md files are detected in real-time
- Hooks are reloaded automatically
- Plugin manifests are re-scanned
- No manual intervention needed

**Benefits**:
- ✅ **Rapid Iteration**: Save changes → automatic reload → test
- ✅ **No Gateway Restart**: Changes apply immediately
- ✅ **Faster Development Cycles**: No need to restart between changes
- ✅ **Live Feedback**: See impact of changes instantly

**Workflow**:
```bash
# Start OpenClaw with livereload
openclaw -l

# Edit your skills
# Changes detected and reloaded automatically
# Test changes
# Iterate quickly without restart
```

### Comparison with Claude Code

| Feature | OpenClaw | Claude Code |
|----------|-----------|-------------|
| **Hot Reload** | ✅ Automatic with `-l` flag | ❌ Requires gateway restart |
| **Skill Discovery** | ✅ Gateway-managed, auto-discovered | ❌ Plugin-scoped only |
| **Change Detection** | ✅ Real-time file watching | ❌ Requires manual restart |
| **Iteration Speed** | ✅ Fast: save → reload → test | ⚠️ Slower: save → restart → new session |

### Context Architecture Note

The "pre-injected" description from archived ClawVault docs was misunderstood. In the CSH Framework:

1. **Rules Files**: Project-specific configuration (settings, policies) are loaded at session start as the **first tokens** to establish the foundation for all interactions in that session.

2. **Hooks**: Additional context is injected **dynamically** throughout the conversation based on events (before commands, after commands, on changes, etc.) to provide real-time guidance.

3. **Skills**: Available as guidance references and are loaded **on-demand** (only when the agent actually decides to use them) to avoid context bloat.

This **three-layer system** enables:
- ✅ Rich initial context from project configuration
- ✅ Event-driven context from hooks
- ✅ Just-in-time skill guidance
- ✅ **Hot reload capability** in dev mode

---

*See Also*: [Hot Reload Guide](../guides/hot-reload-guide.md) for complete workflow
