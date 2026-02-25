# Test Cases & Templates Library

Ready-to-use behavioral test cases for Claude Code testing.

---

## Single Skill Tests

### Test: Skill Discovery & Basic Invocation

```markdown
# TC-SINGLE-001: Basic Skill Discovery

Plugin: [your-plugin]
Skill: [skill-name]
Category: single-skill

CONFIG:
- Skills: ONLY [skill-name]
- Hooks: []
- Env: [required variables]

TEST SCRIPT:

1. /new
2. User: "[Natural request that should trigger this skill]"

   Example prompts for common skill types:
   - remember: "Remember this: [content]"
   - search: "Search for [query]"
   - read: "Read memory [id]"
   - forget: "Delete memory [id]"
   - init: "Initialize the [tool] system"
   - backup: "Backup everything"

3. Observe response.

EXPECTED:
- ✓ Claude recognizes appropriate skill
- ✓ Claude shows CLI command (or says "using X skill")
- ✓ CLI executes successfully
- ✓ Claude reports result with ID/reference
- ✓ Claude does NOT use Write/Edit tools

ACCEPTABLE:
- Claude asks for clarification before proceeding[still OK - skill triggered]
- Claude reports partial success with recovery path

UNACCEPTABLE:
- ✗ "I don't have a skill for that"
- ✗ Uses Write/Edit instead of CLI
- ✗ Hallucinates success without actual CLI call
- ✗ Generic response with no CLI mentioned

RESULT: [PASS / FAIL]
NOTES:
```

---

### Test: Skill Error Handling

```markdown
# TC-SINGLE-002: Error Handling

Plugin: [your-plugin]
Skill: [skill-name]

CONFIG:
- Skills: [skill-name]
- Introduce intentional error:
  [missing binary / invalid config / malformed input]

TEST SCRIPT:

1. /new
2. User: "[request that triggers error]"

EXPECTED:
- ✓ Claude reports SPECIFIC error (e.g., "command not found", "permission denied")
- ✓ Claude explains WHY (e.g., "binary not in PATH", "config missing")
- ✓ Claude suggests FIX (e.g., "install with npm install -g", "run init command")
- ✓ Claude offers to retry or alternative

UNACCEPTABLE:
- ✗ "An error occurred" (too generic)
- ✗ "I couldn't do that" (no details)
- ✗ Hallucinates success

RESULT: [PASS / FAIL]
NOTES:
```

---

## Multi-Skill Tests

### Test: Skill Chaining (3+ Steps)

```markdown
# TC-CHAIN-001: Full Lifecycle Chain

Plugin: [your-plugin]
Skills: create → search → read → delete

CONFIG:
- Skills: ALL [create, search, read, delete]
- Hooks: []

TEST SCRIPT:

1. /new
2. User: "Remember this decision: Use PostgreSQL"
   Expected: create, returns ID ──→ [RECORD ID]
   ✓ Used correct skill        [ ]
   ✓ Reported ID               [ ]
   ✓ Command formatted correctly []

3. User: "Search for database decisions"
   Expected: search, finds ID from step 2
   ✓ Used search skill         [ ]
   ✓ Found correct item        [ ]
   ✓ Referenced previous ID    [ ]

4. User: "Read that memory"
   Expected: read with ID from step 3
   ✓ Used read skill           [ ]
   ✓ Retrieved full content    [ ]
   ✓ Used correct ID           [ ]

5. User: "Delete that memory"
   Expected: delete with ID from step 4
   ✓ Used delete skill         [ ]
   ✓ Deleted correct ID        [ ]

6. User: "Verify it's gone"
   Expected: search again, no results
   ✓ Search returns empty      [ ]

RESULT: PASS / FAIL

FAILURE ANALYSIS:
- [ ] Lost ID between turns
- [ ] Used wrong skill at step X
- [ ] Did not chain (repeated same skill)
- [ ] Hallucinated results
```

---

### Test: Skill Selection (Ambiguous Request)

```markdown
# TC-SELECT-001: Disambiguation Required

Plugin: [your-plugin]
Skills: [skill-A, skill-B]  # Overlapping purposes

CONFIG:
- Skills: [skill-A "persistent save"], [skill-B "quick file save"]
- Hook: None

TEST SCRIPT:

1. /new
2. User: "Save this important information"

EXPECTED:
A. Agent asks for clarification:
   "I can save in different ways:
    - A: persistent memory (searchable)
    - B: text file (temporary)
    Which?"

   ✓ Asks clarification     [ ]

OR

B. Agent chooses and EXPLAINS why:
   "I'll use [skill-A] because you said 'important',
    which suggests long-term storage."

   ✓ Explains choice        [ ]

UNACCEPTABLE:
✗ "I'll save it to a file" (guesses without asking)
✗ "I can't help with that" (aborts)

RESULT: PASS / FAIL
NOTES:
```

