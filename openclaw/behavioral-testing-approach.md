---
title: "Behavioral Testing Approach"
summary: "Agent-centric validation methodology for CSH Framework - testing agent behavior, skill usage, and context injection."
read_when:
  - You are designing behavioral tests
  - You want to validate agent actions
  - You need testing methodology
---

# Behavioral Testing Approach: Agent-Centric Validation

## Testing Philosophy

### What We're Testing

CSH Framework plugins are not just code - they're **autonomous agent behaviors**. The critical question is: **How does the agent behave when this plugin is loaded?**

We test to answer:
- Does the agent use the correct skill for a given request?
- Does the agent understand skill relationships?
- Does the hook inject useful context?
- Does the agent handle errors gracefully?
- Does the agent maintain context across turns?

### What We're NOT Testing

This methodology does NOT test:
- Code unit tests (function outputs)
- Type checking
- Build processes
- CLI tool correctness (those have their own tests)
- OpenClaw gateway stability

### Core Principle

> **Behavior > Code**: A skill is successful if the agent behaves correctly, not if the code is perfect.

The agent is the ultimate user of your plugin. Test what the agent does, not what the code says.

---

## The Behavioral Testing Methodology

### Overview

**Method**: Use Claude Code as a controlled testbed where you can:
- Load/unload plugins dynamically
- Control which skills and hooks are active
- Send chat messages simulating user requests
- Observe agent behavior in real-time
- Iterate instantly without restarts

**Tool**: Claude Code (not OpenClaw) because:
- Claude Code provides a safe, isolated session
- Crashes don't block development
- Real-time config changes without restart
- Instant feedback loop (seconds vs minutes)
- Agent self-healing (can debug its own mistakes)

**Output**: A validated behavioral specification for your plugin

---

## Testing Strategy

### Progressive Behavior Validation

Test in layers of complexity:

#### Layer 1: Single Skill Discovery
```
Plugin: Only skill X enabled
Test: "Request that matches skill X"
Pass: Agent uses skill X
Fail: Agent doesn't use skill X or uses wrong skill
```

#### Layer 2: Multi-Skill Selection
```
Plugin: Skills X and Y enabled (overlapping purposes)
Test: "Ambiguous request that could use either X or Y"
Pass: Agent asks for clarification or chooses correctly
Fail: Agent guesses without asking
```

#### Layer 3: Skill Chaining
```
Plugin: Skills A, B, C enabled
Test: "Multi-step request requiring A→B→C"
Pass: Agent sequences skills correctly, maintains context
Fail: Agent loses context or uses wrong skill at some step
```

#### Layer 4: Hook Impact
```
Plugin: Skills + hook H
Test: "Start new session and observe hook injection"
Pass: Hook context visible, agent uses it
Fail: Hook doesn't fire or agent ignores context
```

#### Layer 5: Error Recovery
```
Plugin: Skills + intentional failure scenario
Test: "Request that causes CLI tool to fail"
Pass: Agent reports specific error, offers fix
Fail: Agent hallucinates success, generic error
```

---

## Testing Workflow

### Phase 1: Prepare Test Environment

```bash
# 1. Start fresh Claude Code session
claude-code

# 2. Load plugin in development mode
# (Settings > Plugins > Add Plugin > /path/to/plugin)
# OR use Claw to install:
/claw plugins install /path/to/plugin

# 3. Verify skills loaded
/skills
# Should list your plugin's skills

# 4. Check initial config
# Document which skills/hooks are enabled
```

### Phase 2: Baseline Testing

**Goal**: Establish baseline behavior

```
Session:
User: "Request that matches your primary skill"

Observe:
- Did agent discover the skill?
- Did agent format the CLI command correctly?
- Did agent interpret the request correctly?
- Did agent report results with expected details?

Record in test log:
[SKILL] Expected: Uses skill, formats command correctly
[SKILL] Observed: [actual behavior]
[SKILL] Result: PASS / FAIL
[SKILL] Notes: [why it passed/failed]
```

