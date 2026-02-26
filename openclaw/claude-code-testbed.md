---
title: "Claude Code Testbed"
summary: "Using Claude Code as a behavioral testbed for CSH Framework - zero-risk iteration, real-time config, debugging."
read_when:
  - You want to test skills/hooks safely
  - You need a debugging environment
  - You are iterating on plugin behavior
---

# Using Claude Code as a Behavioral Testbed

## Why Claude Code for Testing

### Advantages

1. **Zero-Risk Iteration**
   - Crashes only kill the session, not the gateway
   - Start new session instantly
   - No manual intervention required

2. **Real-Time Configuration Control**
   - Enable/disable skills instantly
   - Toggle hooks on/off
   - No restart needed
   - Changes take effect immediately

3. **Instant Feedback**
   - Send message → observe → fix → retest in seconds
   - No reinstall cycles
   - No gateway restarts

4. **Agent Can Self-Debug**
   - If agent hallucinates, ask it to check logs
   - Agent can analyze its own behavior
   - Safe environment for autonomous troubleshooting

5. **Progressive Disclosure Testing**
   - Start with minimal skill set
   - Add skills one by one
   - Observe how agent adapts
   - Test discoverability patterns

---

## Setting Up the Test Environment

### Start Claude Code with Plugin

```bash
# Option 1: Start fresh and load plugin
claude-code

# Once in Claude Code, load plugin:
# Settings (gear icon) > Plugins > Add Plugin
# Navigate to: /path/to/your/plugin
# Plugin loads automatically

# Option 2: Use Claw to install
# Within Claude Code session:
/claw plugins install /path/to/your/plugin
```

### Verify Plugin Loaded

```bash
# List available skills (Claude Code command)
/skills

# Expected output includes:
# - your-skill-name: Description
# - another-skill: Description

# List loaded plugins
# (Claude Code may show in Settings > Plugins)

# Check plugin manifest
cat /path/to/plugin/.claude-plugin/plugin.json | jq .
```

---

## Real-Time Configuration Control

### Enable/Disable Skills

**Method 1: Settings UI**
```
1. Click gear icon (Settings)
2. Select "Plugins"
3. Find your plugin
4. Toggle individual skills ON/OFF
```

**Method 2: Claude Code Commands** (if supported)
```bash
# Disable a specific skill
/claw skills disable your-skill-name

# Enable a specific skill
/claw skills enable your-skill-name

# List enabled skills
/claw skills list
```

**Instant Effect**: Changes apply to all subsequent messages in current session.

### Enable/Disable Hooks

**Method 1: Settings UI**
```
Settings > Plugins > [Your Plugin] > Hooks
Toggle individual hooks ON/OFF
```

**Method 2: Edit hooks.json**
```bash
# Edit plugin's hooks file
# Claude Code auto-reloads plugin (may take 2-3 seconds)

# To disable hook temporarily:
# Comment it out or remove from hooks.json
```

**Note**: Some hooks (like SessionStart) only fire on session start. To test hook changes:
```
/new  # Start new session to trigger SessionStart hooks
```

---

## Sending Test Messages

### Basic Test Pattern

```bash
# Simple request
User: "Use my-skill to do X"

# Claude Code responds with behavior to observe
Claude: [Response + any CLI commands executed]

# Evaluate: Did Claude behave as expected?
```

### Structured Testing

**Use a test script**:

```markdown
# Test Script: remember-skill-basic

1. Request: "Remember this decision: Use PostgreSQL for production."
   Expected: Claude invokes clawvault memory create decision "..."
   ✓/✗: [ ]

2. Request: "What did I just remember?"
   Expected: Claude searches for recent decision or retrieves by context
   ✓/✗: [ ]

3. Request: "Read that memory by ID."
   Expected: Claude uses ID from previous response
   ✓/✗: [ ]
```

Run sequentially in same session to test chaining.

### Conversation Flow Testing

```
User: Remember: Use PostgreSQL for production.
      (Claude: "✅ Created memory: d001")

User: Search for database decisions.
      (Claude: uses search, finds d001)

User: Read memory d001.
      (Claude: uses read, shows content)

User: Delete that memory.
      (Claude: asks for clarification or uses d001 from context)
```

**Test multi-turn context maintenance**.

---

## Observing Behavior

### What to Look For

#### 1. Skill Discovery
```
Does Claude say: "I don't have that skill" ✓
Or: "I can use X skill for that" ✓
Or: No mention of skill at all? ✗
```

#### 2. CLI Invocation
```
Does Claude show the CLI command? ✓
Is command correctly formatted? ✓
Does it include all required arguments? ✓
Does it use --json when appropriate? ✓
```

