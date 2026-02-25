---
title: "Testing Framework Guide"
summary: "Comprehensive testing framework for CSH Framework - agent behavioral validation, not code testing."
read_when:
  - You want to understand the testing approach
  - You are setting up test infrastructure
  - You need testing philosophy
---

# Behavioral Testing Framework

> Testing **how agents behave** with your plugin, not whether code passes.

## What Is This?

This testing framework is about **agent behavioral validation** - ensuring that when your plugin is loaded, the AI agent:

- ✅ Uses the correct skill for each request
- ✅ Chains multiple skills in sequence
- ✅ Handles errors gracefully (no hallucinations)
- ✅ Maintains context across conversation turns
- ✅ Uses hook-injected context appropriately
- ✅ Asks for clarification when ambiguous

The primary tool: **Claude Code as a live testbed**.

---

## Quick Start

1. **Load your plugin in Claude Code**
   ```bash
   claude-code
   # Settings > Plugins > Add Plugin > /path/to/your/plugin
   ```

2. **Run a basic test**
   ```bash
   /new
   User: "Use your-primary-skill with a test request"
   ```

3. **Observe and record**
   - Did agent use correct skill?
   - Was CLI command formatted properly?
   - Were results reported with IDs?

4. **Iterate**
   - Edit SKILL.md or hook code
   - Claude Code auto-reloads
   - Rerun test - see improvement instantly

---

## Documentation Structure

| Document | Purpose | Time to Read |
|----------|---------|--------------|
| **[01-behavioral-testing-approach.md](./01-behavioral-testing-approach.md)** | Philosophy, overview, success criteria | 15 min |
| **[02-claude-code-testbed.md](./02-claude-code-testbed.md)** | How to use Claude Code for testing, config control | 20 min |
| **[03-test-workflows.md](./03-test-workflows.md)** | Practical workflows: single skill, chaining, hooks, edge cases | 30 min |
| **[04-test-cases-templates.md](./04-test-cases-templates.md)** | Copy-paste test cases, templates, logs | Reference |

---

## Test Categories

### 1. Single Skill Validation (5-10 min/skill)
- Skill discovery: Does agent find it?
- Basic invocation: Does it work?
- Error handling: Does it fail gracefully?

### 2. Multi-Skill Behavior (15-20 min)
- Skill selection: Choose correctly when multiple options
- Skill chaining: Sequence skills, maintain context
- Cross-skill data flow: Pass IDs between skills

### 3. Hook Impact (10-15 min/hook)
- Context injection: Is hook content visible?
- Agent usage: Does agent use hook information?
- Hook effectiveness: Which content yields best behavior?

### 4. Edge Cases (15-20 min)
- Ambiguous references ("it", "that")
- Empty results
- Multi-line input
- Partial failures in chains
- Missing dependencies

---

## Example Test Flow

```
1. /new (fresh session)
2. Verify skill loaded: /skills
3. User: "Remember: Use PostgreSQL for production"
   → Should use clawvault-remember
   → Should return ID like "✅ Created memory: d001"
4. User: "Search for database decisions"
   → Should use clawvault-search
   → Should find d001
5. User: "Read that memory"
   → Should use clawvault-read with d001 from context
6. Analyze: Did all steps work? Record pass/fail.
7. /new and retest after any fix.
```

---

## Success Criteria

A plugin is **behaviorally validated** when:

- ✅ All core skills pass single-skill tests
- ✅ Agent correctly selects skills (or asks for clarification)
- ✅ 3+ step workflows complete successfully
- ✅ Hooks inject context that agent actually uses
- ✅ No hallucinations (agent reports actual results only)
- ✅ Error cases handled with specific, actionable guidance
- ✅ Agent maintains context across turns

---

## What This Is NOT

❌ **Not code unit testing** - We don't test individual functions in isolation
❌ **Not automated CI/CD** - This is manual, iterative, human-led
❌ **Not code coverage** - We care about behavior, not line coverage
❌ **Not OpenClaw gateway testing** - Use OpenClaw docs for that

This is **behavioral validation**: The agent is the user; we test what the agent does.

---

## Time Investment

- **Basic validation** (1 skill + hook): 30-45 minutes
- **Full validation** (5+ skills, hooks, edge cases): 2-3 hours
- **Iteration cycles**: Each fix → retest takes 30 sec - 2 min

---

## Complementary Testing

| Testing Type | Tool | Purpose |
|--------------|------|---------|
| **Behavioral** (this) | Claude Code | Validate agent behavior |
| **Code Quality** | ESLint, TypeScript | Catch bugs before testing |
| **CLI Unit Tests** | Vitest/Jest | Test command logic |
| **OpenClaw Integration** | openclaw CLI | Production-like validation |
| **System Testing** | HTTP API, multi-channel | End-to-end with real gateway |

---

## Getting Started Now

1. Pick a **single skill** to test
2. Open **02-claude-code-testbed.md** - learn how to control config
3. Use **03-test-workflows.md** - follow the single-skill workflow
4. Copy templates from **04-test-cases-templates.md** - record your results
5. Iterate until PASS
6. Move to next skill or multi-skill chaining

---

*Ready? Start with [02-claude-code-testbed.md](./02-claude-code-testbed.md).*