### Phase 3: Configuration Experiments

**Goal**: Test behavior with different skill/hook configurations

```
# Experiment 1: Disable hook
Settings > Hooks > Disable [hook-name]

# Restart session to trigger hook again
/new

User: "Test request that hook should respond to"

Observe:
- Did hook fire?
- Did agent see hook context?
- Did agent use hook information?

Record:

# Experiment 2: Enable conflicting skills
Skills: enable skill-A and skill-B (both claim to handle same domain)

User: "Ambiguous request"

Observe:
- Does agent ask for clarification?
- Does agent guess?
- Does agent explain differences?

Record:
```

### Phase 4: Edge Case Exploration

**Goal**: Find behavioral boundaries

```
Edge Cases to Test:
1. Ambiguous request: "Save this" (multiple save skills)
2. Multi-turn context: "Then search for it" (referring to previous result)
3. Missing requirement: "Use skill" when CLI tool not installed
4. Invalid input: CLI tool receives malformed data
5. Conflicting skills: Two skills with same purpose but different implementations
6. Empty response: CLI tool returns nothing
7. Partial failure: First skill succeeds, second fails
8. Cascading: Skill A outputs ID that skill B needs

For each edge case:
- Set up config
- Send request
- Observe agent behavior
- Ask: "Is this behavior safe and helpful?"

Record:
```

### Phase 5: Iteration

**Goal**: Refine plugin based on observations

```
Iteration Loop:

1. Identify issue in test
   Example: Agent doesn't report memory ID after creating it

2. Determine fix location
   - SKILL.md: Add instruction "Always report the returned ID"
   - Hook: Inject reminder about reporting IDs
   - CLI: Format output to highlight ID

3. Make change (edit file)
   # Claude Code auto-reloads in same session

4. Retest immediately
   User: Same request as before

5. Verify fix
   - Observe improved behavior
   - Document change

6. Repeat for next issue

Time per iteration: 30 seconds to 2 minutes
```

---

## Test Organization

### Test Suite Structure

```
tests/behavioral/
├── 01-single-skill/
│   ├── remember-skill.test.md
│   ├── search-skill.test.md
│   └── read-skill.test.md
├── 02-skill-selection/
│   ├── multiple-save-options.test.md
│   └── ambiguous-domain.test.md
├── 03-skill-chaining/
│   ├── remember-search-chain.test.md
│   ├── search-read-chain.test.md
│   └── full-lifecycle.test.md
├── 04-hooks/
│   ├── session-start-context.test.md
│   ├── pre-tool-validation.test.md
│   └── post-tool-analysis.test.md
├── 05-edge-cases/
│   ├── error-handling.test.md
│   ├── missing-dependency.test.md
│   ├── ambiguous-reference.test.md
│   └── multi-turn-context.test.md
├── 06-configurations/
│   ├── minimal-skill-set.test.md
│   ├── full-skill-set.test.md
│   ├── conflicting-skills.test.md
│   └── progressive-disclosure.test.md
└── templates/
    ├── test-case-template.md
    └── test-session-log.md
```

### Test Case Format

```markdown
# Test Case: [Name]

**Plugin**: my-plugin v1.0.0
**Category**: skill-chaining
**Priority**: HIGH

## Configuration
- Skills enabled: [list]
- Hooks enabled: [list]
- Environment: [env vars]
- Claude Code version: [if relevant]

## Test Scenario

**Request**: "[Exact user message]"

**Expected Behavior**:
- Agent should recognize [skill name] is appropriate
- Agent should invoke CLI: `clawvault memory create ...`
- Agent should report result with memory ID
- Agent should provide summary of what was stored

**Observed Behavior**:
[Paste actual Claude Code response]
[Include CLI commands that were executed]

## Validation

✅ Pass Criteria:
- [x] Agent used correct skill
- [x] CLI command formatted correctly
- [x] Memory ID reported
- [x] Summary provided

❌ Fail Criteria:
- [ ] Agent used different skill
- [ ] CLI command malformed
- [ ] Memory ID missing
- [ ] No summary provided

## Result

**Status**: PASS / FAIL / PARTIAL

**Issues Found**:
- [List any problems]

**Iteration Required**:
- [ ] Yes - [describe fix]
- [ ] No

**Notes**:
- Additional observations
- Edge cases discovered
- Related test cases
```