---

## Hook Tests

### Test: SessionStart Hook Context Injection

```markdown
# TC-HOOK-001: SessionStart Context Visible

Plugin: [your-plugin]
Hook: session_start

CONFIG:
- Skills: [any]
- Hook: session_start with content:
  """
  [HOOK CONTENT TO TEST]
  Example: "Memory System: 3 memories available
            Skills: remember, search, read"
  """

TEST SCRIPT:

1. /new  (triggers hook)

2. IMMEDIATELY observe Claude's first automatic message

   Does it contain hook content? Yes / No

   If Yes:
   ✓ Hook fired and injected    [ ]

3. User: "What did the hook just tell me?"
   Claude should be able to reference:
   ✓ Hook content recalled       [ ]
   ✓ Accurate reproduction       [ ]

4. User: "What skills are available?"
   Claude should use hook content if it listed skills

   ✓ Uses hook info             [ ]
   ✗ Gives different answer     [ ]

5. OPTIONAL: Test hook content effectiveness

   Hook content variations:
   A. "Skills: remember, search"
   B. "Commands: clawvault memory create, clawvault query..."
   C. "3 memories, latest 'Use PostgreSQL'"

   For each, ask: "How do I [action]?"
   Which hook content yields most useful agent response?

RESULT: PASS / FAIL
HOOK EFFECTIVENESS SCORE: 1-10
NOTES:
```

---

### Test: PreToolUse Hook Validation

```markdown
# TC-HOOK-002: PreToolUse Interception

Plugin: [your-plugin]
Hook: pretooluse (or platform equivalent)

CONFIG:
- Hook that inspects commands before they run
- Block if: command contains "sudo" OR certain keywords

TEST SCRIPT:

1. /new (hook active)

2. User: "Remember: Normal content"
   Expected: ✓ Proceeds normally

   Observed: [ ]
   Hook output visible? [ ]

3. User: "sudo remember: Evil content"
   Expected: ✗ Hook blocks, Claude reports

   Observed: [ ]
   Blocked? [ ]
   Claude communicates block? [ ]

RESULT: PASS / FAIL
NOTES:
```

---

## Edge Case Tests

### Test: Ambiguous Reference ("it", "that", "the one")

```markdown
# TC-EDGE-001: Ambiguous Reference Resolution

Plugin: [your-plugin]
Skill: any that returns IDs

CONFIG:
- Full skill set

TEST SCRIPT:

1. /new
2. User: "Remember: Decision A"
   Claude: ✅ d001
   [Record: ID = d001]

3. User: "Read it"

   Expected:
   A) ✓ Claude uses d001 from context
   B) ✗ Claude asks "What would you like to read?"

   Observed: [ ]

4. User: "Now remember: Decision B"
   Claude: ✅ d002

5. User: "Delete that"  (could be d001 or d002)
   Expected:
   A) ✓ Claude asks for clarification: "Which memory, d001 or d002?"
   B) ✗ Claude deletes d002 (assumes latest)
   C) ✗ Claude deletes d001

   Observed: [ ]

RESULT: PASS / FAIL
SAFETY CHECK: ✓ Agent asked for clarification (not FAIL, but note safety)
NOTES:
```

---

### Test: Empty or No Results

```markdown
# TC-EDGE-002: Empty Result Handling

Plugin: [your-plugin]
Skill: [search]

CONFIG:
- Search skill
- Vault with NO matching data

TEST SCRIPT:

1. /new
2. User: "Search for: [something definitely not in vault]"

   Expected:
   ✓ Claude: "No results found"
   ✓ Claude: "I searched but couldn't find any matches"
   ✗ Claude: "Found 0 results" (technical)
   ✗ Claude: "I don't have search functionality" (wrong)

RESULT: [ ]
NOTES:
```

---

### Test: Multi-Line Input

```markdown
# TC-EDGE-003: Multi-Line Content

Plugin: [your-plugin]
Skill: [create/remember]

CONFIG:
- Skill that accepts multi-line input

TEST SCRIPT:

1. /new
2. User: "Remember this:

   Line 1
   Line 2
   Line 3
   More details"

   (Use Enter for newlines in chat)

   Expected:
   ✓ Claude passes multi-line correctly to CLI
   ✓ CLI succeeds with exit code 0
   ✓ Claude reports ID

   Observed: [ ]

   Check CLI: Did it receive full content or truncated?
   Ask Claude: "What exactly did you send to the CLI?"

RESULT: PASS / FAIL
NOTES:
```

---

## Configuration Experiments

### Experiment: Hook Content Variation

