# CLI Design Patterns for CSH Primitives

Guidelines and conventions for building CLI tools that AI agents can use effectively as CSH Primitives.

---

## Command Design Principles

### The 5-10 Command Sweet Spot

When building your plugin's CLI, aim for a balanced command structure to minimize cognitive load on the agent.

| Category | Count | Example Commands |
|----------|-------|------------------|
| Core | 5-10 | init, status, help, version |
| Resource | 5-10 | create, list, get, update, delete |
| Action | 5-10 | start, stop, run, build, deploy |

### Command Hierarchy (Pattern)

Standardize your subcommands using a `<noun> <verb>` pattern:

```bash
# Recommended Pattern
mytool plugin list
mytool skill create
mytool hook register
```

---

## Pattern: Standard Commands

While you build your own tools, implementing these standard commands ensures agents can explore your toolset predictably.

### help / --help

Your tool MUST provide a complete manual via the `--help` flag. This is how agents learn to use your tool as a primitive.

```bash
mytool --help
mytool <command> --help
```

### list

Provide a way to list available resources or categories.

```bash
mytool list plugins
mytool list skills
mytool list --types
```

### status

Show the current state of the system or a specific resource.

```bash
mytool status
mytool status <id>
```

---

## Output Formats

### Natural Language Output

CLI tools should output clear, readable text that AI agents can understand naturally. Agents are trained on natural language, not JSON parsing.

```bash
# Good: Natural, readable output
mytool list plugins
# Output:
# Available plugins:
# - my-plugin v1.0.0 (enabled)
# - other-plugin v2.1.0 (disabled)

# Total: 2 plugins
```

Avoid requiring JSON parsing for normal operations. Keep CLI simple.

---

## Exit Codes

Use standard exit codes so the agent (or your Hooks) can programmatically determine success.

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Generic error |
| 2 | Invalid usage (agent sent wrong parameters) |
| 4 | Not found |
| 5 | Permission denied |

---

## Example Implementation (Commander.js)

```javascript
#!/usr/bin/env node
import { Command } from 'commander';

const program = new Command();

program
  .name('mytool')
  .description('A CSH-compliant CLI primitive');

program.command('status')
  .description('Show tool status')
  .action(() => {
    console.log('Tool is healthy');
  });

program.parse();
```

---

## Environment Variables (Portability)

When building portable CSH primitives, leverage standard environment variables provided by the host platforms instead of hardcoded paths.

| Variable | Platform | Description |
|----------|----------|-------------|
| `${CLAUDE_PLUGIN_ROOT}` | Claude Code | Absolute path to the plugin directory |
| `${OPENCLAW_WORKSPACE}` | OpenClaw | Path to the active workspace |
| `$HOME` | Universal | User home directory for persistent state |

