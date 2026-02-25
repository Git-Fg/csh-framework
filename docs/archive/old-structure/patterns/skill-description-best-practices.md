# Skill Description Best Practices

> **Guideline**: Write skill descriptions that are concise, clear, and easy for agents to understand

---

## 📏 Core Principle

**Single-Line When Possible**: Aim for one-line descriptions that can be easily recognized and parsed. Multi-line descriptions are acceptable only when necessary for clarity or examples.

---

## ✅ When to Use Single-Line

### 1. Simple Directives

Use one line for straightforward directives or reminders:

```yaml
---
name: security-audit
description: "YOU MUST USE THIS SKILL BEFORE PROCEEDING with any changes to src/auth or src/middleware."
```

**Bad** (Multi-line):
```yaml
---
description: |
  YOU MUST USE THIS SKILL BEFORE PROCEEDING with any changes to src/auth or src/middleware.
  It is the ONLY authorized way to verify security checklist against authorization bypass.
  Mandatory for all structural reviews.
  Do not skip for any code change in these paths.
```

### 2. Simple Action Triggers

Use one line for clear action triggers:

```yaml
---
name: quick-format
description: "Run formatter on every file edit."
```

### 3. Core Functionality Summary

Use one line for straightforward skills with single purpose:

```yaml
---
name: database-backup
description: "Create daily database backups at midnight."
```

---

## 🚫 When Multi-Line is Acceptable

### 1. Complex Explanations

When a skill requires nuanced explanation, multi-line is acceptable:

```yaml
---
name: migration-guide
description: |
  Performs complex database migration from MySQL to PostgreSQL.
  Handles schema changes, data transformation, and validation.
  Use when database schema changes or migrating between versions.
```

### 2. Examples and Contrasts

When showing good vs. bad patterns, multi-line examples are appropriate:

```yaml
---
name: pattern-demo
description: |
  Good: "Run tests before committing."
  Bad: "Performs a security audit on codebase." 
  This example shows the difference between a simple directive and a complex description.
```

### 3. Structured Content Lists

For skills that require listing multiple items:

```yaml
---
name: list-tools
description: |
  Lists available CLI tools:
  - git: Version control
  - jq: JSON processing
  - curl: HTTP requests
```

---

## 📊 Clarity Guidelines

### 1. Be Specific

**One-line**:
```
"Creates daily backups at midnight."
```

**Multi-line (bad)**:
```
"This skill performs complex database migration from MySQL to PostgreSQL. 
It handles schema changes, data transformation, and validation."
```

### 2. Use Active Voice

**One-line**:
```
"Validates user inputs and blocks unauthorized access."
```

**Multi-line (bad)**:
```
"This skill checks various conditions including authentication, 
authorization, and access control mechanisms to ensure 
only authorized users can access sensitive data and perform 
privileged operations."
```

### 3. Avoid Unnecessary Detail

Don't include implementation details in description - that belongs in the skill body:

**One-line**:
```
"Automatically backs up database on schema changes."
```

**Multi-line (bad)**:
```
"This skill runs pg_dump command with --clean flag to create 
a clean backup, then creates a tarball archive using tar command,
then uploads the archive to S3 using aws cli s3 cp command
with metadata tracking in a separate file for audit purposes."
```

---

## ✅ Examples of Good Descriptions

### Simple Directive

```yaml
---
name: production-mode-warning
description: "⚠️ PRODUCTION: All changes require manual approval before proceeding."
```

### Core Functionality

```yaml
---
name: memory-search
description: "Search across all memories using semantic matching."
```

### Action Trigger

```yaml
---
name: git-hook
description: "Run linter and tests before committing changes."
```

---

## 🎓 Decision Tree

**Should I use single-line?**

| Description Type | Single-Line OK? | Example |
|-----------------|----------------|----------|
| Simple directive | ✅ | "YOU MUST USE THIS SKILL" |
| Action trigger | ✅ | "Run tests before committing" |
| Core function | ✅ | "Search across all memories" |
| Complex explanation | ✅ | "Performs complex database migration" |
| Implementation detail | ❌ | "This skill runs pg_dump command with tar command..." |

**Rule of Thumb**: Use single-line for simple, imperative statements. Use multi-line only when necessary for complexity, examples, or structured content.

---

## 📚 Reference

This guide complements [SKILL.md Format Specification](../reference/02-skill-format.md).

**See Also**: 
- [Agent-Usable CLI Tools](./agent-usable-cli-tools.md) — Build tools that are easy for agents to discover
- [Hooks Patterns](./03-hooks.md) — Hook design patterns for automation
- [Cross-Platform Skills](./04-cross-platform.md) — Full requirements for cross-platform compatibility

---

*Created by: Claw 🦞*
*Date: 2026-02-24*