```markdown
# EXP-HOOK-001: SessionStart Hook Content Impact

HYPOTHESIS: "Different hook content leads to different agent behavior."

CONFIG A:
Hook: "Available skills: remember, search, read"

TEST: /new
User: "How do I search?"

EXPECTED:
- Claude might say: "You can use the search skill..."
- Claude might NOT mention specific command

RESULT A: [observed]

---

CONFIG B:
Hook: "Commands: clawvault memory create, clawvault query"

Same user query:

EXPECTED:
- Claude likely: "Use: clawvault query 'your search'"
- More specific command syntax

RESULT B: [observed]

---

CONFIG C:
Hook: "You have 3 memories. Use clawvault-search for semantic search."

User: "Search for database decisions"

EXPECTED:
- Claude may reference count: "Searching among 3 memories..."
- Claude may use skill name "clawvault-search"

RESULT C: [observed]

---

ANALYSIS:
Which hook content produced most actionable agent responses?
Rank: A / B / C

INSIGHT:
[What did we learn about hook content effectiveness?]
```

---

## Test Execution Log Template

```markdown
# Behavioral Test Session Log

SESSION: [Number/Date]
TESTER: [Name]
PLUGIN: [name] v[version]
CLAUDE CODE: [version if known]
DURATION: [start] - [end]

---

## CONFIGURATION TIMELINE

[Time] Initial load: skills=[list], hooks=[list]
[Time] Change: disabled skill-X
[Time] Change: modified hook content
[Time] ...

---

## TEST RESULTS

### TC-SINGLE-001: Basic Discovery
Status: PASS / FAIL
Observed:
[Paste Claude's actual response]
Notes:
[Why it passed/failed]

### TC-CHAIN-001: Lifecycle Chain
Status: PASS / FAIL
Step 1: [ ]
Step 2: [ ]
Step 3: [ ]
Notes:

---

## FINDINGS

### Critical Issues
1. [Issue] - Impact: [High/Medium/Low] - Fix: [action needed]

### Observations
- [Pattern observed]
- [Unexpected behavior noted]
- [Strength identified]

### Edge Cases Discovered
1. [Edge case description]

---

## ACTIONS

- [ ] Fix critical issue #1
- [ ] Document pattern in README
- [ ] Add new test case for edge case
- [ ] Update SKILL.md with clarification

---

## TIME TRACKING

Test execution: ___ minutes
Analysis: ___ minutes
Fixes: ___ minutes
Documentation: ___ minutes
Total: ___ minutes
```

---

## Automated Test Script Template

For running multiple tests sequentially:

```bash
#!/bin/bash
# behavioral-test-runner.sh

# Set up
PLUGIN_PATH="/path/to/plugin"
TEST_CASES=("tc-single-001" "tc-chain-001" "tc-hook-001")
SESSION_LOG="/tmp/test-session.log"

# Function to send message to Claude Code via API or manual
# (Implementation depends on your Claude Code access method)

run_test() {
  local test_name=$1
  local test_file=$2

  echo "=== Running: $test_name ===" >> $SESSION_LOG
  echo "" >> $SESSION_LOG

  # Extract test script from markdown
  # (Simplified - you'd parse the markdown file)

  # For manual testing: Display instructions
  cat $test_file | grep "^## TEST SCRIPT:" -A 20

  echo ""
  echo "After running, record result:"
  read -p "PASS/FAIL? " result
  echo "$test_name: $result" >> $SESSION_LOG

  # Prompt for notes
  read -p "Notes: " notes
  echo "Notes: $notes" >> $SESSION_LOG
  echo "" >> $SESSION_LOG
}

# Main
for test in "${TEST_CASES[@]}"; do
  run_test "$test" "docs/test-cases/$test.md"
done

echo "Session log saved to: $SESSION_LOG"
```

---

## Quick Test Templates (Copy-Paste)

### Minimal Single Test

```
/new

[Paste test case from above]

Record result:
- ✓ PASS: [ ]
- ✗ FAIL: [reason]

 iterate: fix → retest
```

### Batch Testing

```
Prepare N test cases
Execute each:
1. /new
2. Copy test script prompts
3. Send to Claude
4. Observe and record
5. /new for next test

After batch:
- Review failures
- Prioritize fixes
- Re-run failed tests
```

---

## Test Case Prioritization

### Must Test (All Plugins)
- [ ] TC-SINGLE-001: Basic skill discovery
- [ ] TC-SINGLE-002: Error handling
- [ ] TC-SELECT-001: Ambiguous request handling
- [ ] TC-CHAIN-001: At least one 3-step chain
- [ ] TC-HOOK-001: SessionStart hook (if hook exists)
- [ ] TC-EDGE-001: Ambiguous reference

