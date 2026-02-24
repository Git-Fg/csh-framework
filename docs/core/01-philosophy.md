# Why CLI + Skills + Hooks?

The CSH Framework provides a architecture for building AI agent extensions that scale.

---

## The Core Problem Being Solved

### Traditional Approach (MCP)
- Every agent loads all tool definitions into context (100+ tools)
- Token consumption explodes (50,000+ tokens for simple tasks)
- Intermediate results bloat context (50K tokens from document retrieval)
- Fixed tool schemas restrict agent flexibility

### CSH Framework Approach
- **Progressive disclosure**: Load only what's needed when it's needed
- **Up to 98% token reduction** in production scenarios
- **Agent autonomy preserved**: Can always bypass skills and use primitives directly
- **Skills teach patterns**, don't abstract away capabilities
- **Hooks automate** repetition and provide context without blocking agent

---

## Current Limitation: Mode Switching is Expensive

> **Important**: Current LLMs struggle with skill invocation even when descriptions are perfectly elicited.

- Strong description ("YOU MUST USE THIS SKILL"): ~30% invocation rate
- Task-skill match (based on description): ~92%
- Gap lost to mode-switching resistance: ~65 percentage points

This isn't a skill design flaw — it's how current models were trained. They excel at pattern matching (recognizing *when* skill is relevant) but struggle with mode switching (actually *invoking* skill and loading context).

**What This Means**: Hooks aren't an architectural ideal — they're a pragmatic workaround for current model limitations.

---

## Why Three Layers?

### Layer 1: CLI Primitives (The Foundation)

**Purpose**: Single, focused operations (CRUD, file ops, shell commands)

**Design Principles**:
- Self-documenting (`--help` provides complete manual)
- Absolute paths for agent-usable output
- Progressive disclosure by default (minimal context, `--expand` for verbose)
- Discovery commands (`--list`, `--types`) for exploring capabilities

**What This Provides**:
- Universal tool access (git, aws, docker, kubectl, curl, jq, etc.)
- Zero implementation overhead (tools maintained by service providers)
- Text-based interfaces (universal composability via pipes)
- Direct filesystem access for state persistence
- Battle-tested reliability (decades of Unix tooling)

**Key Insight**: LLMs have been trained on vast amounts of shell scripts, documentation, and command examples. They "know" how to use CLI tools naturally without additional abstraction layers.

---

### Layer 2: Skills (Workflow Orchestrators)

**Purpose**: Teach agents HOW to use primitives correctly for complex workflows

**Design Principles**:
- Skills teach patterns, they don't replace or abstract CLI commands
- Progressive disclosure within skills: Only load skill details when task matches
- Agent can ignore skills (freedom preserved), but skills make right choice obvious
- File-based: SKILL.md with YAML metadata + markdown instructions
- Token efficient: Don't bloat context until needed

**What This Provides**:
- Domain expertise encoding (medical diagnosis, legal research, code review patterns)
- Multi-tool orchestration (step-by-step workflows with clear success criteria)
- Best practice encoding (formatting standards, error handling patterns)
- Reusable capabilities that compound over time (first execution: 12K tokens → skill creation → subsequent: 4K tokens)

**Critical Design Rule**: SKILLS MUST NEVER ABSTRACT AWAY PRIMITIVES. They must make using CLI primitives more natural, efficient, or correct — not hide them or add "magic" wrappers.

**Example**: A "git-commit-workflow" skill teaches: check git status, stage files, create commit with conventional message, push, verify. Agent learns the pattern, not just the git commands.

---

### Layer 3: Hooks (Automation + Context)

**Purpose**: Event-driven automation and context injection

**Design Principles**:
- Hooks run automatically at specific lifecycle events (`session_start`, `before_command`, etc.)
- Agent doesn't "choose" to use hooks; they're always active if configured
- Hooks are deterministic and lightweight (CLI commands or prompt injection)
- Hooks automate without controlling agent decision-making

**What This Provides**:
- **Context reinforcement**: Remind agent about relevant skills at right time
- **Lifecycle automation**: Session start/end hooks for setup/teardown
- **Output processing**: Transform or enrich tool results
- **Usage tracking**: Log operations for analytics

---

## Comparison with Other Approaches

| Aspect | CSH Framework | MCP | LangChain |
|--------|--------------|-----|-----------|
| **Token Efficiency** | High (progressive disclosure) | Low (all tools loaded) | Medium |
| **Tool Scalability** | Unlimited (filesystem-based) | ~100 tools | Limited |
| **Agent Freedom** | High (can bypass skills) | Schema-bound | Medium |
| **Setup Complexity** | Low (CLI-based) | Medium (server-based) | High |
| **Maintenance** | Low (declarative skills) | Medium | High |

---

## When to Use CSH Framework

### Best For
- **Local Development**: File system operations, personal automation
- **Analytics Agents**: Data transformation where token efficiency matters
- **Operations Agents**: Multi-step workflows combining disparate sources
- **Privacy-Sensitive Work**: Intermediate results stay in execution environment

### Not Best For
- **Simple one-off tools**: Overhead exceeds utility
- **Static tool sets**: Where all tools are always needed
- **Enterprise compliance**: Where MCP audit trails are required

---

## Research Validation

This architecture is based on research from:
- **Anthropic Engineering**: Code execution patterns in MCP
- **Hive Research**: Validated 87.5% token reduction (32,000 → 4,000 tokens)
- **FlowHunt Analysis**: Limitations of flat tool schemas
- **Vercel & Cloudflare**: Case studies on "Code Mode" efficiency

**Summary**: 87.5% to 98.7% token reduction is consistent across environments using progressive disclosure.

---

## Key Takeaways

1. **Progressive disclosure** is the core mechanism for token efficiency
2. **CLI primitives** leverage what agents already know
3. **Skills encode patterns**, not just commands
4. **Hooks provide context and automation** without restricting autonomy
5. **Mode-switching resistance** makes hooks necessary today

---

*Next: [Three-Layer Architecture](./02-architecture.md)*
