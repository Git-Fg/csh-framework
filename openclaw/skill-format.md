---
title: "SKILL.md Format Specification"
summary: "Complete specification for the SKILL.md format - YAML frontmatter schema, required fields, and metadata options."
read_when:
  - You are creating a new skill
  - You need to write SKILL.md files
  - You want to understand skill metadata
---

# SKILL.md Format Specification

This document provides the complete specification for the `SKILL.md` format used in the CSH Framework.

---

## 1. YAML Frontmatter Schema

Every skill must begin with YAML frontmatter containing metadata.

### Required Fields

```yaml
---
name: skill-name
description: One-line description of what this skill does
---
```

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `name` | string | Lowercase, hyphens only | Used for slash command invocation |
| `description` | string | Single line, max 200 chars | Elicits skill usage; appears in system prompt |

### Optional Fields

```yaml
---
name: my-skill
description: Brief description of skill purpose
version: 1.0.0
author: team-name
tags: [tag1, tag2]
disable-model-invocation: false
user-invokable: true
allowed-tools: [Read, Grep, Bash]
metadata: {}
---
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | string | - | Semantic version of the skill |
| `author` | string | - | Author or team name |
| `tags` | array | `[]` | Searchable tags for discovery |
| `disable-model-invocation` | boolean | `false` | If true, only user can invoke via slash command |
| `user-invokable` | boolean | `true` | If false, hidden from `/` menu but available to agent |
| `allowed-tools` | array | all tools | List of tool names available during skill execution |
| `metadata` | object | `{}` | Platform-specific configuration |

---

## 2. Markdown Structure

After the frontmatter, the skill content follows standard markdown structure.

### Recommended Section Order (Standard)

Every OpenClaw Hub skill (`SKILL.md`) should follow this internalized structure for high-density, tiered orchestration:

| Section | Importance | Purpose |
|---------|------------|---------|
| **Mission** | High | What is the overall goal of this skill? |
| **Success Criteria** | High | What does success look like? |
| **Protocol** | CRITICAL | Instructions on how to interact with this skill. |
| **Workflows** | CRITICAL | Links to spoke files (execution details). |
| **Quick Commands** | Medium | List of CLI commands (with terminal warning). |
| **Memory Guide** | Medium | Guidance on what to store in long-term memory. |

### Metadata Frontmatter

The frontmatter MUST follow the **Trigger-Protocol-Workflow** pattern.

```yaml
---
name: clawvault-memory
description: "Manage persistent memories. Use when user asks to 'find decisions' or 'remember X'. Workflows: workflow-search, workflow-remember. ALWAYS invoke this skill and read relevant workflow(s), they teach you correct way to use the CLI."
---
```

### Core Sections Template

#### Mission & Success Criteria
Define the high-level intent.

```markdown
## Mission
Ensure every architectural decision and significant lesson is captured in the vault. Don't let knowledge decay.

## Success Criteria
1. Every task that changes logic/infra is recorded via `clawvault remember`.
2. Existing knowledge is searched BEFORE proposing new solutions.
```

#### Protocol & Workflows
This is the "gatekeeper" section.

```markdown
## Protocol
YOU MUST read the relevant workflow before performing any action. These workflows contain the exact CLI syntax and safety checks required.

## Workflows
- **Search**: workflow-search (Read when: looking for past decisions, logs, or facts)
- **Remember**: workflow-remember (Read when: task complete, decision made, or lesson learned)
```

#### Quick Commands
Always include a warning about CLI vs Native tools.

```markdown
## Quick Commands
> [!WARNING]
> Use the `clawvault` CLI in bash (via `run_in_terminal`). These are CLI commands, NOT native AI tools.

- `clawvault search "query"`: Search all vault categories.
- `clawvault remember decision "key" --content "value"`: Store a new decision.
```

## Prerequisites

- Required tool 1
- Required tool 2

## Workflow

1. First step
2. Second step
3. Third step

## Success Criteria

- Criterion 1
- Criterion 2

## Error Handling

If [error condition]:
  1. Recovery step 1
  2. Recovery step 2

## Examples

### Example 1

```bash
# Command example
```

### Example 2

```bash
# Another example
```

## Notes

Additional notes or tips.
```

### Section Guidelines

