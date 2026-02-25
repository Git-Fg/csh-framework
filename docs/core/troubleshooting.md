# Troubleshooting Guide

Common issues and solutions when working with the CSH Framework.

---

## Quick Diagnostic Checklist

Before diving into specific issues, verify the basics:

```bash
# 1. Check gateway health (OpenClaw)
openclaw gateway status
openclaw health

# 2. Verify plugin is installed
openclaw plugins list

# 3. Check skills are loaded
openclaw skills list

# 4. Verify hooks are registered
openclaw plugins list | grep -i hook

# 5. Check logs for errors
openclaw logs --follow --tail 20
```

---

## CLI Tools

### Tool Not Found

**Symptom**: Agent reports "command not found" or "tool doesn't exist"

**Diagnosis**:
1. Tool not installed on system
2. Tool not in PATH
3. Agent doesn't know about the tool

**Solution**:
```bash
# Check if tool exists
which mytool

# Add to PATH if needed
export PATH="/path/to/tools:$PATH"

# Verify tool works
mytool --help

# Restart gateway to pick up new tools
openclaw gateway restart
```

### Tool Output Not Parsed

**Symptom**: Agent can't understand tool output

**Diagnosis**:
1. Output not clear/readable
2. Missing helpful error messages
3. Tool returns cryptic messages

**Solution**:
```bash
# Use natural, readable output
mytool list

# Check --help for usage
mytool --help

# Verify output is clear
mytool status
```

### Tool Not Showing in `--help`

**Symptom**: Subcommands don't appear in help output

**Diagnosis**:
1. Help implementation incomplete
2. Subcommands not registered
3. Command group not configured

**Solution**:
```bash
# Test help explicitly
mytool subcommand --help

# Verify command is defined
mytool --help | grep subcommand

# Check command structure
mytool list-commands  # if available
```

---

## Skills

### Skill Not Discovered

**Symptom**: Agent doesn't know about a skill

**Diagnosis**:
1. SKILL.md not in correct location
2. YAML frontmatter malformed
3. Skill disabled in config

**Solution**:
```bash
# Check skill location
ls -la ~/.openclaw/skills/
ls -la my-plugin/skills/

# Verify YAML frontmatter
head -10 my-skill/SKILL.md

# Check if skill is enabled
openclaw skills list | grep my-skill

# Enable if needed
openclaw skills enable my-skill
```

### Skill Not Invoked

**Symptom**: Agent knows about skill but doesn't use it

**Diagnosis**:
1. Skill description unclear
2. Agent doesn't recognize when to use it
3. Mode-switching resistance

**Solution**:
```bash
# Check skill description
head -20 my-skill/SKILL.md

# Improve description (single-line preferred)
# Make it clear when to use:
description: "Use when searching across all project files"

# Add hooks for reminder
# hooks/session-start.sh
echo "Remember to use my-skill when [condition]"
```

### Skill Returns Wrong Results

**Symptom**: Skill executes but produces incorrect output

**Diagnosis**:
1. Skill logic error
2. Wrong CLI command used
3. Input validation missing

**Solution**:
```bash
# Test skill manually
openclaw agent -m "Use my-skill to do X" --local

# Check skill content
cat my-skill/SKILL.md

# Verify CLI command works
mytool command --arg value

# Add validation steps to skill
## Verify
# 1. Command executed successfully
# 2. Output contains expected data
# 3. No errors in logs
```

---

## Hooks

### Hook Not Firing

**Symptom**: Hook configured but never executes

**Diagnosis**:
1. Hook event name incorrect (case-sensitive)
2. Matcher pattern wrong
3. Hook in wrong location
4. Hook disabled

**Solution**:
```bash
# Check hook configuration
openclaw plugins list | grep -i hook

# List all hooks
openclaw hooks list  # OpenClaw
# or
cat ~/.claude/settings.json | jq '.hooks'

# Verify event name
# Use exact event name: "session_start", not "SessionStart" or "sessionstart"
```

**OpenClaw Hook Events**:
- `session_start`, `session_end`
- `command:new`, `command:stop`
- `webhook`, `cron`, `heartbeat`

**Claude Code Hook Events**:
- `SessionStart`, `SessionEnd`
- `PreToolUse`, `PostToolUse`
- `TaskCompleted`, `Stop`

### Hook Returns Error

**Symptom**: Hook executes but returns error code

**Diagnosis**:
1. Script syntax error
2. Missing permissions
3. Environment variables not set

**Solution**:
```bash
# Test hook manually
./hooks/my-hook.sh

# Check permissions
chmod +x hooks/my-hook.sh

# Check environment variables
echo $ENV_VAR

# Debug hook output
set -x  # Enable debugging
./hooks/my-hook.sh 2>&1 | tee /tmp/hook-debug.log
```

### Hook Runs Too Slow

**Symptom**: Hook delays session start or command execution

**Diagnosis**:
1. Hook performing heavy computation
2. Network calls in hook
3. Infinite loop in hook

**Solution**:
```bash
# Profile hook execution
time ./hooks/my-hook.sh

# Add timeout to commands
timeout 5s command || exit 1

# Move heavy work to background
# hooks/session-start.sh
{
  heavy_task &
  echo "Task started in background"
} > /dev/null
```

---

## Plugin Integration

### Plugin Not Loading

**Symptom**: Plugin installed but not active

**Diagnosis**:
1. Manifest file missing or malformed
2. Plugin not installed correctly
3. Gateway not restarted

