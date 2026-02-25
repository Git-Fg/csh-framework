# Testing Validation Best Practices

> **Validation Principle**: NEVER trust a response that claims a skill or CLI was used without verification; always validate independently
> **Safety Principle**: ALWAYS add strong instructions to prevent unwanted edit tool usage during testing

---

## 🎯 Purpose

CSH Framework plugins and skills must be tested and validated thoroughly. These best practices prevent common issues:

1. **False Positives**: AI agents hallucinate skill usage
2. **Unintended Actions**: Edit tool modifies files when not expected
3. **Validation Gaps**: Testing doesn't verify actual behavior
4. **OpenClaw Integration**: Troubleshooting and project review via chat messages

---

## 📊 Never Trust Agent Claims

### The Hallucination Problem

AI agents frequently report that a skill or CLI tool was used when, in reality, they:
- Misinterpreted a command's output
- Hallucinated skill invocation
- Confused similar-sounding commands
- Reported successful execution when it failed

### Independent Verification Required

**Never** trust an agent's statement like:

- ❌ "The skill was called" (without checking logs)
- ❌ "I used the CLI tool" (without verifying execution)
- ❌ "The hook was triggered" (without checking hook logs)

**ALWAYS** verify independently:

✅ **Check Hook Logs**: Verify hooks actually fired
✅ **Check Agent State**: Query agent state programmatically
✅ **Check Tool Execution**: Verify CLI tool was called
✅ **Check Session Messages**: Review actual conversation messages

### Verification Workflow

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

---

## 🛡️ Always Prevent Edit Tool Usage

### The Unwanted Edit Problem

During testing and development, the **edit tool** (or `Write` tool) should NEVER be used unless:

1. **You explicitly request it** (user says: "please edit file X")
2. **The task requires it** (agent says: "I need to update the file")
3. **It's part of the test case** (automated test scripts)

### Unintended Edit Tool Usage

**When NOT to use Edit/Write**:

**Development**:
- ❌ Do NOT use edit tool for routine skill testing
- ❌ Do NOT use edit tool to "try something"
- ❌ Do NOT use edit tool to "experiment with code"
- ❌ Do NOT use edit tool to "make a quick fix"
- ✅ Use edit tool ONLY for the specific test case

**Testing**:
- ❌ Do NOT use edit tool for skill discovery
- ❌ Do NOT use edit tool for hook verification
- ❌ Do NOT use edit tool to "check if this works"
- ✅ Use edit tool ONLY when the test case explicitly validates file editing

**Review**:
- ❌ Do NOT use edit tool for code review
- ❌ Do NOT use edit tool to "refactor this"
- ❌ Do NOT use edit tool to "improve this"
- ✅ Use edit tool ONLY for intentional modifications (user requested or test requires)

### Best Practices: Preventing Unwanted Edits

#### 1. Strong Anti-Edit Instructions

**In Skills**:

Always include explicit instructions to prevent edit tool usage:

```yaml
---
name: file-operations
description: "⚠️ DO NOT USE EDIT TOOL unless explicitly asked. Perform all file operations using Read, Write, Bash, or Grep. NEVER use Edit tool for file operations."
---

# File Reading

Use Read tool to read files:
```
## Read File: /path/to/file.md
```

**Wrong**: Using Edit tool
```
## Edit File: /path/to/file.md
Change the description to: "This file needs updating."
```

#### 2. Edit Tool Only When Required

**In Skills**:

Only mention Edit tool when the task CANNOT be done any other way:

```yaml
---
name: file-modifier
description: "When file modification is required (user request or test case that validates editing), you may use Edit tool. For all other operations, use Write tool instead."
---

# Correct: User explicitly requests edit
You (User): "Please update the README file to add the new feature."

Agent: "I'll update the README file."
Action: Use Edit tool (acceptable because user explicitly requested)
```

#### 3. Provide Alternative Workflows

**In Skills**:

Always show how to accomplish tasks without using Edit tool:

```yaml
---
name: file-operations
description: "⚠️ DO NOT USE EDIT TOOL unless explicitly asked. Prefer Write tool for file operations. Use Edit tool only when Write tool cannot achieve the goal (e.g., formatting changes, specific line replacements)."

---

# Wrong: Suggesting Edit tool for all file operations
You (User): "Help me organize my files."

Agent: "I'll reorganize your files using Edit tool."
Action: Using Edit tool when Write tool would be better (should have used Write)

Better: Suggest Write tool for file operations
You (User): "Help me organize my files."

