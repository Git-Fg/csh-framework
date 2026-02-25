# Behavioral Testing Workflows

## Practical Workflows for Different Testing Scenarios

---

## Workflow 1: Single Skill Discovery & Validation

**Purpose**: Verify that a single skill is discoverable and invoked correctly.

**Time**: 5-10 minutes per skill

### Setup

```
Plugin config:
- Skills: [target-skill] ONLY
- Hooks: [none]
- Environment: Set required environment variables
```

### Test Script

```markdown
# Test: {skill-name} - Basic Invocation

1. /new  # Start fresh session

2. User: "[Natural request that should trigger skill]"

   Examples:
   - For clawvault-remember: "Remember this: [decision/fact]"
   - For file-search: "Find files containing 'pattern'"
   - For git-commit: "Commit these changes with message '...'"

3. Observe Claude's response:

   EXPECTED:
   - Mentions skill name (explicitly or implicitly)
   - Shows CLI command being executed
   - Reports success/failure clearly
   - Includes IDs or reference information

   UNEXPECTED:
   - Says "I don't have that skill"
   - Uses general-purpose tools (Write, Edit) instead
   - No CLI command visible
   - Vague response without results

4. Validate output:

   If skill should create resource with ID:
   - ✓ Claude says "✅ Created [resource]: [ID]"
   - ✗ Claude says "✅ Done" with no ID

5. Record result:

   ✓ PASS - skill discovered and used correctly
   ✗ FAIL - [reason]

6. Test edge case: malformed input

   User: "[invalid input for skill]"

   EXPECTED:
   - Claude reports specific validation error
   - Claude suggests correct format
   - No hallucination

   Record:
```

### Iteration

If FAIL:
1. Check SKILL.md description is clear and matches test request
2. Ensure CLI tool has proper `--help` and `--json` flags
3. Add explicit instruction in SKILL.md: "When user asks to X, always use Y"
4. Retest

---

## Workflow 2: Multi-Skill Selection & Disambiguation

**Purpose**: Ensure agent chooses correct skill when multiple options exist.

**Time**: 10-15 minutes

### Setup

```
Plugin config:
- Skills: [skill-a, skill-b, skill-c]  # Have overlapping purposes
- Hooks: [none]
```

### Test Matrix

```markdown
# Test Matrix: Skill Selection

## Scenario 1: Ambiguous Request

User: "Save this important information"

Skills available:
- vault-remember (persistent memory system)
- file-write (writes to text file)
- note-save (saves to notes app)

Expected: Agent asks for clarification or explains differences

Observed: [ ]

Score:
✓ Agent asks: "Which method would you prefer?"
✓ Agent explains: "I can use X for Y, Z for W..."
✗ Agent guesses (e.g., "I'll save it to a file")
✗ Agent says "I can't help with that"

---

## Scenario 2: Clear Intent

User: "Save this to my persistent memory for future reference"

Expected: Agent uses vault-remember (matches "persistent memory")

Observed: [ ]

Score:
✓ Correct skill used
✗ Wrong skill used

---

## Scenario 3: Clear Intent - Alternative

User: "Write this to a quick note file"

Expected: Agent uses file-write

Observed: [ ]

Score:
✓ Correct skill used
✗ Wrong skill used
```

### Disambiguation Pattern Test

```markdown
# Test: Agent Should Never Guess

Config: Multiple skills for same domain

User: "Store this decision"

Bad behavior (✗):
Claude: "I'll store it in the memory system."
[Uses vault-remember without asking]

Good behavior (✓):
Claude: "I have several ways to store information:
        - ClawVault memory system (persistent, searchable)
        - Text file (simple, quick)
        - Notes app (integrates with your workflow)

        Which would you prefer?"

Record: [ ]
```

---

## Workflow 3: Skill Chaining & Context Maintenance

**Purpose**: Test multi-step workflows requiring multiple skills in sequence.

**Time**: 15-20 minutes

### Setup

```
Plugin config:
- Skills: [skill-create, skill-search, skill-read, skill-delete]
- Hooks: [optional - test with and without]
```

### Test Script: Full Lifecycle

