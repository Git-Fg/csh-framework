---
title: "Testing Claude Code Plugins"
summary: "How to test Claude Code plugins - behavioral testing, hooks validation, and debugging."
read_when:
  - You want to test plugin behavior
  - You need to validate hooks fire correctly
  - You are debugging plugin issues
---

# Testing Claude Code Plugins

Testing strategies for Claude Code plugins using the CSH framework.

---

## Testing Philosophy

**Behavioral Testing over Unit Testing**: Test what the agent does, not the code itself.

| Approach | What It Tests |
|----------|---------------|
| Unit Tests | Code correctness |
| Behavioral Testing | Agent actions, skill invocation, hook execution |

---

## Hook Testing

### Verify Hook Fires

```bash
# Test a with sample input hook script
echo '{
  "hook": { "event": "PreToolUse", "id": "test" },
  "payload": { "tool": "Bash", "input": { "command": "ls" } }
}' | ./hooks/pre-tool-use.sh
```

### Test All Events

```json
// Test each event type
{
  "hook": { "event": "SessionStart" },
  "payload": { "matcher": "startup" }
}

{
  "hook": { "event": "PreToolUse" },
  "payload": { "tool": "Read", "input": { "file_path": "/test" } }
}

{
  "hook": { "event": "PostToolUse" },
  "payload": { "tool": "Bash", "output": { "stdout": "result" } }
}
```

---

## Skill Testing

### Verify Skill Loads

```bash
# Check skill is discoverable
claude --skills-list | grep my-skill
```

### Test Skill Execution

```markdown
# In a session:
use skill 'my-skill'

# Verify skill content loaded
```

---

## Integration Testing

### Full Plugin Test

```bash
# 1. Install plugin
claude plugin install ./my-plugin

# 2. Run task that should trigger skill
claude "Use my-skill to do X"

# 3. Check logs for:
# - Skill was invoked
# - Hooks fired
# - No errors
```

### Test Checklist

- [ ] Plugin installs without errors
- [ ] Skills appear in skill list
- [ ] Hooks fire at correct events
- [ ] Skill invocation produces expected behavior
- [ ] No errors in logs

---

## Debugging

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Hook not firing | Wrong event name | Check case: `SessionStart` not `session_start` |
| Skill not loading | Missing SKILL.md | Verify file exists in `skills/` directory |
| Script error | Not executable | `chmod +x hooks/*.sh` |
| Invalid JSON | Malformed manifest | Run `jq . plugin.json` to validate |

---

## References

- [Claude Code Hooks](./hooks.md)
- [Claude Code Skills](./skills.md)
- [Testing Approach](../openclaw/behavioral-testing-approach.md)
