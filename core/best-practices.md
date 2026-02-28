---
title: "Best Practices"
summary: "Comprehensive best practices for developing CSH Framework skills, tools, and testing strategies."
read_when:
  - You want to write effective skill descriptions
  - You are designing new tools for agents
  - You need testing validation guidelines
---

# Best Practices for CSH Framework Development

Comprehensive best practices for developing CSH Framework skills, tools, and testing strategies.

---

## Part 1: Skill Description Best Practices

> **Guideline**: Write skill descriptions that are concise, clear, and easy for agents to understand

### Core Principle: The Trigger-Protocol-Workflow Pattern

**Single-Line (MANDATORY)**: Aim for a concise, single-line description that includes the trigger and the protocol.

#### Description Best Practices

**Pattern**: `[What it does]. Use when [Triggers]. Workflows: [Spoke Names]. [Mandatory Protocol].`

| Feature | Guideline |
|---------|-----------|
| **Triggers** | Use keywords like 'find', 'remember', 'browse'. |
| **Workflows** | Cite spoke files without extensions (`workflow-search`). |
| **Protocol** | Explicitly command the agent to read the workflow. |

**Example (MANDATORY)**:
```yaml
---
name: clawvault-memory
description: "Manage persistent memories. Use when user asks to 'find decisions', 'remember X', or 'browse logs'. Workflows: workflow-search, workflow-remember. ALWAYS invoke this skill and read relevant workflow(s), they teach you correct way to use the CLI."
```

#### CLI vs. Native Tool Distinction

Always clarify that specialized tools are **CLI-based** (requiring the terminal/exec tool) and NOT native functions (like `read_file` or `edit_file`).

**Example**:
```markdown
## Quick Commands
Use the `clawvault` CLI in bash (via `run_in_terminal`). These are CLI commands, NOT native tools.
```

### When to Use Single-Line Description

#### Hub Skills
All Hub skills MUST use a single-line description to stay within the agent's primary context without causing bloat.

```yaml
---
name: security-audit
description: "Identify vulnerabilities. Use when working on 'auth' or 'api'. Workflows: audit-jwt, audit-cors. ALWAYS read workflow(s) first."
```

#### Action Triggers
Use one line for clear action triggers that don't follow the Hub & Spoke pattern:

```yaml
---
name: quick-format
description: "Run formatter on every file edit."
```

### When Multi-Line Description is Acceptable

**Note**: Multi-line descriptions are **HIGHLY DISCOURAGED** for Hub skills and should only be used for standalone, non-orchestrating utilities.

#### Complex Context Injection
When a skill requires nuanced explanation for standalone usage:

```yaml
---
name: migration-guide
description: |
  Performs complex database migration.
  Use when 'schema changes' or 'v1 to v2'.
  Provides SQL generation and validation.
```

### Clarity Guidelines

#### Be Imperative
Always use direct, verb-first instructions.

**Good**: "Search for memories before creating new ones."
**Bad**: "The agent can search for memories if it wants to avoid duplicates."

#### Use Mission & Success Criteria
Instead of a generic "About this skill" section, use **Mission** and **Success Criteria** to define high-level behavioral goals.

**Mission**: "Prevent long-term context loss by ensuring every decision is captured."
**Success Criteria**: "Every major task ends with a `memory-remember` call."


### Decision Tree

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

## Part 2: Testing Validation Best Practices

> **Validation Principle**: NEVER trust a response that claims a skill or CLI was used without verification; always validate independently
> **Safety Principle**: ALWAYS add strong instructions to prevent unwanted edit tool usage during testing

### Purpose

CSH Framework plugins and skills must be tested and validated thoroughly. These best practices prevent common issues:

1. **False Positives**: AI agents hallucinate skill usage
2. **Unintended Actions**: Edit tool modifies files when not expected
3. **Validation Gaps**: Testing doesn't verify actual behavior
4. **OpenClaw Integration**: Troubleshooting and project review via chat messages

### Never Trust Agent Claims

#### The Hallucination Problem

AI agents frequently report that a skill or CLI tool was used when, in reality, they:
- Misinterpreted a command's output
- Hallucinated skill invocation
- Confused similar-sounding commands
- Reported successful execution when it failed

#### Independent Verification Required

**Never** trust an agent's statement like:

- ❌ "The skill was called" (without checking logs)
- ❌ "I used the CLI tool" (without verifying execution)
- ❌ "The hook was triggered" (without checking hook logs)

**ALWAYS** verify independently:

✅ **Check Hook Logs**: Verify hooks actually fired
✅ **Check Agent State**: Query agent state programmatically
✅ **Check Tool Execution**: Verify CLI tool was called
✅ **Check Session Messages**: Review actual conversation messages

#### Verification Workflow

```bash
# Good: Independent verification
echo "Testing skill: /memory-search"

# 1. Check if skill was invoked
SKILL_INVOKED=$(openclaw logs --follow --tail 50 | grep -i "memory-search")
if [ "$SKILL_INVOKED" ]; then
  echo "✅ Skill was invoked"
else
  echo "❌ Skill was NOT invoked"
fi

# 2. Check agent state
echo "Checking agent state..."
openclaw agent get --agent-id "agent-telegram:main" | jq -r '.last_user_message'

# 3. Verify actual skill execution
# (Don't trust agent's claims)

# 4. Check logs for actual tool calls
openclaw logs --follow --tail 20 | grep -i "memory-search"
```

