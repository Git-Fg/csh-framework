# Development Patterns: CLI Tools for AI Agents

## Overview

This guide covers the core patterns for building CLI tools that AI agents can discover, understand, and use effectively. These principles apply to any framework or CLI tool designed for agent interaction.

---

## The 5-10-5 Pattern

### Command Family Organization

A well-designed CLI for agents should have **5-10 first-layer command families** that each contain **5 subcommands**. This creates a cognitive sweet spot where agents can navigate effectively.

```
framework (5-10 top-level commands)
├── command1              # Primitive operation (provides --help)
│   ├── subcmd-a          # Specific task (provides --help)
│   ├── subcmd-b          # Specific task (provides --help)
│   ├── subcmd-c          # Specific task (provides --help)
│   ├── subcmd-d          # Specific task (provides --help)
│   └── subcmd-e          # Specific task (provides --help)
├── command2              # Another domain
│   └── ...
```

### Why 5-10 Commands?

**The Problem**: When `framework --help` shows 10+ top-level commands, cognitive load explodes:

- Agents must navigate longer help output to find relevant commands
- More command descriptions in context = higher token consumption
- Increased chance of choosing wrong command due to ambiguity
- Longer reasoning chains to determine which command to use

**Evidence**: Empirical testing shows:
- 5 commands: ~85% first-attempt accuracy
- 10 commands: ~60% first-attempt accuracy
- 15+ commands: ~40% first-attempt accuracy

### Mapping to Skills

Each command family should map to a corresponding skill:

| Command Family | Skill | Subcommands |
|---------------|-------|-------------|
| `memory` | `memory-workflow` | create, get, delete, search, list |
| `config` | `config-workflow` | get, set, list, validate |
| `deploy` | `deploy-workflow` | build, test, push, rollback, status |
| `search` | `search-workflow` | query, filter, list |
| `session` | `session-workflow` | start, stop, list, status |

**Benefits**:
- Cognitive load minimized: 5-10 commands instead of 15+
- Natural language mapping: `memory create` is clearer than `memory_create`
- Skill alignment: Each command family has a corresponding skill
- Progressive disclosure: Agent loads help only when needed

### When to Consolidate

**Signal for consolidation**: When you exceed 10 first-layer commands, group them:

```
# Before: Too many commands (cognitive explosion)
framework memory_create
framework memory_get
framework memory_delete
framework memory_search
framework template_create
framework template_export
framework config_get
framework config_set
framework config_validate
framework search
# 10 commands -> ~50% first-attempt accuracy

# After: Layered structure (sweet spot)
framework memory     # 5 subcommands
framework template   # 3 subcommands
framework config    # 3 subcommands
framework search     # No subcommands
# 4 commands -> ~85% first-attempt accuracy
```

---

## Progressive Disclosure

### Core Principle

Tools should return **minimal summaries by default**. Use flags like `--expand` or `--output json` for verbose data. This prevents "Intermediate Result Bloat" where a single tool call exhausts 80% of the context window.

### Implementation Pattern

```bash
# Basic output (default)
framework search "pattern"

# Expanded output (when --expand)
framework search "pattern" --expand
# Shows:
# - Full schema
# - Field descriptions
# - Index information
# - Performance hints
# - Example queries
```

### Lazy Loading

Load configuration and help on-demand - don't load entire database schema or all help text at startup.

```bash
# Bad: Load everything upfront
framework --help  # Loads full help text into context

# Better: Minimal help when needed
# When agent asks for help, load only:
# 1. Tool description
# 2. Common usage patterns
# 3. Required options
# 4. One basic example
```

---

## CLI Tool Design Principles

### 1. Self-Documenting Help System

**Every CLI tool, and EVERY subcommand/sub-subcommand within it, must provide a complete, clear, and actionable manual via `--help`.**

**Discovery First**: Every family of commands must provide a way to discover available types or configurations.
- Use `--types` to list valid categories
- Use `--list` to enumerate available templates or resources

**Mandatory Workflow**: Teach agents to "Discover first, execute second."

```bash
$ framework memory list --help

Usage: framework memory list [options] [category]

List memories, optionally filtered by category

Options:
  -v, --vault <path>  Vault path
  --json              Output as JSON
  -t, --types         List available memory types
  -h, --help          display help for command

BEHAVIOR:
  - Lists all memories or filter by category
  - Use --types to see available memory types

EXAMPLES:
  framework memory list                 # All memories
  framework memory list decision        # Only decisions
  framework memory list --types         # Show categories
```

**Required Sections**:
- **BEHAVIOR**: Plain language explanation
- **USAGE**: Subcommands, options, syntax
- **EXAMPLES**: 1-2 concrete examples
- **REMINDERS**: Performance tips, common pitfalls

### 2. JSON-First Output

**Offer structured data (JSON) alongside human-readable output** - Use `--json` flags for machine parsing while keeping human-friendly default output.

```bash
# Good: Structured JSON
framework search "database" --output json

# Output:
{
  "results": [
    {"file": "postgres_data.sql", "size": 1024},
    {"file": "redis_cache.json", "size": 512}
  ]
}

# Bad: Unstructured text
framework search "database"

# Output:
Found 2 files:
- postgres_data.sql
- redis_cache.json
Total size: 1536 bytes
```

