---
title: "Skills Implementation"
summary: "Patterns for implementing and using skills in the CSH Framework - composition, error recovery, and SKILL.md format."
read_when:
  - You are creating a new skill
  - You want to compose multiple skills together
  - You need to write SKILL.md files
---

# Skills Implementation Patterns

This document covers patterns for implementing and using skills in the CSH-Framework, covering composition, error recovery, context optimization, and SKILL.md format.

---

## 1. Skill Composition Pattern

**Problem**: How to build complex workflows from multiple small primitives

**Solution**: Skills compose primitives through file references and sub-skills.

```markdown
---
name: multi-database-query
description: Query PostgreSQL, MongoDB, and Redis for user information

## Prerequisites
- `psql`: PostgreSQL CLI tool installed
- `mongo`: MongoDB CLI tool installed
- `redis-cli`: Redis CLI tool installed

## Step 1: Query PostgreSQL
csh memory get <psql-user-id> --format json

## Step 2: Query MongoDB
csh memory get <mongo-user-id> --format json

## Step 3: Query Redis
csh memory get <redis-key> --format json

## Step 4: Aggregate results
# Combine results in memory and return
---
```

**Key Points**:
- Skills call multiple CLI primitives in sequence
- Each primitive returns structured data (JSON via `--output json`)
- Agent processes results incrementally
- No bloat from loading all tool schemas at once
- Progressive disclosure within skill: Only load primitive-level skills, not full implementation

---

## 2. Error Recovery Pattern

**Problem**: How to handle failures in long-running workflows

**Solution**: Skills include error handling instructions and retry logic.

```markdown
---
name: resilient-web-scrape
description: Scrape web pages with automatic retry and error recovery

## Error Handling
If command fails:
  1. Wait 5 seconds
  2. Retry with exponential backoff
  3. If fails after 3 retries, log and continue

## Success Criteria
- Page successfully scraped
- All data extracted
- No rate limit exceeded

## Workflow
1. Fetch page with `curl`
2. Parse HTML with `pup` (HTML parser)
3. Extract data with `jq`
4. Save to memory: `csh memory create web-data "<url>" --content "$JSON_DATA"`
5. Log errors: `csh remember error "web-scrape:$URL: $ERROR"`

## Hooks Integration
before_command: Verify URL is accessible
after_command: Check for errors and log
---
```

**Key Points**:
- Skills define error modes and recovery strategies
- Hooks provide automation (before/after commands)
- Agent doesn't need to handle errors directly - skill manages them
- Retry logic is embedded in skill instructions

---

## 3. Context Optimization Pattern

**Problem**: How to manage large datasets without exhausting context window

**Solution**: Skills implement filtering, pagination, and data processing in code.

```markdown
---
name: large-dataset-processor
description: Process large CSV files with filtering and analysis

## Workflow
1. Read file in chunks with `head -n 100`
2. Filter rows with `awk`
3. Process and aggregate in memory
4. Clear intermediate variables

## Optimization Techniques
- Streaming: Process in chunks, don't load entire file
- Filtering: Apply `grep` patterns to exclude irrelevant rows
- Aggregation: Use associative arrays or temporary files for intermediate results
- Memory management: Clear variables after each file

## Example Command
```bash
# Process 1M rows with filtering
head -n 1000000 huge-file.csv | \
  awk -F, 'NR==1 && /data_pattern/ {print $0}' | \
  head -n 10000 | \
  awk -F, '/filter_pattern/ {print $0}' | \
  awk 'NR!=1 {print}' | \
  wc -l > ~/memory/processed_1M.txt
```

## Token Savings
- Traditional: Read entire file (50M tokens)
- Optimized: Process in chunks (5K tokens for 100K rows)
- Savings: 90% token reduction
```

**Key Points**:
- Skills perform data transformation in code, not in agent's reasoning
- Intermediate results are processed and filtered before reaching agent
- Memory footprint is dramatically reduced
- Agent can handle arbitrarily large datasets

---

## 4. SKILL.md Format

### Required Frontmatter

Every skill must include YAML frontmatter with:

```markdown
---
name: skill-name
description: One-line description of what this skill does
# Optional fields:
version: 1.0.0
author: team-name
tags: [tag1, tag2]
---
```

### Skill Structure

```markdown
---
name: my-skill
description: Brief description of skill purpose

## When to Use
Use when [specific use case].

## Prerequisites
- Required tool or skill 1
- Required tool or skill 2

## Workflow
1. Step one description
2. Step two description
3. Step three description

## Success Criteria
- Criterion 1
- Criterion 2

## Error Handling
If [error condition]:
  1. Recovery step 1
  2. Recovery step 2
---
```

### Best Practices for SKILL.md