### Test Session Log

```markdown
# Test Session: [Date] - [Plugin Name]

## Session Setup
- Claude Code instance: [local/remote]
- Plugin version: [git commit or version]
- Test duration: [start] → [end]

## Configuration Changes During Session
[Time] - Disabled hook X
[Time] - Enabled skill Y
[Time] - Modified SKILL.md

## Test Cases Executed

### TC-001: Single Skill Invocation
[Copy-paste test case]

### TC-002: Skill Chaining
[Copy-paste test case]

## Findings Summary

### Critical Issues
1. [Issue] - [Impact] - [Fix needed]

### Improvements
1. [Observations] - [Adjustment]

### Edge Cases Discovered
1. [Edge case] - [Behavior observed]

## Next Actions
- [ ] Fix issue #1
- [ ] Add test for edge case
- [ ] Document pattern in README

## Time Breakdown
- Test execution: X min
- Analysis: Y min
- Fixes: Z min
- Total: T min
```

---

## Success Criteria

A plugin's behavioral testing is complete when:

- ✅ **All core skills** are tested individually
- ✅ **Skill selection** is validated (correct choice or clarification)
- ✅ **Multi-step workflows** work (skill chaining)
- ✅ **Hooks** inject context that agent uses
- ✅ **Error cases** are handled gracefully (no hallucinations)
- ✅ **Edge cases** documented and addressed
- ✅ **Configuration flexibility** confirmed (skills can be enabled/disabled)
- ✅ **Iteration cycle** validated (fixes can be made and tested quickly)

Optional but recommended:
- ⚠️ **Cross-platform**: Behavior validated in both Claude Code and OpenClaw
- ⚠️ **Performance**: Response times documented
- ⚠️ **Accessibility**: Hook context is clear and useful

---

## What This Replaces

This behavioral testing methodology **replaces**:

❌ Bash scripts for OpenClaw CLI commands
❌ Automated test runners
❌ Code coverage tools
❌ CI/CD integration templates
❌ Unit test frameworks

**Because those test the framework, not the agent behavior.**

---

## Complementary Testing

This methodology focuses on **agent behavior**. Complement it with:

1. **Code Quality**: ESLint, TypeScript checking, Prettier
   - Run: `npm run lint`, `npm run type-check`

2. **CLI Tool Tests**: Unit tests for CLI commands
   - Framework: Vitest/Jest
   - Run: `npm test` for command-level validation

3. **OpenClaw Integration**: Production-like testing
   - Install plugin: `openclaw plugins install -l /path/to/plugin`
   - Test via HTTP: `curl -X POST http://127.0.0.1:18789/v1/chat/completions`
   - Verify: Multi-channel, session persistence

**Testing Pyramid**:
```
        Agent Behavioral Testing (Claude Code) - MANUAL, ITERATIVE
       /                              \
      /                                \
     /                                  \
    /                                    \
   /                                      \
  /                                        \
 /                                          \
/____________________________________________\
Code Quality      CLI Tests      OpenClaw Integration (Automated)
```

---

## References

- **Claude Code Integration**: [Claude Code Overview](../claudecode/overview.md)
- **Skill Implementation**: [Skill Implementation Guide](../core/skills.md)
- **Hook Patterns**: [Hook Design Patterns](../core/hooks.md)
- **Best Practices**: [Validation Best Practices](../core/best-practices.md)
- **Case Study**: [ClawVault Behavioral Testing](../examples/clawvault-case-study.md)

---

*Created by: Claw 🦞*
*Date: 2026-02-24 (refactored for behavioral testing focus)*