#### 3. Result Reporting
```
Does Claude report exit code? ✓
Does Claude show output summary? ✓
Does Claude include IDs/keys for future reference? ✓
Does Claude hallucinate? ✗
```

#### 4. Error Handling
```
When CLI fails, does Claude:
- Report specific error? ✓
- Explain why? ✓
- Suggest fix? ✓
- Offer to retry? ✓
- Hallucinate success? ✗
```

#### 5. Context Usage
```
Does Claude use hook-injected context? ✓
Does Claude reference previous turns? ✓
Does Claude maintain IDs across turns? ✓
Does Claude ask for clarification when ambiguous? ✓
```

---

## Configuration Experiments

### Experiment Matrix

**Goal**: Test behavior across different plugin configurations.

#### Experiment 1: Minimal vs Full

```
Config A (minimal):
- Skills: [remember only]
- Hooks: [none]

Test: "Remember X, then search for it"
Expected: "I don't have search skill"
Result: [ ]

---

Config B (full):
- Skills: [remember, search, read, forget]
- Hooks: [session_start]

Test: "Remember X, then search for it"
Expected: Success - chains remember → search
Result: [ ]
```

#### Experiment 2: Conflicting Skills

```
Config:
- Skills:
  - vault-remember (stores in ClawVault)
  - note-save (saves to Notes app)
  - file-write (writes to file)

Test: "Save this important decision"
Expected behaviors:
A. Agent asks "Which method?"
B. Agent chooses one and explains why
C. Agent uses default (note this)

Observed: [ ]
Score: ✓ if A or B, ✗ if C with no justification
```

#### Experiment 3: Hook Variations

```
Test hook content A:
"Inject: Use semantic search, not grep"

User: "How do I search?"
Claude: References hook → "Use semantic search"

---

Test hook content B:
"Inject: Available skills: remember, search"

User: "What can I do?"
Claude: Lists skills mentioned in hook

---

Compare: Does hook content directly affect agent responses?
✓ Yes → Hook is effective
✗ No → Hook ignored or not understood
```

---

## Edge Case Testing

### Template: Edge Case Test

```markdown
## Edge Case: Ambiguous Reference

**Config**: Full skill set enabled

**Test**:
Turn 1: User: "Remember: Use PostgreSQL"
       Claude: [Should create d001]

Turn 2: (New session with same plugin)
       User: "Delete it"
       (No other context)

**Expected**: Agent asks "What would you like to delete? Please provide memory ID."

**Actual**: [Observe]

**Score**: ✓ if asks for clarification, ✗ if deletes something
```

### Common Edge Cases

1. **Empty Response**: CLI returns nothing
   - Does agent handle gracefully?
   - Does agent say "No results found" or "Empty"?

2. **Partial Failure**: Skill chain breaks at step 2
   - Does agent recover?
   - Does agent report partial success?
   - Does agent continue with remaining steps?

3. **Dependency Missing**: Skill requires binary that's missing
   - Does agent detect and report?
   - Does agent suggest installation?

4. **Conflicting Instructions**: Hook says "use grep" but skill says "use search"
   - Which does agent follow?
   - Does agent notice conflict?

5. **Multi-User Context**: Two users in same session
   - Does agent distinguish?
   - Does agent use "we" correctly?

---

## Iteration Patterns

### Pattern: Observe → Diagnose → Fix → Verify

#### Example: Agent Doesn't Report Memory ID

**Observe**:
```
User: "Remember: Use PostgreSQL"
Claude: "✅ Memory created"

# Where's the ID? Should be "✅ Created memory: d001"
```

**Diagnose**:
```
Ask Claude: "Why didn't you include the memory ID in your response?"

Claude: "The CLI output only said 'Memory created'. I didn't see an ID."

Root cause: CLI doesn't output ID in human-readable mode
```

**Fix**:
```
Option A: Update CLI to always print ID in success output
Option B: Update skill to parse JSON output and extract ID
Option C: Update hook to provide example of expected output

Chosen: B - skill should use --json flag
```

**Verify**:
```bash
# Edit skill description to add: "Always use --json flag"

# Claude Code auto-reloads

User: "Remember: Use PostgreSQL"
Claude: [Now uses --json]
        ✅ Created memory: d001

# Success: ID now included
```

### Pattern: Hook Context Not Used

**Observe**: Hook injects context but Claude doesn't reference it.

**Diagnose**:
```
Ask: "The hook provided information about available skills.
      Did you see it?"

Claude: "I did see it, but I didn't think it was relevant to
         your request."
```