| Section | Required | Purpose |
|---------|----------|---------|
| `# Title` | Yes | Human-readable skill name |
| Description | Yes (in frontmatter) | Elicitation for agent |
| `## When to Use` | Recommended | Clear trigger conditions |
| `## Prerequisites` | Recommended | Required tools, skills, or environment |
| `## Workflow` | Yes | Step-by-step instructions |
| `## Success Criteria` | Recommended | How to verify success |
| `## Error Handling` | Recommended | Recovery procedures |
| `## Examples` | Recommended | Usage examples |

---

## 3. Dynamic Content

### Argument Substitution

Pass user input into skills using variables:

| Syntax | Description |
|--------|-------------|
| `$ARGUMENTS` | All user input |
| `$0`, `$1`, ... | Positional arguments |
| `$KEYWORD` | Named parameters |

```markdown
## Task
Process $ARGUMENTS using the following steps:
1. Run `my-cli command $0`
2. Parse output for $1
```

### Command Execution

Inject shell output at load time using backticks:

```markdown
## Context
Current user: !`whoami`
Current directory: !`pwd`
Current date: !`date +%Y-%m-%d`
```

---

## 4. Complete Examples

### Minimal Skill

```markdown
---
name: hello
description: Print a greeting message
---

# Hello Skill

Prints "Hello, World!" to stdout.

## Workflow

Run: `echo "Hello, World!"`
```

### Full-Featured Skill

```markdown
---
name: github
description: GitHub operations via gh CLI: issues, PRs, CI runs. Use when checking PR status, creating issues, or viewing CI logs. NOT for local git operations.
version: 1.0.0
author: csh-framework
tags: [github, cli, git, pr, issues]
allowed-tools: [Bash, Read]
metadata: {"openclaw": {"emoji": "🐙", "requires": {"bins": ["gh"]}}}
---

# GitHub Skill

Perform GitHub operations using the `gh` CLI.

## When to Use

- Checking PR status, reviews, or merge readiness
- Viewing CI/workflow run status and logs
- Creating, closing, or commenting on issues
- Creating or merging pull requests

## When NOT to Use

- Local git operations (commit, push, pull) — use `git` directly
- Non-GitHub repos — different CLIs required

## Prerequisites

- `gh` CLI installed and authenticated

## Setup

```bash
# Authenticate (one-time)
gh auth login

# Verify
gh auth status
```

## Workflow

1. Identify the operation needed (PR, issue, or CI)
2. Construct the appropriate `gh` command
3. Execute with `--repo owner/repo` when not in git directory

## Common Commands

### Pull Requests

```bash
# List PRs
gh pr list --repo owner/repo

# View PR
gh pr view 55 --repo owner/repo

# Create PR
gh pr create --title "feat: add feature" --body "Description"
```

### Issues

```bash
# List issues
gh issue list --repo owner/repo --state open

# Create issue
gh issue create --title "Bug: something broken" --body "Details..."
```

### CI Runs

```bash
# List recent runs
gh run list --repo owner/repo --limit 10

# View failed logs
gh run view <run-id> --repo owner/repo --log-failed
```

## Success Criteria

- Command executes without errors
- Output contains expected data
- Rate limits not exceeded

## Error Handling

If rate limited:
1. Wait for the specified time
2. Retry with `--cache 1h` flag for repeated queries
3. If auth error, run `gh auth login`

## Notes

- Always specify `--repo owner/repo` when not in a git directory
- Use URLs directly: `gh pr view https://github.com/owner/repo/pull/55`
```

---

## 5. Validation Rules

### Name Constraints

- Must be lowercase
- Use hyphens: `my-skill`, not `my_skill` or `mySkill`
- Max 50 characters
- No special characters

### Description Constraints

- Max 200 characters
- Single line (no line breaks)
- Should answer: "When should I use this?"

### Frontmatter Rules

- Use `---` delimiters
- Keys on single lines (no multi-line YAML values)
- Boolean values: `true` / `false` (lowercase)
- Arrays: `[item1, item2]` or:
  ```yaml
  tags:
    - item1
    - item2
  ```

---

## 6. Best Practices

### DO

- Keep descriptions concise and action-oriented
- Include specific prerequisites
- Define clear success criteria
- Add error handling guidance
- Use code blocks for examples

### DON'T

- Create monolithic skills that do everything
- Use vague descriptions like "does useful things"
- Skip prerequisites — assume nothing about the environment
- Leave error handling as "contact support"
- Use multi-line YAML in frontmatter

---

*Related: [Cross-Platform Skills](./../patterns/04-cross-platform.md), [Hooks Events](./03-hook-events.md)*
