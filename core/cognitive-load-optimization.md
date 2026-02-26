---
title: "Cognitive Load Optimization"
summary: "Design CLI commands and skills for AI cognitive limits - the 5-10-5 pattern."
read_when:
  - You are designing command interfaces
  - You want to maximize AI accuracy
  - You need to understand cognitive load limits
---

# Cognitive Load Optimization

Design CLI commands and skills for human (and AI) cognitive limits.

## The Problem

AI agents (and humans) struggle with many options:

| Commands | Accuracy | Notes |
|----------|----------|-------|
| 3 | 95% | Easy to remember |
| 5 | 85% | Good balance |
| 10 | 60% | Getting difficult |
| 15+ | 40% | Agent ignores / picks randomly |

## The 5-10-5 Pattern

**Rule**: Organize commands into families with ≤10 top-level and ≤5 subcommands each.

### Example: ClawVault

```
clawvault (6 top-level)
├── search <query>       # Semantic search
├── remember <type> <title>  # Create memory
├── read <path>          # Read memory
├── list                  # Browse memories
├── forget <path>        # Delete memory
└── status               # Vault health

context (3 subcommands)
├── context session      # Current session context
├── context recent      # Recent memories
└── context checkpoint # Create checkpoint

admin (2 subcommands)
├── admin reindex      # Rebuild embeddings
└── admin status       # Vault status
```

## Command Family Design

### Group by Purpose

| Family | Subcommands | Purpose |
|--------|-------------|---------|
| `vault` | search, read, list, remember, forget | Core memory ops |
| `context` | session, recent, checkpoint | Session management |
| `admin` | status, reindex, config | Maintenance |

### When to Consolidate

If a family exceeds 5 subcommands, split it:

```
# Before: Too many
vault config set <key> <value>
vault config get <key>
vault config list
vault config reset
vault config export
vault config import  # 6 subcommands - split!

# After: Split into families
vault config set <key> <value>
vault config get <key>
vault config list
vault config reset

backup export
backup import
backup schedule
```

## Skill Mapping

Each command family should map to a skill:

| Command Family | Skill | When Invoked |
|----------------|-------|--------------|
| `vault search` | vault-search | User wants to find memory |
| `vault remember` | vault-remember | User wants to save memory |
| `vault read` | vault-read | User wants to view memory |

## Progressive Disclosure

Show essentials by default, details on request:

```bash
# Default: Brief
$ vault status
Vault: /data/vault
Health: OK
Documents: 1,247

# Expanded: Details
$ vault status --expand
Vault: /data/vault
Health: OK
Documents: 1,247
Embeddings: enabled (256d, ONNX)
Last indexed: 2026-02-25
Index size: 4.2MB
```

## Token Efficiency

Return minimal by default to save tokens:

| Approach | Tokens | Use Case |
|----------|--------|----------|
| `--format json` | High | Machine parsing |
| Default | Medium | Human + AI |
| `--quiet` | Low | Pipelines |

---

## See Also

- [CLI-First Architecture](./08-cli-first-architecture.md)
- [Development](./02-development.md)
