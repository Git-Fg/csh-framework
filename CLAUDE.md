# CLAUDE.md - CSH Framework Documentation

## Overview

This is the documentation repository for the **CSH Framework** (CLI-Skills-Hooks) - a methodology using CLI, Skills, and Hooks to create highly portable AI agent extensions.

**Location**: `/root/csh-dev/csh-framework/`

---

## Global Structure

```
csh-framework/
├── README.md              # Quick overview + navigation
├── USER_README.md        # Complete user guide (5,000+ lines)
├── CLAUDE.md            # This file - rules for working on docs
├── core/                # Core CSH concepts (14 files)
│   ├── philosophy.md    # WHY CSH exists
│   ├── architecture.md  # Three-layer architecture
│   ├── skills.md         # Skill implementation patterns
│   ├── hooks.md          # Hook patterns + mandatory language
│   ├── cli-first-architecture.md
│   ├── cognitive-load-optimization.md
│   ├── cross-platform.md
│   ├── development.md
│   ├── best-practices.md
│   ├── project-structure.md
│   ├── troubleshooting.md
│   ├── quickstart.md
│   └── overview.md
├── openclaw/           # OpenClaw platform integration (16 files)
│   ├── overview.md
│   ├── skills.md
│   ├── hooks.md
│   ├── skill-format.md
│   ├── hook-events.md
│   ├── platform-comparison.md
│   ├── cli-reference.md
│   ├── testing.md
│   ├── testing-guide.md
│   ├── behavioral-testing-approach.md
│   ├── test-workflows.md
│   ├── test-cases-templates.md
│   ├── exit-codes.md
│   ├── terminology.md
│   ├── reference.md
│   └── claude-code-testbed.md
└── claudecode/         # Claude Code platform integration (4 files)
    ├── overview.md
    ├── skills.md
    ├── hooks.md
    └── reference.md
```

---

## Frontmatter Standards (REQUIRED)

All MD files MUST include YAML frontmatter:

```yaml
---
title: "Short Descriptive Title"
summary: "One-line explanation of what this doc covers."
read_when:
  - When user wants to understand X
  - When implementing Y
  - Before starting Z project
---
```

**Rules:**
- `title`: Max 60 characters
- `summary`: Max 200 characters
- `read_when`: 2-4 bullet points max
- NO `version`, `author`, `date` fields (keeps frontmatter clean)

---

## Anti-Patterns to Enforce in Documentation

### 1. Skill Invocation Format (CRITICAL)

**CORRECT**: `use skill 'skill-name'` (with quotes, no slash)
```markdown
use skill 'decision-capture-workflow'
use skill 'workspace-audit-learn'
```

**INCORRECT** (old format to avoid):
```markdown
/skill decision-capture-workflow    # ❌ Slash format
use decision-capture-workflow        # ❌ Missing quotes
use the decision-capture skill      # ❌ Informal
```

### 2. Mandatory Language in Hooks

When documenting hooks that enforce skill usage:

**CORRECT**:
```markdown
[MANDATORY] You MUST use skill 'decision-capture-workflow' to learn:
- WHEN to capture decisions
- WHAT to structure for retrieval
- HOW to use CLI after learning workflow
```

**INCORRECT**:
```markdown
Consider using the decision-capture workflow skill   # ❌ Too weak
You should use skill                                 # ❌ Missing quotes
```

### 3. Hook → Skill Reference Pattern

Hooks should reference skills, NOT embed instructions:

**CORRECT**:
```markdown
## Session Start Hook
Injects: Skills menu with "use skill 'name'" format
Does NOT embed: Full skill instructions
```

**INCORRECT**:
```markdown
## Session Start Hook
Shows: Full CLI commands like "clawvault remember decision..."  # ❌
```

### 4. CLI --help Injection

Always include at session start:
```markdown
**CLI Reference:**
\`\`\`
clawvault --help    # Full command list
\`\`\`
```

---

## Writing Rules

### 1. Use Backticks for Code/Commands
- CLI commands: `clawvault remember`
- File paths: `/root/mastervault/`
- Tool names: `memory_search`