```markdown
# Test: Complete Memory Lifecycle

1. /new  # Fresh session

2. User: "Remember this decision: We'll use PostgreSQL for production"
   Claude: Should use skill-create, return ID d001

   ✓ used create skill
   ✓ reported ID d001
   [Record ID for later]

3. User: "Now search for recent database decisions"
   Claude: Should use skill-search, find d001

   ✓ used search skill
   ✓ found d001 in results
   [Record if agent references d001 from context]

4. User: "Read the memory you just found"
   Claude: Should use skill-read with ID d001

   ✓ used read skill
   ✓ used correct ID (from turn 3, not asked explicitly)
   ✓ shows full content

5. User: "Delete that memory"
   Claude: Should use skill-delete with ID d001

   ✓ used delete skill
   ✓ used correct ID (from turn 4 or 3)
   ✗ asks for clarification? [that's OK too - safety first]

6. User: "Verify it's deleted"
   Claude: Should use skill-search, find no results

   ✓ used search
   ✓ reports no results

## Chain Analysis

✓ Context maintained across 6 turns
✓ Agent correctly referenced previous results without explicit ID restatement
✓ Agent used correct skill at each step
✓ No regressions (earlier skills still work)

Result: PASS / FAIL
```

### Multi-Branch Workflow

```markdown
# Test: Branching Logic

1. User: "Remember: We need to evaluate database options"
   → ID: d001

2. User: "Search for past database research"
   → Should find d001? depends on semantics
   [Observe what it finds]

3. User: "Show me the oldest research item"
   → Should use skill-search with sorting? or just show first result?

4. User: "Now remember: We chose PostgreSQL"
   → ID: d002

5. User: "Link d001 to d002"
   → How does agent handle linking? Should use skill-update?

   Observe: [Does agent understand cross-reference?]

## Success Criteria
- Agent picks appropriate skill for each step
- Agent maintains IDs and references across turns
- Agent adapts when information changes
- Agent handles branching (different search results)
```

---

## Workflow 4: Hook Behavior Testing

**Purpose**: Validate that hooks inject useful context and agent uses it.

**Time**: 10-15 minutes per hook

### Hook Discovery

```bash
# First, identify which hooks are registered:
cat /path/to/plugin/openclaw.plugin.json | jq '.hooks'
# OR
cat /path/to/plugin/.claude-plugin/plugin.json | jq '.hooks'
```

Note all hooks and their events.

### Workflow: SessionStart Hook

```markdown
# Test: SessionStart Hook Injection

1. /new  # Start new session (triggers SessionStart)

2. IMMEDIATELY observe Claude's first message:

   Does Claude automatically output something before you type?

   Example hook output:
   "🧠 Memory System Active
    - 3 memories available
    - Skills: remember, search, read"

   ✓ Hook fired and injected context

3. User: "What did the hook just say?"
   Claude: Should be able to reference it

   ✓ Agent has access to hook context
   ✗ Agent says "I don't know what you're referring to"

4. User: "What can I do?"
   Claude: Should use hook context to list available skills/commands

   ✓ Agent uses hook-provided information
   ✗ Agent gives generic response unrelated to hook

## Hook Content Effectiveness

Test different hook contents:

Hook A (generic):
"Skills: remember, search"
→ Agent: "I can help with memory operations."

Hook B (actionable):
"Quick command: clawvault query 'search term'"
→ Agent: "You can search with: clawvault query '...'"

Hook C (contextual):
"Note: 3 memories stored, last: 'Use PostgreSQL'"
→ Agent: "You have 3 memories, latest about PostgreSQL"

Which is most useful? [Observe which agent actually references]
```

### Workflow: PreToolUse Hook

```markdown
# Test: PreToolUse Hook (Validation)

1. Configure PreToolUse hook that inspects commands

2. User: "Remember: X"  (triggers skill, which triggers CLI)

   Hook should fire before CLI runs

   Can hook:
   - Validate command format
   - Add warnings
   - Log activity
   - Modify command

3. Observe: Does hook output appear?
   Does agent mention hook validation?

4. Test hook that blocks:

   Hook condition: Block if command contains "sudo"

   User: "Remember: X"  (no sudo)
   ✓ Should proceed

   User: "sudo remember: X"
   ✗ Hook should block

   Claude: "That command is blocked by security policy."

   ✓ Hook blocking works, agent communicates it
```

### Workflow: PostToolUse Hook

