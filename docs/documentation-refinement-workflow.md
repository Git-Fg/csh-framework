---
title: "Documentation Refinement Workflow"
summary: "Critical lesson: When refining documentation, ALWAYS integrate into existing structure rather than creating new files."
read_when:
  - You are refining documentation for any project
  - You are about to create a new documentation file
  - You need to understand documentation workflow principles
---

# Documentation Refinement Workflow

> **PRINCIPLE**: INTEGRATE, DON'T DUPLICATE

When refining documentation, always add to existing files instead of creating new ones.

---

## Why This Matters

| Problem | Solution |
|---------|----------|
| Duplicate content | Integrate into existing files |
| Navigation confusion | Maintain single source of truth |
| Maintenance burden | Easier when not duplicated |
| Structure violation | Follows project architecture |

---

## Workflow: Documentation Refinement

### Step 1: Read CLAUDE.md First

```bash
# ALWAYS read CLAUDE.md first
cat /root/csh-dev/csh-framework/CLAUDE.md
```

**Check for**:
- Frontmatter standards
- Style guidelines
- Anti-patterns to enforce
- Linking conventions
- File structure rules

### Step 2: Find Existing Files

```bash
# Search for existing relevant files
find /root/csh-dev/csh-framework -name "*.md" | grep -i hook

# Read them to understand structure
cat csh-framework/openclaw/hook-events.md
cat csh-framework/core/hooks.md
```

### Step 3: Integrate Content

```markdown
## Section Title (use existing heading levels)

Add new subsections, examples, or clarifications
following existing style and structure.
```

**Follow**:
- Existing heading levels (`##`, `###`)
- Existing frontmatter format
- Existing code block styles
- Existing table formats

### Step 4: Update Navigation

```bash
# Update README.md or TOC if needed
cat csh-framework/README.md
```

---

## When to Create New Files (Rare)

**ONLY create new files when**:
1. Content is truly new (not refining existing documentation)
2. No appropriate existing file exists for the topic
3. Project architecture explicitly requires it

**Examples**:
- New plugin with unique functionality
- Major architectural change requiring migration guide
- New concept not documented anywhere

## When to Integrate (Common)

**ALWAYS integrate when**:
1. Adding examples to existing guides
2. Clarifying existing concepts
3. Adding troubleshooting to existing docs
4. Updating reference information

---

## Real Example: ClawVault Hooks

### Wrong Approach

```bash
# Created NEW files instead of integrating
touch csh-framework/docs/CLAWVAULT_HOOKS_GUIDE.md      # ❌ NEW file
touch csh-framework/docs/HOOKS.md                       # ❌ NEW file
```

**Problems**:
- `openclaw/hook-events.md` already exists
- `core/hooks.md` already covers hook patterns
- Creating new files duplicates structure

### Correct Approach

```bash
# Updated EXISTING files
# 1. Add ClawVault case study to openclaw/hook-events.md
# 2. Add specific examples to core/hooks.md
```

**Benefits**:
- Maintains existing structure
- Single source of truth
- Follows navigation

---

## Anti-Pattern: "Just Create a New File"

### Symptoms

- Creating `guide-X.md` when `X.md` exists
- Creating `reference-Y.md` when `reference/` folder exists
- Creating `how-to-Z.md` when similar content is in existing docs

### Root Causes

1. Didn't read existing structure
2. Didn't understand navigation
3. Didn't check CLAUDE.md for guidelines
4. Thought "easier to create new than update existing"

### Correction

**ALWAYS**:
1. Read CLAUDE.md first
2. Find existing relevant files
3. Integrate content appropriately
4. Only create new if truly needed

---

## Checklist: Documentation Work

### Before Starting

- [ ] Read `CLAUDE.md` (always!)
- [ ] Understand existing structure
- [ ] Find relevant existing files
- [ ] Check for duplication

### During Work

- [ ] Integrate into existing files (not create new)
- [ ] Follow existing style (headings, frontmatter)
- [ ] Use proper formatting (tables, code blocks)
- [ ] Update navigation/TOC

### Before Committing

- [ ] No duplicate files created
- [ ] Content integrated properly
- [ ] Navigation updated if needed
- [ ] Frontmatter correct (title, summary, read_when)

---

## Key Principles

1. **Read CLAUDE.md first** - Always understand project guidelines
2. **Find existing files** - Search before creating new ones
3. **Integrate content** - Add to appropriate existing sections
4. **Follow existing style** - Use same formatting and structure
5. **Only create new if necessary** - When truly no appropriate file exists
6. **Single source of truth** - Maintain documentation integrity

---

## Sources

- [AGENTS.md](../../AGENTS.md) - Agent documentation standards
- [CLAUDE.md](../../CLAUDE.md) - Project guidelines
- Lesson: Never create new documentation files - [Lesson](../../../mastervault/lessons/never-create-new-documentation-files.md)
