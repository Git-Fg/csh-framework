# CLI-First Architecture

Build CLI tools first, then wrap with platform plugins. This is the canonical CSH pattern.

## Why CLI-First?

| Benefit | Description |
|---------|-------------|
| **Platform Agnostic** | CLI works everywhere, plugins are thin adapters |
| **Independent Testing** | Test CLI in isolation before platform integration |
| **Progressive Complexity** | Start simple, add plugins as needed |
| **No Lock-in** | Core logic portable across OpenClaw, Claude Code, direct use |

## Architecture Pattern

```
my-project/
├── bin/                    # CLI entry points
│   └── mycli
├── src/                    # Core logic (TypeScript)
├── lib/                    # Shared utilities
├── extension-openclaw/    # OpenClaw plugin wrapper
│   ├── src/index.ts
│   ├── skills/
│   └── hooks/
├── extension-claudecode/  # Claude Code plugin wrapper
│   └── ...
└── packages/              # Shared internal packages
```

## CLI Structure: 5-10-5 Pattern

Optimize cognitive load with command families:

```
Top-level commands: 5 (max 10)
├── vault (5 subcommands)
│   ├── vault search <query>
│   ├── vault read <path>
│   ├── vault list
│   ├── vault remember <type> <title>
│   └── vault forget <path>
├── context (3 subcommands)
│   ├── context session
│   ├── context recent
│   └── context checkpoint
└── admin (2 subcommands)
    ├── admin status
    └── admin reindex
```

**Rule**: Keep top-level commands ≤10, subcommands ≤5 per family

## Template → Edit Hybrid

Avoid shell quoting hell. Use CLI for scaffolding, agent Edit for content:

```bash
# 1. CLI scaffolds template
$ mycli memory create decision "My Decision"
Created template at /tmp/memory-abc123.md

# 2. Agent uses Edit tool natively
# (Edit /tmp/memory-abc123.md with full content)

# 3. CLI validates and processes
$ mycli memory finalize /tmp/memory-abc123.md
✓ Saved to vault with embeddings
```

Benefits:
- No shell quoting issues
- Leverages agent's strongest tool (file editing)
- Agent sees full structure before committing

## State Persistence in CLI

Hooks and CLI can share state via files:

```bash
STATE_FILE="/tmp/myproject_state.json"

# Load state
if [[ -f "$STATE_FILE" ]]; then
  STATE=$(cat "$STATE_FILE")
fi

# Save state
echo "$NEW_STATE" > "$STATE_FILE"
```

## Build System

Use tsup for multi-target builds:

```typescript
// tsup.config.ts
export default defineConfig({
  entry: ['src/index.ts'],
  targets: [
    { format: 'cjs', outDir: 'dist' },           // Node.js
    { format: 'esm', outDir: 'dist/esm' },       // ESM
    { platform: 'node', format: 'cjs', outDir: 'bin' }  // CLI
  ]
})
```

## Testing Strategy

1. **Unit**: Test CLI commands directly
2. **Integration**: Test hooks with sample JSON
3. **Platform**: Test plugin installation

---

## See Also

- [Development](./02-development.md)
- [Skills](./03-skills.md)
- [Hooks](./04-hooks.md)
- [Project Structure](./11-project-structure.md)