Agent: "I can reorganize your files. Here's how to do it:"
Action 1: "Use Bash tool to list files"
Action 2: "Use Write tool to create organized directory structure"
Action 3: "Use Read tool to move files"
```

#### 4. Use Hook Restrictions

**In Hooks**:

Configure hooks to prevent Edit tool usage:

```yaml
---
name: anti-edit-hook
description: "Hook that prevents edit tool usage during testing"
---
# PreToolUse Hook (OpenClaw HOOK.md)

```yaml
---
name: anti-edit
events:
  - session_start
  - command:new
  - PreToolUse
handler: async (event) => {
  // Check if tool is Edit tool
  if (event.tool === "Edit") {
    // Check if agent explicitly requested it
    const requestExplicit = checkForExplicitEditRequest(event.messages);
    
    if (!requestExplicit) {
      // Block Edit tool for all operations during testing
      return {
        blocked: true,
        reason: "Edit tool usage is not allowed during testing unless explicitly requested",
        suggestion: "Use Write tool for file operations instead"
      };
    }
  }
  
  return { allowed: true };
}
```

### Implementation Notes

**Explicit Request Detection**:

Check if the user's messages or the task description explicitly request editing:

```javascript
function checkForExplicitEditRequest(messages) {
  // Look for explicit edit requests
  const editKeywords = ["edit", "update", "modify", "change", "rewrite"];
  
  for (const msg of messages) {
    if (msg.role === "user" || msg.role === "system") {
      const content = msg.content.toLowerCase();
      const hasEditKeyword = editKeywords.some(keyword => content.includes(keyword));
      
      if (hasEditKeyword) {
        // Check if it's a generic request or test-specific
        const isTestSpecific = content.includes("test") || content.includes("validation");
        
        // Generic edit request (e.g., "please edit this file")
        return true;
      }
    }
  }
  
  return false;
}
```

**Test-Specific Detection**:

```bash
# In test case, check if editing is part of the test
if grep -q "test.*edit" <<< "$TEST_DESCRIPTION"; then
  echo "✅ This is a test-specific edit request"
else
  echo "ℹ️ This is a generic edit request, may not need Edit tool"
fi
```

---

## 🧪 OpenClaw Integration: Troubleshooting & Review

### Using Chat Messages for Self-Reflection

OpenClaw gateway can send messages to itself (or to agents) to:
- Reflect on its behavior
- Diagnose issues
- Review CSH project structure
- Self-improve based on feedback

### Best Practices

#### 1. Project Review via Chat

```bash
# Send review message to OpenClaw gateway
openclaw message --channel telegram -m "Review CSH Framework project structure:
1. Are all skills properly located in /root/csh-dev/csh-framework/skills/?
2. Are all hooks properly configured in /root/csh-dev/csh-framework/hooks/?
3. Is the SKILL.md format consistent across all skills?
4. Are there any circular dependencies between skills and hooks?
5. Is the documentation complete and up to date?
6. Are there any deprecated patterns being used?
7. Are the examples concrete and realistic?
8. Are the troubleshooting sections comprehensive?"
```

**Expected Behavior**: OpenClaw processes the review message and provides feedback based on its knowledge of the project.

#### 2. Self-Diagnosis Messages

```bash
# Ask OpenClaw to diagnose issues
openclaw message --channel telegram -m "Self-diagnosis: I'm experiencing issues with skill discovery. Skills are not being detected even though they exist in /root/csh-dev/csh-framework/skills/. Can you check the plugin configuration and logs?"

# OpenClaw responds based on its internal state
# This helps identify configuration issues, permission problems, or environment mismatches
```

#### 3. Project Structure Review

```bash
# Request comprehensive project review
openclaw message --channel telegram -m "Comprehensive review requested:
1. Verify all plugin manifests (openclaw.plugin.json) are valid
2. Check that all SKILL.md files follow the frontmatter specification
3. Verify that all hooks have proper event names and handlers
4. Ensure no deprecated patterns are used (HTTP+SSE, old hook names)
5. Validate that all CLI tools are agent-usable (--help, --json)
6. Review that the documentation is complete and consistent
7. Check for any security issues or exposed credentials
8. Provide recommendations for improvements"

# OpenClaw performs thorough analysis
# Returns detailed report with findings and recommendations
```

### Why This Matters

**Self-Reflection**: Chat messages enable OpenClaw to:
- Review its own code and configuration
- Identify bugs or inconsistencies
- Understand how it's being used in practice
- Improve based on real-world usage

**Project Understanding**: When reviewing CSH Framework alongside its source code:
- OpenClaw can understand the architecture and intended usage
- Can identify mismatches between documentation and implementation
- Can suggest improvements based on both sources

**Iterative Improvement**: Regular reviews lead to better documentation and more reliable plugins

---

## 🔍 Verification Testing

### Test Validation Framework

Create a comprehensive testing framework that validates skills and hooks independently:

