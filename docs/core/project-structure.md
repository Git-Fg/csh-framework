---
title: "Project Structure"
summary: "How to organize CSH projects for scale and maintainability - simple vs monorepo patterns."
read_when:
  - You are setting up a new CSH project
  - You need to organize a monorepo
  - You want to understand recommended directory layouts
---

# Project Structure

How to organize CSH projects for scale and maintainability.

## Simple Project

For small tools with single platform:

```
my-tool/
├── bin/
│   └── mytool
├── src/
│   └── index.ts
├── package.json
└── tsup.config.ts
```

## Monorepo with Plugins

For tools targeting multiple platforms:

```
my-tool/
├── bin/                    # CLI entry
│   └── mytool
├── src/                    # Core logic (shared)
│   ├── commands/
│   ├── lib/
│   └── types.ts
├── extension-openclaw/    # OpenClaw plugin
│   ├── src/index.ts       # Plugin entry
│   ├── skills/            # Bundled skills
│   ├── hooks/             # Bundled hooks
│   ├── openclaw.plugin.json
│   └── package.json
├── extension-claudecode/   # Claude Code plugin
│   ├── src/index.ts
│   ├── skills/
│   ├── hooks/
│   ├── plugin.json
│   └── package.json
├── packages/              # Shared internal packages
│   └── ui-lib/
├── package.json           # Root workspace
├── pnpm-workspace.yaml
└── tsup.config.ts
```

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `bin/` | CLI executables |
| `src/` | Core TypeScript logic |
| `lib/` | Shared utilities |
| `packages/` | Internal packages |
| `extension-*/src/` | Plugin entry points |
| `extension-*/skills/` | Bundled skills |
| `extension-*/hooks/` | Bundled hooks |

## Plugin Structure

Each platform extension follows:

```
extension-openclaw/
├── src/
│   └── index.ts          # register(api) function
├── skills/
│   └── my-skill/
│       └── SKILL.md
├── hooks/
│   └── my-hook/
│       ├── HOOK.md
│       └── handler.ts
├── openclaw.plugin.json
└── package.json
```

## Build Configuration

Use tsup for multi-target:

```typescript
// tsup.config.ts
export default defineConfig({
  entry: {
    cli: 'bin/cli.ts',
    lib: 'src/index.ts'
  },
  targets: [
    { format: 'cjs', outDir: 'dist' },
    { format: 'esm', outDir: 'dist/esm' }
  ]
})
```

---

## See Also

- [CLI-First Architecture](./08-cli-first-architecture.md)
- [Development](./02-development.md)
