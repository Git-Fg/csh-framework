# Cross-Platform Skills

This document defines the requirements for creating `SKILL.md` files that are compatible with both **Claude Code** and **OpenClaw** while residing within plugins.

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
| `description` | Single line | **Elicitation**: Must precisely describe triggers and conditions. Acts as the "search index" for agent memory. |

### Compatible Configuration (Booleans)

| Field | Compatible Value | Effect |
|-------|-----------------|--------|
| `disable-model-invocation` | `true` / `false` | If `true`, only the user can trigger the skill via slash command |
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

OpenClaw requires additional metadata to determine if a skill is eligible based on the environment. This must be formatted as a **single-line JSON object** for compatibility with the OpenClaw parser.

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
| `requires.env` | Array | List of environment variables required for the skill |
| `os` | Array | List of supported platforms (`linux`, `darwin`, `win32`) |

### Complete Metadata Example

```yaml
---
name: github
description: GitHub operations via gh CLI
metadata: {"openclaw": {"emoji": "🐙", "requires": {"bins": ["gh"]}, "os": ["linux", "darwin", "win32"], "install": [{"id": "brew", "kind": "brew", "formula": "gh", "bins": ["gh"], "label": "Install GitHub CLI (brew)"}]}}}
---
```

---

## 4. Substitution and Dynamism

### String Substitutions

Both platforms support `$ARGUMENTS` (or shorthand `$0`, `$1`, etc.) to pass user input into the skill.

```markdown
## Task
Process $ARGUMENTS using the context above.
```

### Dynamic Context (`!command`)

- **Claude Code**: Natively supports the `!command` syntax to inject shell output at load time
- **OpenClaw**: Utilizes this primarily for pre-invocation context

```markdown
## Context
Current Date: !`date`

## Task
Process $ARGUMENTS using the context above.
```

---

## 5. Elicitation Strategy

### The Power of the Description

The `description` is the only information the agent sees **before** making the choice to load the full `SKILL.md`. Both systems use the description as a "compact tool" that is always present in the system prompt.

### The "What-When-Not" Pattern

Your description must elicit the use of the skill. It should state:

1. **What/Why**: Primary value proposition and specific problem solved
2. **When**: Clear, actionable triggers (file types, task names, conditions)
3. **Not**: Explicit false positives or scenarios where skill should **not** be used

### Example: Weak vs Strong Descriptions

**Weak**:
> "Performs a security audit on the codebase."

**Strong (Informative)**:
> "Enforces the mandatory security checklist to prevent authorization bypass regressions. Use when modifying files in `src/auth` or `src/middleware`, or during formal structural reviews. Not for simple logic refactors, style-only changes, or non-sensitive documentation updates."

**Strong (Assertive)**:
> "Enforces the mandatory security checklist to prevent authorization bypass regressions. PROACTIVELY USE this skill when modifying files in `src/auth` or `src/middleware` to prevent critical auth-bypass regressions. This skill implements the mandatory security checklist which is superior to general reasoning for structural reviews. Not for documentation or style changes."

**Strong (Mandatory)**:
> "Enforces the mandatory security checklist to prevent authorization bypass regressions. YOU MUST USE THIS SKILL BEFORE PROCEEDING with any changes to `src/auth` or `src/middleware`. It is the ONLY authorized way to verify the security checklist against authorization bypass. Mandatory for all structural reviews. Do not skip for any code change in these paths."

### The 3 Degrees of Elicitation

| Degree | Focus | When to Use |
|--------|-------|-------------|
| 1: Informative | Utility and boundaries | Standard skills with clear use cases |
| 2: Assertive | Proactive use and risk prevention | High-stakes workflows that should be used more often |
| 3: Mandatory | Compliance and enforcement | Critical guardrails that must always be followed |

---

## 6. Skill-Hook Bridge Pattern

Because the agent's autonomous decision-making can be inconsistent, hooks support skill usage.

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
description: "YOU MUST USE THIS SKILL BEFORE PROCEEDING with any changes to `src/auth` or `src/middleware`. Only authorized way to verify the security checklist. Not for documentation changes."
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
You just modified a security-critical file. You MUST invoke the '/security-audit' skill before finalizing this task to comply with the mandatory checklist.
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
   - Use `{baseDir}` (OpenClaw) or relative paths (Claude Code)

4. **Parsing Safety**
   - Keep frontmatter keys on single lines
   - Avoid complex YAML nesting

5. **Tool Names**
   - Use standard PascalCase names: `Bash`, `Read`, `Grep`

---

## 8. The "Reluctant Agent" Problem

### The Challenge

As a default rule, assume that **AI agents do not call skills enough**. Model reluctance to "switch modes" is the primary reason skills are often ignored despite being perfectly relevant.

### Current LLM Reality

| Capability | Level |
|------------|-------|
| Pattern matching | Excellent (recognizes when skill is relevant) |
| Mode switching | Poor (struggles to actually invoke skill) |
| Invocation rate | ~30% even with strong elicitation |

### Why Hooks Matter

Hooks compensate for current model limitations:

- **Phase 1**: Direct Support ("Ensure skill usage has context")
- **Phase 2**: Optional Guidance ("Provide additional context if relevant")

As models improve (fine-tuned on skill usage, AgentSkills training data), hook enforcement may become unnecessary.

---

*Related: [SKILL.md Format Reference](./../reference/02-skill-format.md), [Hooks Pattern](./03-hooks.md)*