### 2. Use Bold for Key Terms
- **Skills**: Agent guidance modules
- **Hooks**: Context injection events
- **CLI**: Command-line interface

### 3. Tables for Comparisons
```markdown
| Aspect | CSH | MCP |
|--------|-----|-----|
| Architecture | CLI + files | Server-based |
| Complexity | Simple | Requires server |
```

### 4. Code Blocks with Language
```typescript
// TypeScript
api.on("message_received", async (ctx) => {
```

### 5. No Version Numbers in Headers
- ✅ `# Hook Patterns`
- ❌ `# Hook Patterns v2.0`

---

## Content Guidelines

### Core Docs (core/)
- Focus on **principles** and **why**
- Include anti-patterns
- Explain the problem before solution
- Reference other core docs with relative links

### Platform Docs (openclaw/, claudecode/)
- Focus on **implementation**
- Include code examples
- Reference core docs for principles
- Platform-specific quirks

### Testing Docs
- Document behavior, not implementation
- Use test case templates
- Include expected outputs

---

## Link Patterns

### Internal Links (use these)
```markdown
[Skill Implementation](./core/skills.md)
[Hook Events](./openclaw/hook-events.md)
[Philosophy](./core/philosophy.md#the-core-problem)
```

### External Links
```markdown
[OpenClaw Docs](https://docs.openclaw.ai/)
[Claude Code](https://code.claude.com/)
```

---

## Common Tasks

### Adding a New Doc
1. Create file in appropriate folder (core/openclaw/claudecode)
2. Add frontmatter (title, summary, read_when)
3. Use hierarchical headers (##, ###)
4. Add to README.md navigation

### Updating Existing Doc
1. Keep frontmatter minimal
2. Don't add version numbers
3. Preserve relative links
4. Update read_when if scope changes

### Cross-Referencing
- Core → Core: Direct links `[](./core/...)`
- Platform → Core: `[Philosophy](../core/philosophy.md)`
- Tests → Implementation: Link to relevant sections

---

## Key Principles to Maintain

1. **"use skill 'name'"** - Mandatory format for skill invocation
2. **Mandatory language** - "YOU MUST", "YOU SHOULD" in hooks
3. **CLI --help** - Inject at session start
4. **Hook → Skill** - Reference, don't embed instructions
5. **Frontmatter** - Required for all docs
6. **Minimal frontmatter** - Only title, summary, read_when

---

## Plugin Installation (Key Information)

### Dev-First Flow (Recommended for Development)

```bash
# 1. Link CLI locally for iterative development
cd /path/to/my-cli && npm link

# 2. Install plugin with -l (creates SYMLINK, not copy!)
openclaw plugins install -l /path/to/my-plugin

# 3. Edit code → changes auto-reload (no restart!)
```

### User Flow (Production)

```bash
# 1. Install CLI globally
npm install -g my-cli-tool

# 2. Install plugin (copies files)
openclaw plugins install /path/to/my-plugin
openclaw gateway restart
```

### What `-l` Does
- **Creates symlink** (not copy) - changes auto-reload
- **Hot reload** for hooks & plugin code - no restart needed
- **Installs BOTH skills AND hooks** automatically

### ⚠️ Hot Reload Reality
| Component | Hot Reload? |
|-----------|-------------|
| Hooks (api.on) | ✅ Yes |
| Plugin code | ✅ Yes |
| Skills (SKILL.md) | ❌ No - cached at startup, requires restart |
- **Recommended for development** - iterate faster!

### Core Principle
> **Plug-and-Play**: User clones repo → installs CLI → installs plugin → it just works
> **Dev-First**: Use `npm link` + `-l` for iterative development

### ⚠️ plugins.allow Issue

If plugin doesn't load after install:

```bash
# Check allowlist
cat ~/.openclaw/openclaw.json | jq '.plugins.allow'

# Empty = auto-discover ✅
# Has entries = ONLY those load! Must add your plugin
```

---

## Related Memory

See ClawVault memories tagged `csh-framework` for historical context:
```bash
clawvault search "csh framework"
```