**Bad**: Trust agent blindly
```bash
# Bad: Trust agent's word without verification
echo "Agent said it used /memory-search, so it worked!"

# But what if agent halluncinated or misconfigured?
# We're testing, not assuming success.
```

### Always Prevent Edit Tool Usage

#### The Unwanted Edit Problem

During testing and development, the **edit tool** (or `Write` tool) should NEVER be used unless:

1. **You explicitly request it** (user says: "please edit file X")
2. **The task requires it** (agent says: "I need to update the file")
3. **It's part of the test case** (automated test scripts)

#### Unintended Edit Tool Usage

**When NOT to use Edit/Write**:

**Development**:
- ❌ Do NOT use edit tool for routine skill testing
- ❌ Do NOT use edit tool to "try something"
- ❌ Do NOT use edit tool to "experiment with code"
- ❌ Do NOT use edit tool to "make a quick fix"
- ✅ Use edit tool ONLY for the specific test case

**Testing**:
- ❌ Do NOT use edit tool during validation tests
- ❌ Do NOT let agent "fix things" during testing
- ❌ Do NOT allow automatic file modifications
- ✅ Use read/verify tools ONLY

**Production**:
- ❌ Do NOT use edit tool in production workflows
- ✅ Use edit tool ONLY when explicitly requested by user

#### Preventing Edit Tool Usage

**Option 1: Skill Instructions**
```markdown
---
name: test-skill
description: "Test validation without file modifications"

---

## Testing Rules

**CRITICAL**: DO NOT use the Edit or Write tool during testing.

Use ONLY:
- Read tool to inspect files
- Bash/Exec tool to run commands
- Verify tool to check outputs

This prevents accidental file modifications during testing.
```

**Option 2: Hook Enforcement**
```bash
#!/bin/bash
# hooks/pre-tool-use.sh

read -r event
tool=$(echo "$event" | jq -r '.payload.tool')

if [[ "$tool" == "Edit" || "$tool" == "Write" ]]; then
  # Block edit/write during testing
  echo "⚠️ Edit/Write tool blocked during testing."
  echo "Use Read, Bash, or Exec tools instead."
  exit 1
fi
```

### OpenClaw-Specific Testing

#### Gateway Status Verification

Before testing any skill, ensure the OpenClaw Gateway is operational:

```bash
# Check the immediate status
openclaw gateway status

# If state is "stopped" or "error", restart:
openclaw gateway restart

# Deep health check for model availability
openclaw health
```

**Success Criteria**:
- Service state: running
- Control plane: active
- RPC probe: ok

#### Skill and Hook Verification

Use the CLI to verify that OpenClaw "sees" your CSH changes:

```bash
# List all loaded skills
openclaw skills list | grep "csh"

# List enabled plugins and hooks
openclaw plugins list

# Verify your specific skill is active
openclaw skills list | grep -i "memory-search"
```

#### Real Interaction Testing (HTTP)

Use the HTTP endpoint to simulate how sessions behave in production:

```bash
curl -s -X POST http://127.0.0.1:18789/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN" \
  -d '{
    "model": "agent:main",
    "messages": [
      {"role": "user", "content": "Use memory-search skill to find recent decisions"}
    ]
  }'
```

#### Behavioral Log Audit

After testing, verify internal behavior via system logs:

```bash
# Check if skill was invoked
openclaw logs --follow --tail 50 | grep -i "memory-search"

# Check if hook fired
openclaw logs --follow --tail 50 | grep -i "session_start"

# Verify no errors
openclaw logs --follow --tail 50 | grep -i "error"
```

### Testing Checklist

Before declaring a skill or plugin "ready":

- [ ] Skill description is clear and concise (single-line preferred)
- [ ] Skill provides guidance, not implementation details
- [ ] All CLI tools have `--help` documentation
- [ ] All CLI tools output natural, readable text
- [ ] Hooks are tested independently (fired at correct events)
- [ ] Agent state is verified (not just claimed)
- [ ] Tool execution is verified (not just claimed)
- [ ] Session messages are reviewed (not just claimed)
- [ ] Edit/Write tool usage is prevented during testing
- [ ] Gateway health is verified before testing
- [ ] Logs are audited for actual behavior
- [ ] All errors are documented and resolved

---

## References

- **Skill Format**: [SKILL.md Format Specification](../openclaw/skill-format.md)
- **Hook Events**: [Hook Events Reference](../openclaw/hook-events.md)
- **Development Patterns**: [CLI Development Patterns](./development.md)
- **Skill Implementation**: [Skill Implementation Guide](./skills.md)
- **Hook Patterns**: [Hook Patterns Guide](./hooks.md)
- **OpenClaw Testing**: [OpenClaw Testing Guide](../openclaw/testing.md)

---

*Created by: Claw 🦞*
*Date: 2026-02-24*