**Solution**:
```bash
# Check plugin is installed
openclaw plugins list | grep my-plugin

# Verify manifest exists
ls -la my-plugin/openclaw.plugin.json  # OpenClaw
ls -la my-plugin/.claude-plugin/plugin.json  # Claude Code

# Validate manifest JSON
cat my-plugin/openclaw.plugin.json | jq .

# Restart gateway
openclaw gateway restart

# Check logs
openclaw logs --follow --tail 20 | grep my-plugin
```

### Skills Not Available After Plugin Install

**Symptom**: Plugin loaded but skills not discovered

**Diagnosis**:
1. Skills not referenced in manifest
2. Skill requirements not met
3. Skills directory incorrect

**Solution**:
```bash
# Check manifest references skills
cat my-plugin/openclaw.plugin.json | jq '.skills'

# Verify skills directory exists
ls -la my-plugin/skills/

# Check skill requirements
cat my-plugin/skills/my-skill/SKILL.md | head -20

# Test skill discovery
openclaw skills list | grep my-skill
```

### Dependencies Missing

**Symptom**: Plugin requires tools not installed

**Diagnosis**:
1. Binary requirements not met
2. Environment variables not set
3. Platform requirements not satisfied

**Solution**:
```bash
# Check plugin requirements
cat my-plugin/openclaw.plugin.json | jq '.metadata.openclaw.requires'

# Install missing binaries
npm install -g required-tool

# Set environment variables
export API_KEY="value"

# Verify requirements met
which required-tool
echo $API_KEY
```

---

## Testing and Validation

### False Positive Test Results

**Symptom**: Test claims success but feature doesn't work

**Diagnosis**:
1. Trusting agent claims without verification
2. Not checking actual execution
3. Test scope too narrow

**Solution**:
```bash
# NEVER trust agent claims
# Instead verify independently:

# Check logs
openclaw logs --follow --tail 20

# Check state
openclaw agent get --agent-id "agent:main"

# Check files
cat /path/to/output.txt

# Check process
ps aux | grep my-process
```

### Edit Tool Used During Testing

**Symptom**: Files modified unexpectedly during test

**Diagnosis**:
1. Agent not prevented from using Edit/Write
2. Test instructions unclear

**Solution**:
```bash
# Add explicit instructions to skill
## Testing Rules

**CRITICAL**: DO NOT use the Edit or Write tool during testing.

Use ONLY:
- Read tool to inspect files
- Bash/Exec tool to run commands
- Verify tool to check outputs

# Or use hooks to block edit/write
# hooks/pre-tool-use.sh
read -r event
tool=$(echo "$event" | jq -r '.payload.tool')

if [[ "$tool" == "Edit" || "$tool" == "Write" ]]; then
  echo "⚠️ Edit/Write tool blocked during testing."
  exit 1
fi
```

---

## OpenClaw Specific

### Gateway Won't Start

**Symptom**: `openclaw gateway start` fails

**Diagnosis**:
1. Port already in use
2. Configuration error
3. Model not available

**Solution**:
```bash
# Check what's using the port
lsof -i :18789

# Kill conflicting process
kill -9 <PID>

# Check configuration
openclaw config get | jq .

# Test model availability
openclaw health

# Start with debug logging
openclaw gateway start --debug
```

### Session Not Responding

**Symptom**: Agent hangs or times out

**Diagnosis**:
1. Model unreachable
2. Network issue
3. Hook blocking execution

**Solution**:
```bash
# Check gateway status
openclaw gateway status

# Check logs for errors
openclaw logs --follow --tail 50

# Test model endpoint
curl http://127.0.0.1:18789/v1/models

# Restart session
openclaw session stop <session-id>
openclaw session start
```

### Hot Reload Not Working

**Symptom**: Changes to skills not reflected

**Diagnosis**:
1. Plugin not installed in link mode
2. Skills not re-indexed
3. Gateway caching old data

**Solution**:
```bash
# Install with link mode
openclaw plugins install -l /path/to/plugin

# Force re-index
openclaw skills reindex

# Restart gateway
openclaw gateway restart

# Verify changes
openclaw skills list | grep my-skill
```

---

## Getting Help

When all else fails:

1. **Check Logs**: `openclaw logs --follow --tail 100`
2. **Verify Configuration**: `openclaw config get | jq .`
3. **Test Components**: Test each part independently
4. **Search Documentation**: Check related guides
5. **Minimal Reproduction**: Create minimal test case
6. **Report Issue**: Include logs, config, and reproduction steps

---

## Common Error Messages

| Error | Cause | Solution |
|-------|--------|----------|
| `command not found` | Tool not installed or in PATH | Install tool or add to PATH |
| `permission denied` | File permissions incorrect | `chmod +x script.sh` |
| `JSON parse error` | Invalid JSON in output or config | Validate with `jq .` |
| `hook not found` | Hook not registered in manifest | Check plugin manifest |
| `skill not available` | Skill disabled or requirements not met | `openclaw skills enable skill-name` |
| `gateway not running` | OpenClaw gateway stopped | `openclaw gateway start` |
| `model unavailable` | Model endpoint unreachable | Check `openclaw health` |

---

## References

- **Skill Format**: [SKILL.md Format Specification](../openclaw/skill-format.md)
- **Hook Events**: [Hook Events Reference](../openclaw/hook-events.md)
- **Platform Comparison**: [Platform Decision Guide](../openclaw/platform-comparison.md)
- **OpenClaw Testing**: [OpenClaw Testing Guide](../openclaw/testing.md)

---

*Created by: Claw 🦞*
*Date: 2026-02-24*
