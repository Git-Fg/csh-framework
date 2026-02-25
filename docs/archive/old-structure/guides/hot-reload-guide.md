# CSH Framework Hot Reload & Architecture Advantages

> **Development Workflow**: Use `openclaw -l` for automatic skill reload during development

---

## Quick Start

### Enable Hot Reload in Development

```bash
# Start OpenClaw with livereload
openclaw -l

# Now edit any SKILL.md file
# Changes are detected and reloaded automatically
```

### Benefits

- ✅ **Immediate Feedback**: See changes instantly without restart
- ✅ **Rapid Iteration**: Save-change → reload → test cycle
- ✅ **Faster Development**: No gateway restart needed for skill modifications
- ✅ **Multi-Project Skills**: Skills work globally, not project-scoped

---

## Architecture Advantages

### CSH Framework (OpenClaw) vs. Claude Code

| Feature | CSH Framework | Claude Code |
|----------|-------------|-------------|
| **Hot Reload** | ✅ Automatic with `-l` flag | ❌ Requires gateway restart |
| **Skill Discovery** | ✅ Gateway-managed, auto-discovered | ❌ Plugin-scoped only |
| **Change Detection** | ✅ Real-time file watching | ❌ Requires manual restart |
| **Iteration Speed** | ✅ Fast: save → reload → test | ⚠️ Slower: save → restart → new session |
| **Multi-Project** | ✅ Skills work across projects | ❌ Skills tied to single project |

### Why CSH Enables Hot Reload

**Three-Layer Architecture**:

1. **CLI Primitives**: Direct execution, Unix philosophy
2. **Skills**: Workflow guidance, project-specific rules
3. **Hooks**: Context injection, automation, enforcement

**How It Works**:
- **Rules Files**: Pre-injected at session start (establishes foundation)
- **Hooks**: Dynamic context based on events throughout conversation
- **Skills**: On-demand guidance (loaded only when needed)

This provides:
- ✅ Rich initial context from project configuration
- ✅ Event-driven context from hooks
- ✅ Just-in-time skill guidance (no bloat)
- ✅ **Automatic reload** when files change in dev mode

---

## Development Workflow

### For CSH Framework (OpenClaw)

1. **Enable Livereload**: Use `-l` flag
2. **Make Changes**: Edit SKILL.md files
3. **Automatic Reload**: OpenClaw detects and reloads
4. **Test**: Verify skill works after reload
5. **Iterate**: Repeat quickly without restart

### For Claude Code

1. **No Hot Reload**: Accept longer iteration cycle
2. **Restart Required**: Start new session after changes
3. **Manual Testing**: Each change needs new session to test

---

*See Also*: [Cross-Platform Skills](../patterns/04-cross-platform.md) - Complete compatibility guide