```markdown
# Test: PostToolUse Hook (Analysis)

1. Configure PostToolUse hook that analyzes CLI output

2. User: "Remember: X"

   After CLI runs, hook fires

   Hook could:
   - Extract ID from output
   - Check for warnings
   - Log operation
   - Suggest follow-up actions

3. Observe: Does agent reference hook's analysis?

   Example: Hook extracts "d001" and suggests "Save this ID"

   Claude: "✅ Created memory: d001. (Save this ID for future reference.)"

   ✓ Hook's suggestion appears in agent response
```

---

## Workflow 5: Error Handling & Recovery

**Purpose**: Ensure agent handles failures gracefully without hallucination.

**Time**: 15-20 minutes

### Setup

```
Plugin config: Normal (all skills enabled)
```

### Introduce Failures (One at a Time)

```markdown
# Failure Scenario 1: CLI Tool Missing

1. Temporarily rename/remove required binary:
   mv /usr/local/bin/clawvault /usr/local/bin/clawvault.bak

2. User: "Remember: X"

   Expected:
   - Claude reports "command not found"
   - Claude suggests installing tool
   - Claude does NOT say "✅ Created memory"

   Observed: [ ]

   ✓ Specific error with fix guidance
   ✗ Generic error
   ✗ Hallucinates success

3. Restore binary:
   mv /usr/local/bin/clawvault.bak /usr/local/bin/clawvault

---

# Failure Scenario 2: Vault Not Initialized

1. Set invalid vault path:
   export CLAWVAULT_VAULT=/nonexistent

2. User: "Remember: X"

   Expected:
   - Claude: "Vault not found at /nonexistent"
   - Claude explains: Set CLAWVAULT_VAULT or run 'clawvault init'
   - Claude offers to initialize

   Observed: [ ]

   ✓ File-specific error
   ✓ Fix guidance provided
   ✗ Generic "operation failed"

---

# Failure Scenario 3: Malformed Input

3. User: "Remember: [very long input, or special chars]"

   Expected:
   - CLI validation error
   - Claude reports specific validation message
   - Claude suggests correct format

   Observed: [ ]

---

# Failure Scenario 4: Partial Chain Failure

4. Multi-step:
   - Step 1: remember → succeeds, get ID d001
   - Step 2: search → fails (no index)
   - Step 3: read d001 → ???

   Expected:
   - Claude reports search failure
   - Claude suggests: "I couldn't search, but I can read d001 directly if you have the ID"
   - Claude asks if user wants to continue

   Observed: [ ]

   ✓ Agent reports partial success/failure
   ✓ Agent suggests recovery path
   ✓ Agent doesn't continue blindly
```

---

## Workflow 6: Configuration Experiments

**Purpose**: Test how behavior changes with different configurations.

### Experiment Template

```markdown
# Experiment: {Name}

## Hypothesis
"If I disable {hook} and enable {skill}, agent will {expected behavior}."

## Config A
Skills: [list]
Hooks: [list]
Env: [variables]

Test: [single test case]

Result: [observed behavior]

## Config B (changed X)
Skills: [modified list]
Hooks: [modified list]

Test: [same test case]

Result: [observed behavior]

## Comparison

Did behavior change as expected?
✓ Yes - [explain]
✗ No - [explain]

## Insight
[What did we learn about skill/hook interplay?]
```

### Example Experiments

#### Exp 1: Hook Removal Impact

```
Hypothesis: "Without session_start hook, agent won't know about available skills."

Config A (with hook):
- Hook injects: "Available: remember, search"
- User: "What can I do?"
- Agent: Lists both skills ✓

Config B (without hook):
- Same user query
- Agent: "I have memory-related skills available." (vaguer) ✓

Conclusion: Hook provides concrete examples, agent still knows capabilities
```

#### Exp 2: Progressive Disclosure

```
Test: Start with 1 skill, add skills one by one, test same query

Config: [remember only]
User: "Save this and find similar later"
Agent: "I can save it, but I don't have search."

Config: [remember, search]
User: "Save this and find similar later"
Agent: "✅ Saved. I can search for similar if you'd like."

Config: [remember, search, read]
User: "Save this and find similar, then read the best match"
Agent: Chains all three ✓

Insight: Agent adapts to available skills, progressive disclosure works
```

---

## Workflow 7: Multi-Session & Context Retention

**Purpose**: Test how hooks and state persist across sessions.

**Time**: 10 minutes

### Test: Hook Re-injection