```bash
#!/bin/bash
# CSH Framework Validation Tests

echo "=== CSH Framework Validation Tests ==="
echo ""

# Test 1: Skill Discovery
echo "Test 1: Skill Discovery"
SKILL_NAME="memory-search"
echo "Testing skill discovery for: $SKILL_NAME"

# List available skills
SKILLS=$(openclaw skills list | jq -r '.skills[] | jq -r '.name')

# Check if our skill is listed
if echo "$SKILLS" | grep -q "$SKILL_NAME"; then
  echo "✅ Skill is discoverable"
  SKILL_DISCOVERABLE=true
else
  echo "❌ Skill is NOT discoverable"
  SKILL_DISCOVERABLE=false
fi

# Test 2: Skill Invocation
echo ""
echo "Test 2: Skill Invocation"
echo "Sending test message to agent..."
openclaw agent send --channel telegram -m "Test skill invocation: /$SKILL_NAME"

# Wait for response
sleep 5

# Check agent state
echo "Verifying agent state..."
AGENT_STATE=$(openclaw agent get --agent-id "agent-telegram:main" | jq -r '.last_user_message')

# Check if skill was called
if echo "$AGENT_STATE" | grep -q "$SKILL_NAME"; then
  echo "✅ Skill was invoked (verified by agent state)"
  SKILL_INVOKED=true
else
  echo "❌ Skill was NOT invoked (verified by agent state)"
  SKILL_INVOKED=false
fi

# Test 3: Independent Verification
echo ""
echo "Test 3: Independent Verification"

# Don't trust agent claim, verify with logs
echo "Checking logs for skill invocation..."

# Check logs for tool calls
LOGS=$(openclaw logs --follow --tail 20 | grep -i "memory-search")

if [ -n "$LOGS" ]; then
  echo "✅ Tool calls found in logs"
else
  echo "ℹ️ No tool calls found in logs"
fi

# Test 4: Hook Verification
echo ""
echo "Test 4: Hook Verification"
echo "Verifying hook configuration..."

# List configured hooks
HOOKS=$(openclaw config get | jq -r '.plugins.entries | keys[]')

if [ -n "$HOOKS" ]; then
  echo "✅ Hooks are configured"
else
  echo "❌ No hooks configured"
  HOOKS_CONFIGURED=false
fi

# Check for anti-edit hook (if exists)
if echo "$HOOKS" | grep -q "anti-edit"; then
  echo "✅ Anti-edit hook is configured"
  ANTI_EDIT_HOOK=true
else
  echo "ℹ️ Anti-edit hook not configured (recommended for testing)"
  ANTI_EDIT_HOOK=false
fi

# Test 5: Documentation Consistency
echo ""
echo "Test 5: Documentation Consistency"

# Check if documentation is complete
DOCS_EXIST=true

for doc_file in "docs/patterns/skill-description-best-practices.md" "docs/patterns/04-cross-platform.md" "docs/guides/02-plugin-development.md"; do
  if [ ! -f "/root/csh-dev/csh-framework/$doc_file" ]; then
    echo "❌ Missing: $doc_file"
    DOCS_EXIST=false
  else
    echo "✅ Found: $doc_file"
  fi
done

if [ "$DOCS_EXIST" = "true" ]; then
  echo "✅ All required documentation exists"
else
  echo "❌ Some documentation is missing"
fi

# Print summary
echo ""
echo "=== Validation Summary ==="
echo "Skill Discovery: $SKILL_DISCOVERABLE"
echo "Skill Invocation: $SKILL_INVOKED"
echo "Independent Verification: $([ -n "$LOGS" ] && echo true || echo false)"
echo "Hooks Configured: $HOOKS_CONFIGURED"
echo "Anti-Edit Hook: $ANTI_EDIT_HOOK"
echo "Documentation Complete: $DOCS_EXIST"
```

### Validation Checklist

| Test | Description | Pass Criteria |
|------|-------------|-------------|
| **Skill Discovery** | Verify skill is discoverable via OpenClaw | Skill appears in skills list |
| **Skill Invocation** | Verify agent actually calls skill | Agent state shows skill name |
| **Independent Verification** | Check logs for actual tool calls | Log entries found for skill |
| **Hook Verification** | Verify hooks are configured | Hook configuration exists |
| **Documentation Consistency** | Check all required docs exist | All docs found |

---

## 🎓 Best Practices Summary

### 1. Never Trust Agent Claims

- ✅ Verify independently with logs and state queries
- ❌ Don't accept agent statements at face value
- ✅ Cross-reference multiple sources of information
- ✅ Use programmatic verification, not agent assertions

### 2. Prevent Edit Tool Usage