**DO**:
- Keep descriptions concise (1-2 sentences)
- Include specific prerequisites
- Define clear success criteria
- Add error handling guidance

**DON'T**:
- Create monolithic skills that do everything
- Expose dozens of flat tools (MCP style)
- Wrap CLI commands in skills without adding value
- Load entire data into agent context at once

---

## 5. Progressive Disclosure Pattern

**Problem**: How to avoid overwhelming the agent with unnecessary information

**Solution**: Load only minimal description initially, full details only when needed.

```bash
# Step 1: Minimal description
cat > ~/.openclaw/skills/my-skill/SKILL.md << 'EOF'
---
name: data-exporter
description: Export data from multiple databases to CSV format

## When to use
Use when you need to export data from PostgreSQL, MongoDB, and Redis.

## Prerequisites
Skills used: postgres-query, mongo-query, redis-query

## Workflow
1. Query PostgreSQL (via postgres-query skill)
2. Query MongoDB (via mongo-query skill)
3. Query Redis (via redis-query skill)
4. Combine results in memory
5. Export to CSV format
---
EOF

# Step 2: Load sub-skills only when needed
csh skill load postgres-query
csh skill load mongo-query
csh skill load redis-query
```

**Key Points**:
- Use `--sub-skills` to reference other skills by name
- Process intermediate results in code to minimize data passed to agent
- Only load full skill details when task matches
- Massive token savings through selective loading

---

## 6. Testable Skills Pattern

**Problem**: How to ensure skills work correctly without breaking the agent

**Solution**: Design skills to be independently testable.

```
my-skill/
├── SKILL.md
├── test-data/
│   ├── input-1.json
│   ├── input-2.json
│   └── expected-output-1.json
├── tests/
│   ├── test-basic.sh
│   └── test-edge-cases.sh
└── README-test.md
```

```bash
#!/bin/bash

# Run skill test
cd ~/csh/skills/my-skill
csh remember my-skill --test

# Verify output
if [ $? -eq 0 ]; then
  echo "Skill test passed"
else
  echo "Skill test failed"
fi
```

**Key Points**:
- Testing in isolation prevents cascade failures
- Test data provides expected outputs for validation
- Edge cases should be documented and tested
- Skills can be maintained independently without affecting agent

---

## 7. The 5-10 Command-Skill Mapping Pattern

**Problem**: How to design CLI structure that maps naturally to skills

**Solution**: Create 5-10 first-layer commands that map 1:1 to skills.

### Cognitive Load Consideration

When CLI shows 10+ top-level commands, cognitive load explodes:

| Command Count | First-Attempt Accuracy |
|--------------|----------------------|
| 5 commands   | ~85%                |
| 10 commands  | ~60%                |
| 15+ commands | ~40%                |

### Layered Command Structure

```
main_csh (5-10 top-level commands)
├── memory          # Memory operations
│   ├── create     # Create new memory
│   ├── get        # Read memory
│   ├── delete     # Delete memory
│   ├── search     # Search memories
│   └── list       # List memories
├── session         # Session lifecycle
│   ├── start      # Wake with recovery
│   ├── end        # Sleep with handoff
│   └── checkpoint  # Quick save
├── file            # File operations
│   ├── read       # Read file
│   ├── write      # Write file
│   ├── delete     # Delete file
│   └── search     # Search files
└── config          # Configuration
    ├── get        # Get config
    ├── set        # Set config
    └── validate   # Validate config
```

### Command-to-Skill Mapping

| Command Family | Subcommands | Corresponding Skill |
|---------------|-------------|-------------------|
| `memory` | create, get, delete, search, list | `memory-workflow` |
| `session` | start, end, checkpoint | `session-lifecycle` |
| `file` | read, write, delete, search | `file-operations` |
| `deploy` | build, test, push, rollback, status | `deployment-workflow` |
| `config` | get, set, validate | `configuration-management` |

---

## Anti-Patterns

### What NOT to Do

**Overprotective Skills**:
- Don't add validation that checks every parameter
- Don't create opaque skills that hide functionality
- Don't rely on AI to "figure out" poorly documented skills

**Performance Anti-Patterns**:
- Loading entire libraries or frameworks in skills
- Running expensive operations without progress indication
- Not using progressive disclosure
- Ignoring error messages and continuing anyway

---

## Summary

These patterns demonstrate how to build production-ready skills:

1. **Composition**: Skills compose primitives for complex workflows
2. **Resilience**: Error handling and retry patterns prevent cascading failures
3. **Efficiency**: Progressive disclosure and context optimization minimize token usage
4. **Testability**: Independent skills can be tested and validated
5. **Mapping**: 5-10 command structure aligns with cognitive limits

The key is composability: Start simple, compose over time, build a robust skill system.