### Should Test (Recommended)
- [ ] TC-EDGE-002: Empty results
- [ ] TC-EDGE-003: Multi-line input
- [ ] TC-EDGE-004: Partial chain failure
- [ ] Hook variations if multiple hooks

### Nice to Test (If Time)
- [ ] Token efficiency measurement
- [ ] Multi-session retention
- [ ] Long conversation context
- [ ] Configuration experiments

---

## Common Failure Patterns & Fixes

### Pattern 1: Agent Says "I Don't Have That Skill"

**Cause**:
- Skill description is unclear
- Request phrasing doesn't match skill description
- Skill not actually loaded (check /skills)

**Fix**:
1. Verify skill is in `/skills` list
2. Update SKILL.md `description:` to be more explicit
3. Add "When to use" section with exact phrase matches
4. Test with simpler request first

---

### Pattern 2: Agent Uses Write/Edit Instead of CLI

**Cause**:
- Skill description mentions "write" or "store" generally
- Agent thinks Write tool is appropriate
- No clear instruction to use specific CLI

**Fix**:
1. In SKILL.md, add: "MUST USE the [tool] CLI command"
2. Add: "DO NOT use Write or Edit tools"
3. Make skill description: "Use [tool] to do X"
4. Test: Agent should now show CLI command

---

### Pattern 3: Hallucinates Success When CLI Fails

**Cause**:
- CLI error output not captured
- Agent doesn't check exit code
- Skill doesn't instruct "verify success"

**Fix**:
1. Update skill: "After running command, check exit code with `echo $?`"
2. Add to skill: "If exit code is not 0, report the error"
3. Consider: CLI should output JSON for easier parsing
4. Test: Introduce controlled failure, verify agent reports it

---

### Pattern 4: Loses Context Between Turns

**Cause**:
- Long conversation (>50 turns)
- Claude's context window overflow
- Agent doesn't explicitly record IDs

**Fix**:
1. In skill, add: "Always include the ID in your response"
2. Add instruction: "If user references 'that' or 'it', check your last response for ID"
3. Consider: Hook can inject reminder: "Remember to preserve IDs"
4. Limit test conversations to reasonable length (<30 turns)

---

### Pattern 5: Doesn't Use Hook Context

**Cause**:
- Hook content too verbose or vague
- Hook content appears after agent starts responding
- Agent prioritizes real-time reasoning over hook

**Fix**:
1. Reduce hook content to ≤3 lines
2. Make hook content actionable: "Use command X" not just "Skill X exists"
3. Place hook at absolute beginning of session (SessionStart)
4. Test: Ask agent "What did the hook tell you?" to ensure visibility

---

## Performance Benchmarks

### Measure Behavioral Performance

```markdown
# Performance Check

TEST: Simple skill invocation

1. /new
2. User: "Remember: test message"

MEASUREMENTS:
- Time to first token: ___ seconds
- Time to complete: ___ seconds
- Tokens used in turn 1: ___
- Tokens used in turn 2 (response): ___

ACCEPTABLE:
- Latency: <5 seconds for simple operations
- Token overhead of plugin: <500 tokens per turn

If slower:
- Hook content may be too long
- Too many skills loaded (Claude parses all SKILL.md)
- Consider reducing skill descriptions
```

---

## Documentation Update Checklist

After behavioral testing, update:

- [ ] **SKILL.md**: Refine description based on what phrasing triggers skill
- [ ] **SKILL.md**: Add error handling guidance if discovered needed
- [ ] **HOOK**: Adjust content for clarity and actionability
- [ ] **README**: Document discovered edge cases
- [ ] **README**: Add usage examples from successful test flows
- [ ] **CHANGELOG**: Note behavioral improvements

---

## Complete Test Plan Template

```markdown
# Test Plan: [Plugin Name]

## Objectives
- Validate agent discovers and uses each skill correctly
- Ensure agent handles errors without hallucination
- Confirm hooks inject useful context
- Test multi-step workflow completion

## Test Schedule

Day 1: Single skill validation (5 skills × 10 min = 50 min)
Day 2: Multi-skill chaining (20 min)
Day 3: Hook testing (15 min)
Day 4: Edge case exploration (30 min)
Day 5: Configuration experiments (30 min)
Day 6: Iteration and fixes (60 min)
Day 7: Re-run failures, final validation (45 min)

Total: ~5 hours spread across iterations

## Success Criteria

All Must-Test items PASS with no critical failures
No hallucination in any test case
All skills: discovery > invocation > error handling validated
Hooks: context visible and used by agent

## Sign-off

Tester: ________________ Date: ______
Plugin Author: ________________ Date: ______
Behavioral Test Complete: Yes / No
Ready for OpenClaw Integration: Yes / No
```

---

*Test Cases Library: Claude Code Behavioral Testing*
*Status: Ready to use - copy templates into your Claude Code session*