**Fix**:
```
Update hook to be more actionable:
  Before: "Available skills: remember, search, read"
  After: "Quick commands:
          - Remember: clawvault memory create ...
          - Search: clawvault query ...
          Use these for memory operations."

Or update skill description to reference hook context:
  "This skill works with the ClawVault context injected at session start."
```

**Verify**:
```
/new  # New session to get hook

User: "What memory commands can I use?"
Claude: [References hook content directly]

✓ Success
```

---

## Recording and Analysis

### Test Session Log Template

```markdown
# Behavioral Test Session

**Date**: 2026-02-24
**Plugin**: my-plugin v1.2.3
**Claude Code**: Desktop 0.2.71
**Session Duration**: 45 minutes

## Configuration Timeline

| Time | Config Change |
|------|---------------|
| 10:00 | Plugin loaded, all skills enabled |
| 10:05 | Disabled hook "session_start" to test without context |
| 10:15 | Enabled all skills, started edge case tests |
| 10:30 | Modified SKILL.md to add clarification requirement |

## Test Cases Executed

### TC-01: Basic Remember
- Request: "Remember: Use PostgreSQL"
- Expected: Uses clawvault-remember, reports d001
- Result: ✓ PASS
- Notes: First iteration didn't include ID, fixed SKILL.md

### TC-02: Search After Remember
- Request: Chain remember → search
- Expected: Agent maintains context, searches for decision
- Result: ✓ PASS
- Notes: Agent correctly used "recent database decisions"

## Findings

### Critical
1. Agent didn't report ID on first test - fixed by adding instruction to SKILL.md

### Observations
1. Agent naturally chains skills when context is clear
2. Hook context ignored unless explicitly referenced in skill

### Edge Cases Discovered
1. "Delete it" without context → agent asks for ID (good!)
2. Multi-line input → agent uses STDIN correctly

## Next Actions
- [ ] Add TC-03: Error handling when vault not initialized
- [ ] Test with conflicting skills
- [ ] Document pattern: "Agent maintains context across turns"

## Time Breakdown
- Testing: 25 min
- Analysis: 10 min
- Fixes: 10 min
- Total: 45 min
```

---

## Test Case Library

### Template Categories

1. **Single Skill Tests**: Validate each skill independently
2. **Skill Selection Tests**: Multiple options, choose correctly
3. **Chaining Tests**: Multi-step workflows
4. **Hook Tests**: Context injection and usage
5. **Error Tests**: Failure modes and recovery
6. **Edge Cases**: Boundary conditions

### Example: Single Skill Template

```markdown
---
name: skill-{name}-basic
category: single-skill
priority: HIGH

config:
  skills: [{skill-name}]
  hooks: []
  env: {}

requests:
  - prompt: "[Primary use case for this skill]"
    expected:
      - skill: "{skill-name}"
      - cli: "exact command expected"
      - reports-id: true
      - no-hallucination: true

  - prompt: "[Secondary use case]"
    expected:
      - skill: "{skill-name}"
      - handles-error: true

metrics:
  - token-usage: "Should be under X tokens for simple request"
  - latency: "Should respond within 5 seconds"
```

---

## What Success Looks Like

### Behavioral Indicators

✅ **Discovery**: Agent correctly identifies which skill to use
✅ **Execution**: CLI command is correctly formatted
✅ **Reporting**: Results are clearly summarized with IDs
✅ **Error Handling**: Failures are reported with actionable guidance
✅ **Context**: Agent maintains state across conversation turns
✅ **Adaptation**: Agent adjusts when skills are enabled/disabled
✅ **Safety**: Agent asks for clarification when ambiguous
✅ **Autonomy**: Agent can complete multi-step tasks without human intervention

### NOT Success

❌ Code runs but agent behaves incorrectly
❌ Agent hallucinates success
❌ Agent uses wrong skill consistently
❌ Agent loses context between turns
❌ Agent fails to ask for clarification on ambiguous requests

---

## Next Steps

After completing behavioral testing:

1. **Run OpenClaw Integration Tests**
   - `openclaw plugins install -l /path/to/plugin`
   - Test via HTTP endpoint
   - Validate multi-channel behavior

2. **Run Code Quality Checks**
   - `npm run lint`
   - `npm run type-check`
   - `npm test` (if CLI tests exist)

3. **Documentation Update**
   - Update README with behavioral findings
   - Note any configuration requirements
   - Document discovered edge cases

4. **Production Readiness**
   - All high-priority behavioral tests PASS
   - OpenClaw integration validated
   - No critical issues remaining

---

*Document: Claude Code as Behavioral Testbed*
*Status: Active methodology*