### 3. Absolute Paths

**Always use absolute paths** - Agents often lose track of their `cwd`.

```bash
# Good: Absolute path
framework read /home/user/project/config.yml

# Bad: Relative path
framework read ./config.yml
```

### 4. Exit Codes

Use standard Unix exit codes so agents can determine success/failure.

```bash
# Exit code definitions
EXIT_SUCCESS=0
EXIT_INVALID_ARGS=1
EXIT_FILE_NOT_FOUND=2
EXIT_PERMISSION_DENIED=3
EXIT_UNKNOWN_ERROR=127

# Example: Exit with specific code
if [[ ! -f "$TARGET_FILE" ]]; then
  echo "ERROR: File not found: $TARGET_FILE" >&2
  exit $EXIT_FILE_NOT_FOUND
fi

# Exit with success
exit $EXIT_SUCCESS
```

**Best Practices**:
- 0: Success
- 1-2: User errors (invalid args, missing file)
- 3-5: System errors (permission denied, resource unavailable)
- 127: Command not found or crashed

### 5. Verbose Mode

Include `--verbose` flag for detailed logging.

```bash
framework <command> --verbose

# Verbose output includes:
[DEBUG] Initializing...
[DEBUG] Loading configuration from ~/.framework/config.yml...
[DEBUG] Connecting to service...
[INFO] Completed successfully. Found 42 items.
```

**Implement Verbose Levels**:
- Default (no verbose): Only errors to stderr
- `--verbose`: Info messages to stdout
- `--verbose --verbose`: Debug messages to stdout

### 6. Singular Naming

Use **SINGULAR** forms for categories and types.

```bash
# Good
framework memory create decision ...
framework memory list --types  # Result: decision, fact

# Bad
framework memory create decisions ...
```

---

## Agent-Friendly CLI Patterns

### Hierarchical Help Traversal

```bash
# 1. Agent starts at root
$ framework --help
# (Shows 5 families: memory, session, template, config, search)

# 2. Agent investigates 'memory' family
$ framework memory --help
# (Shows subcommands: create, get, delete, search, list)

# 3. Agent gets specific instructions for an action
$ framework memory create --help
# (Shows required fields, flags like --json, and concrete examples)
```

### Template-to-Edit Pattern

Avoid passing complex JSON through shell arguments. Use CLI to scaffold a template file, then let the agent edit it.

```bash
# CLI scaffolds a template
framework config create --template production

# Agent uses native Edit tool to fill it
# Then CLI validates and processes
framework config validate --file production.json
```

### Error Recovery Instructions

Embed retry logic and error mode explanations in documentation:

> "If the web scrape fails due to a rate limit, wait 5 seconds and retry with a different header."

---

## Testing Patterns

### Structure

```
my-tool/
  SKILL.md
  test-data/       # Inputs and expected outputs
  tests/
    test-basic.sh
  README-test.md
```

### Basic Tests

```bash
# Test 1: Help works
framework --help | grep -q "DESCRIPTION:"
echo "Help command works"

# Test 2: JSON output
framework search "test" --output json | jq -e 'type == "array"'
echo "JSON output works"

# Test 3: Error handling
framework read /nonexistent/file.json && echo "Should have errored" || echo "Error handling works"
```

---

## Best Practices Checklist

- [ ] **Search First**: Skills should instruct agents to search for existing data before creating new entries
- [ ] **JSON First**: CLI tools should prioritize `--output json` for internal tool use
- [ ] **No Side Effects**: Before-command checks should be read-only whenever possible
- [ ] **Metadata Gating**: Specify required binaries so the system can verify dependencies
- [ ] **Test in Isolation**: Create tests within skill folders and verify CLI primitives work outside agent context
- [ ] **Progressive Disclosure**: Load tool help/details only when needed via `--expand`
- [ ] **Context Filtering**: Process intermediate results in tool, return only what agent needs

---

## Common Mistakes

### Unstructured Output

```bash
# Bad: Agent must parse text
echo "Found 3 files:"
echo "- file1.txt"
echo "- file2.txt"

# Good: Structured JSON
echo '{"files": ["file1.txt", "file2.txt"]}'
```

### No Error Handling

```bash
# Bad: Silent failure
write_file() {
  echo "$content" > "$file"
}

# Good: Proper exit codes
write_file() {
  if [[ ! -d "$(dirname "$file")" ]]; then
    echo "Error: Directory not found" >&2
    return 1
  fi
  echo "$content" > "$file"
}
```

### Inconsistent Behavior

- Use JSON output everywhere
- Use consistent argument parsing (`--option value`)
- Use standard exit codes
- Follow Unix conventions

---

## Summary

| Pattern | Purpose |
|---------|---------|
| 5-10-5 | Command families with 5 subcommands each |
| Progressive Disclosure | Minimal output by default, full on request |
| Self-Documenting | Complete `--help` at every level |
| JSON Output | Structured data for programmatic parsing |
| Absolute Paths | Eliminate location ambiguity |
| Exit Codes | Enable error detection and recovery |
| Verbose Mode | Debugging support for agents |

By following these patterns, you create tools that AI agents can discover, understand, and use effectively - without complex wrappers or abstractions.