```markdown
1. /new  # Session 1
   Hook fires, injects context C1
   User: "Remember: X"
   Claude: ✅ d001

2. /new  # Session 2 (new session)
   Hook fires AGAIN, injects fresh context C2

   User: "What did we just remember in previous session?"

   Expected behaviors:
   a) If vault is shared: Claude searches and finds d001
   b) If vault is per-session: Claude says "Previous session data not available"
   c) If hook warns about session isolation: Claude explains limitation

   Observed: [ ]

   ✓ Agent's response matches actual vault behavior
   ✗ Agent hallucinates about session isolation
```

### Test: Long-Running Conversation

```markdown
1. Start session, keep active for N turns

2. Perform operations that create state:
   - Remember X (d001)
   - Search (results stored in Claude's context)
   - Read d001 (ID in active memory)

3. Continue with 20+ turns of other conversation

4. Later: "Read that memory again" (referring to d001 from turn 2)

   Expected:
   - ✓ Claude still has d001 in context
   - ✓ Claude reads it correctly
   - ✗ Claude forgets ID and asks which one

   Observed: [ ]

   Note: Claude's context window is limited (~200K tokens).
   Long conversations may lose early context.
```

---

## Workflow 8: Token Efficiency Validation

**Purpose**: Ensure plugin doesn't bloat Claude's context unnecessarily.

**Time**: 5 minutes

### Measure Impact

```markdown
# Before Plugin Loaded

/new
User: "Help me write a function"

# Check token usage (Claude Code may show in status)
# Record: X tokens per message

---

# After Plugin Loaded

/new
User: "Same request: Help me write a function"

# Record: Y tokens per message

## Analysis

Plugin overhead = Y - X tokens

If overhead > 1000 tokens:
- Hook content may be too verbose
- Too many skills loaded (Claude parses all SKILL.md)
- Consider reducing skill descriptions

If overhead < 200 tokens:
✓ Excellent - minimal impact
```

---

## Quick Reference: Testing Checklist

**Before declaring a plugin "behaviorally validated"**:

### Skill-Level
- [ ] Each skill is discovered in appropriate context
- [ ] Agent uses correct skill for intended use case
- [ ] Agent formats CLI command correctly
- [ ] Agent reports IDs/keys for future reference
- [ ] Agent handles edge cases (invalid input, missing data)
- [ ] Skill doesn't interfere with other skills

### Hook-Level
- [ ] Hook fires at correct event (session start, pre-tool, etc.)
- [ ] Hook content is visible to agent
- [ ] Agent actually uses hook information (not just sees it)
- [ ] Hook doesn't bloat context unnecessarily (<200 tokens overhead)
- [ ] Multiple hooks work together without conflict

### Agent Behavior
- [ ] Agent asks for clarification when request is ambiguous
- [ ] Agent doesn't hallucinate (reports actual results only)
- [ ] Agent provides actionable error messages
- [ ] Agent maintains context across conversation turns
- [ ] Agent adapts when skills are enabled/disabled mid-session
- [ ] Agent chains multiple skills correctly

### Error Resilience
- [ ] Missing binary: Agent reports and suggests install
- [ ] Invalid config: Agent reports specific missing setting
- [ ] CLI failure: Agent reports exact error, not generic
- [ ] Partial workflow failure: Agent suggests recovery path
- [ ] Ambiguous reference: Agent asks for clarification

---

## Time Estimates

| Workflow | Time | When to Use |
|----------|------|-------------|
| Single Skill | 5-10 min | Initial skill validation |
| Skill Selection | 10-15 min | When multiple overlapping skills exist |
| Skill Chaining | 15-20 min | Multi-step workflow validation |
| Hook Testing | 10-15 min per hook | After skill validation |
| Error Handling | 15-20 min | Once core functionality works |
| Config Experiments | 20-30 min | Refinement and optimization |
| Multi-Session | 10-15 min | Check state isolation |
| Token Efficiency | 5 min | Performance optimization |

**Total for complete plugin**: ~2-3 hours of iterative testing

---

## What's Next?

After behavioral validation:

1. **OpenClaw Integration Test** (./testing.md)
   - Install in OpenClaw gateway
   - Test via HTTP API
   - Validate multi-channel behavior

2. **Code Quality** (separate from behavioral testing)
   - Lint: `npm run lint`
   - Type check: `npm run type-check`
   - CLI unit tests: `npm test`

3. **User Documentation**
   - Update README with usage patterns discovered
   - Document edge cases and limitations

---

*Workflow Guide: Claude Code Behavioral Testbed*
*Status: Complete reference*