- ✅ Always add strong anti-edit instructions to skills
- ✅ Provide alternative workflows using Write tool
- ✅ Only use Edit tool when explicitly required
- ✅ Configure hooks to prevent unwanted edits during testing
- ✅ Verify hook behavior with independent checks

### 3. Independent Verification

- ✅ Check agent state programmatically (`/v1/agents/state`)
- ✅ Review logs for actual tool invocations
- ✅ Cross-reference agent claims with hook execution logs
- ✅ Don't assume success based on agent responses

### 4. OpenClaw Self-Reflection

- ✅ Use chat messages for project review and diagnosis
- ✅ Ask OpenClaw to analyze its own behavior
- ✅ Review CSH Framework alongside its source code
- ✅ Identify documentation-implementation gaps
- ✅ Improve based on real-world usage patterns

---

## 🚀 Anti-Patterns: Common Issues

### Issue 1: Blind Trust in Tests

**Problem**: Test suite assumes agent's claims are accurate

**Symptom**: Tests pass even when skills aren't actually working

**Solution**:
```bash
# Bad: Trust agent's word
echo "Agent said it used /memory-search, so test passed!"

# Good: Verify independently
echo "Verifying skill was actually invoked..."
if [ "$SKILL_INVOKED" = "true" ]; then
  echo "✅ Test PASSED: Skill was actually invoked"
else
  echo "❌ Test FAILED: Skill was not invoked"
  exit 1
fi
```

### Issue 2: Unintended Edit Tool Usage

**Problem**: Agent uses Edit tool for file operations during testing

**Symptom**: Test files get modified unexpectedly

**Solution**:
```yaml
---
name: file-operations
description: "⚠️ DO NOT USE EDIT TOOL during testing. All file operations must use Read, Write, Bash, or Grep. Edit tool is strictly forbidden for test files."

---
# Test files should use only Read tool for verification
echo "Verifying no edit tool usage..."
grep -r "Edit" /root/csh-dev/csh-framework/test-logs/
if [ $? -eq 0 ]; then
  echo "✅ No edit tool usage found"
else
  echo "❌ Edit tool was used in tests"
  exit 1
fi
```

### Issue 3: False Positives in Logs

**Problem**: Logs show tool calls that didn't happen

**Symptom**: Agent claims "I used CLI tool X" but logs show no invocation

**Solution**:
```bash
# Verify actual log entries
echo "Checking for CLI tool invocations..."
grep -i "my-tool" /root/csh-dev/csh-framework/test-logs/ | tail -10

# Cross-reference with agent claims
# Don't trust agent's statement "I used my-tool"
# Verify with log data
```

---

## 📋 Implementation Checklist

### For Plugin Developers

- [ ] **Add Anti-Edit Instructions**: All skills include strong instructions
- [ ] **Provide Alternatives**: Show how to do tasks without Edit tool
- [ ] **Use Hook Restrictions**: Configure anti-edit hooks in development mode
- [ ] **Independent Verification**: Create validation tests that don't trust agent claims
- [ ] **Log Review**: Implement script to analyze logs for verification
- [ ] **OpenClaw Self-Reflection**: Use chat messages for project review
- [ ] **Documentation Consistency**: Ensure all docs match implementation

### For Testing Framework

- [ ] **Validation Script**: Create comprehensive validation test suite
- [ ] **Test Coverage**: Test skill discovery, invocation, and behavior
- [ ] **Verification**: Cross-reference agent claims with logs and state
- [ ] **Documentation**: Verify examples are concrete and realistic

---

## ✅ Success Criteria

- [x] Never trust agent claims clearly explained
- [x] Always prevent Edit tool usage with strong instructions
- [x] Provide alternative workflows without Edit tool
- [x] Independent verification methodology documented
- [x] OpenClaw self-reflection best practices included
- [x] Verification test framework provided
- [x] Common issues and anti-patterns documented

---

## 📚 Related Documentation

- [OpenClaw Testing Guide](../guides/openclaw-testing-guide.md) - Complete testing workflows
- [Skill Description Best Practices](./skill-description-best-practices.md) - Single-line principles
- [Agent-Usable CLI Tools](./agent-usable-cli-tools.md) - Building reliable CLI tools
- [Cross-Platform Skills](./04-cross-platform.md) - Multi-platform compatibility

---

## 🚀 Next Steps

1. **Implement Anti-Edit Hooks**: Add anti-edit hooks to prevent unintended file modifications
2. **Create Validation Suite**: Build comprehensive test framework with independent verification
3. **Add Review Processes**: Use OpenClaw chat messages for project review and diagnosis
4. **Document Best Practices**: Add more concrete examples and anti-patterns

---

*Created by: Claw 🦞*
*Date: 2026-02-24*
