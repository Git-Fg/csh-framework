---
title: "Platform Comparison"
summary: "Comparison between Claude Code and OpenClaw - when to choose each platform for CSH Framework integration."
read_when:
  - You are choosing a platform
  - You want to understand trade-offs
  - You need to justify platform choice
---

# Platform Comparison: Claude Code vs. OpenClaw

Choosing between platforms for your framework integration and understanding trade-offs for each architecture.

---

## Executive Summary

| Requirement | Claude Code | OpenClaw |
|-------------|-------------|-----------|
| Personal automation | ✅ Excellent | ✅ Excellent |
| Team collaboration | ❌ Limited | ✅ Excellent |
| Persistent sessions | ❌ Ephemeral | ✅ Persistent |
| Cron/webhook automation | ❌ No | ✅ Yes |
| Quick prototyping | ✅ Excellent | ✅ Good |
| Production deployment | ⚠️ Requires effort | ✅ Production-ready |
| Multi-platform support | ❌ Claude only | ✅ Universal |
| Enterprise features | ⚠️ Basic | ✅ Available |

**Bottom Line**: Choose based on your use case — there is no "best" platform.

---

## Architecture Comparison

### Core Philosophies

| Aspect | Claude Code | OpenClaw |
|--------|-----------|-----------|
| **Approach** | Native tools + Skill guidance | CSH Primitive CLI + skill + hook plugin |
| **Philosophy** | Local Agent Performance | Universal Logic + Shared Memory |
| **Execution** | Native Bash tool (C1) | Shell/Exec Primitive (C1) |
| **Protocol** | **CSH Protocol** (Native Bash + Edit) | **CSH Protocol** (Direct Execution) |
| **Manifest** | `.claude-plugin/plugin.json` | `openclaw.plugin.json` |
| **Naming** | Namespaced: `/plugin:skill` | Global (precedence-based) |
| **Skill Discovery** | Standard Plugin Install | Gated (environment requirements) |
| **State** | In-memory/Session-bound | Persistent (Gateway-owned) |

---

## Skill System Comparison

### 1. Discovery and Loading

**Claude Code**:
- Automatically discovers skills from standard directories (`~/.claude/skills/`)
- Skills in plugins are prefixed (e.g., `/my-plugin:my-skill`)
- Implicit skill loading (agent automatically has access)
- Skills tied to Claude project configuration
- Limited to skills within that project

**OpenClaw**:
- Skills are file-based (`SKILL.md` files in `skills/` directories)
- Dynamically filters skills based on system environment (e.g., `requires.bins: ["curl"]`)
- Loads metadata (YAML frontmatter) first to optimize prompt density
- Universal discovery: Any agent using OpenClaw can discover available skills
- Skills can be shared across projects and teams
- Skills are versioned (git-based versioning)
- More flexible: Skills can be developed independently of the OpenClaw gateway

### 2. Skill Formats

**Claude Code**:
- **Location**: `~/.claude/skills/`
- **Format**: `SKILL.md` files with YAML frontmatter
- **Discovery**: Auto-scan on startup
- **Invocation**: `/skill-name` or agent invokes directly
- **Control**: Agent has high freedom and can execute any discovered skill

**OpenClaw**:
- **Location**: Plugin `skills/` directories
- **Format**: `SKILL.md` files with YAML frontmatter + markdown instructions
- **Discovery**: Via plugin manifest (`openclaw.plugin.json`)
- **Invocation**: `/skill read skill-name` or agent invokes
- **Control**: More granular — skills can be explicitly enabled/disabled

### 3. Skill Storage

| Aspect | Claude Code | OpenClaw |
|--------|-----------|-----------|
| **Location** | `~/.claude/skills/` | Plugin `skills/` directories |
| **Versioning** | Project-level | Git-based (independent) |
| **Distribution** | Via project share | Via npm, git, ClawHub |
| **Sharing** | Limited to project | Universal across deployments |
| **Precedence** | Single location | Workspace > Managed > Bundled |

---

## Hook System Comparison

### 1. Hook Architecture

**Claude Code**:
- **Nature**: Configured via `hooks/hooks.json` in plugins or project root
- **Triggers**: Supports 14 distinct events (e.g., `PreToolUse`, `SessionStart`, `TaskCompleted`)
- **Format**: Receives hook input as JSON on `stdin` (requires `jq` for extraction)
- **Scope**: Project-level hooks, no native hook system

**OpenClaw**:
- **Nature**: Configured via `HOOK.md` (instructions) and `handler.ts` (logic) in plugins
- **Automation Pipeline**: Triggered by messages, heartbeats, webhooks, or cron
- **Discovery**: Scanned with precedence: Workspace > Managed > Bundled
- **Scope**: Can be bundled within plugins to provide out-of-the-box infrastructure

### 2. Available Hook Events

| Hook Type | Claude Code | OpenClaw |
|----------|-----------|-----------|
| **Lifecycle** | `SessionStart`, `SessionEnd`, `Stop`, `TeammateIdle`, `TaskCompleted`, `PreCompact` | `session_start`, `session_end` |
| **Tool/Action** | `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` | `before_command`, `after_command` |
| **Agent/Team** | `SubagentStart`, `SubagentStop`, `TeammateIdle` | N/A (uses subagents instead) |
| **System** | `Notification`, `UserPromptSubmit` | `webhook`, `cron`, `heartbeat` |

### 3. Hook Registration Examples

**Claude Code**:
```json
{
  "hooks": {
    "events": {
      "SessionStart": {
        "command": "./start.sh"
      },
      "PreToolUse": {
        "command": "./before-tool.sh"
      },
      "PostToolUse": {
        "command": "./track.sh"
      }
    }
  }
}
```

**OpenClaw**:
```json
{
  "hooks": [
    {
      "event": "session:start",
      "handler": "./hooks/start.cjs",
      "description": "Load plugin context"
    },
    {
      "event": "before:command",
      "handler": "./hooks/track.cjs",
      "description": "Track command usage"
    }
  ]
}
```

---

## Session and State Management

### Claude Code
- **Session Type**: Interactive sessions tied to process lifecycle and `CLAUDE_SESSION_ID`
- **State**: In-memory, session-bound
- **Persistence**: Limited to session duration
- **Recovery**: No native recovery — restart resets state
- **Usage**: Primarily for interactive CLI usage

### OpenClaw
- **Session Type**: Centralized gateway manages session state
- **State**: Persistent (Gateway-owned)
- **Persistence**: Survives restarts via JSONL transcript storage
- **Queue Modes**:
  - `collect`: Coalesce messages before sending
  - `followup`: Sequential message handling
  - `steer`: Injection at tool boundaries for subagent guidance
- **DM Isolation**: Secure DM mode isolates context per sender/channel, preventing cross-talk in multi-user environments
- **Recovery**: Session recovery from JSONL transcripts

---

## Integration Patterns

### Using Both Together

It's common to run **Claude Code** with **OpenClaw** extension installed.

**Best Practices for Hybrid Environments**:

1. **Naming**:
   - Use unique prefixes for skills (`clawvault-` vs `claude-`) to prevent naming collisions
   - Avoid generic names that might overlap

2. **Hook Order**:
   - OpenClaw hooks typically run before Claude Code hooks
   - Ensure global context is loaded before project-specific hooks

3. **Memory Coordination**:
   - OpenClaw has native memory (memory-core, LanceDB) — use these
   - **Don't create memory plugins** — instead, build skills/hooks that enhance native memory
   - Use skills to teach effective memory usage patterns
   - Use hooks to extract/inject context at session boundaries

---

## Trade-offs and Use Cases

### When to Choose Claude Code

**Use When**:
- Running locally for personal use
- Need quick iteration and prototyping
- Want minimal infrastructure
- Single-user scenario
- Prefer seamless, out-of-the-box experience
- Your workflows are mostly interactive and short-lived
- You want built-in MCP support without extra configuration

**Strengths**:
- ✅ Excellent for Claude-native tasks
- ✅ Tightly integrated workflows within same project
- ✅ Seamless, out-of-the-box experience
- ✅ Quick to set up and iterate

**Limitations**:
- ❌ Limited team collaboration
- ❌ No persistent sessions
- ❌ No cron/webhook automation
- ❌ Skills limited to project scope
- ❌ Production deployment requires significant effort

### When to Choose OpenClaw

**Use When**:
- Need persistent sessions across restarts
- Require gateway management and monitoring
- Multi-user or team scenario
- Need automation triggers (cron, webhooks, heartbeats)
- Building production systems
- Creating universal, platform-independent workflows
- Multi-platform or multi-team scenarios
- Building skills that can be shared across the OpenClaw ecosystem
- Leveraging gateway features for enterprise coordination
- Independent skill development and distribution

**Strengths**:
- ✅ Universality: Skills work with any agent platform, not just Claude
- ✅ Independence: Skills can be versioned and distributed independently
- ✅ Composability: Skills can be combined across different projects and teams
- ✅ Gateway Ecosystem: Access to OpenClaw gateway features for enterprise features
- ✅ Better Separation of Concerns: Hooks and skills are decoupled from agent logic
- ✅ Extensibility: Anyone can create hooks and skills without modifying OpenClaw core

**Limitations**:
- ❌ Integration Complexity: Skills need to understand OpenClaw's gateway APIs and hook system
- ❌ Discoverability: Requires CLI tools and knowledge of OpenClaw conventions
- ❌ Testing Challenges: Skills must work correctly across different agent platforms
- ❌ Coordination Overhead: Skills may need to coordinate through the gateway for multi-agent scenarios

---

## Honest Trade-offs Assessment

### CLI + Skills + Hooks Framework

#### Advantages ✅

1. **Maximum Token Efficiency**
   - Progressive disclosure eliminates up to 98% of token waste
   - Context filtering in code reduces intermediate result bloat
   - Empirical: 32,000 → 4,000 tokens in production scenarios (87.5% reduction)

2. **Unlimited Tool Scalability**
   - Filesystem access provides infinite tools (git, kubectl, aws, docker, npm, curl, jq, etc.)
   - CLIs already exist for virtually every service
   - Zero implementation overhead

3. **Agent Autonomy Preserved**
   - Agent can always bypass skills and use CLI primitives directly
   - Skills teach patterns, they don't restrict agent freedom
   - Hooks are automation/enforcement, not decision-making

4. **Privacy by Default**
   - Intermediate results stay in execution environment
   - Data harness layer can redact sensitive data before reaching model
   - No third-party model provider exposure for PII

5. **Composability**
   - Unix philosophy: Small, composable tools chained with pipes
   - Text-based interfaces enable universal composability

6. **Skill Evolution**
   - Agents can create, persist, and evolve their own skills
   - Compounding benefits: 67% improvement per skill (12K → 4K tokens)
   - State persistence through filesystem

#### Disadvantages ❌

1. **Infrastructure Overhead**
   - Requires secure sandbox (containers, Firecracker, V8 isolates)
   - Persistent storage for skill development
   - Monitoring and observability systems
   - Authentication and credential management
   - **Not plug-and-play**: Production deployment requires engineering work

2. **Prompting Complexity**
   - Requires careful instruction design for skills
   - Must iterate on prompts based on agent behavior
   - Poor prompting leads to inefficient token usage
   - Complexity scales with number of skills

3. **Reliability Trade-offs**
   - Code execution increases error probability per tool invocation
   - Requires robust error handling and retry mechanisms
   - Modern LLMs compensate, but not perfectly
   - More failure modes than direct tool calls

4. **Learning Curve**
   - Must understand three-layer architecture
   - Skills have learning curve for patterns
   - Hooks system has configuration complexity
   - CLI commands have learning curve (but universal)

5. **Enterprise Readiness**
   - Missing enterprise features (no approvals, no RBAC, no content safety)
   - Requires additional development for compliance features
   - Not suitable for regulated industries without extensions

### Use Cases: When to Use CLI + Skills + Hooks

**Excellent For**:

1. **Local Development**
   - Building tools for local use
   - File system operations
   - Personal scripts and automation
   - **Why**: Zero infrastructure overhead, maximum control, privacy

2. **Analytics Agents**
   - Data transformation and filtering
   - Research agents combining multiple sources
   - Operations agents with multi-step workflows
   - **Why**: Token efficiency critical, complex workflows need progressive disclosure

3. **Research Agents**
   - Code review and security scanning
   - Documentation generation and maintenance
   - Multi-step knowledge synthesis
   - **Why**: Skill evolution enables domain expertise compounding over time

4. **Operations Agents**
   - Deployment automation (CI/CD, infrastructure as code)
   - Monitoring and observability
   - Log aggregation and alerting
   - Multi-step workflows requiring coordination
   - **Why**: Hooks automate repetitive tasks, skills encode operational patterns

5. **General-Purpose Agents**
   - Personal assistant (calendar, email, documents)
   - Universal tool access across domains
   - **Why**: CLI provides universal interface, composable toolchain

**Not Ideal For**:

1. **Simple, Single-Step API Calls**
   - Customer support ticket creation
   - Retrieving single document from database
   - One-off data transformation without complex processing
   - **Why**: MCP provides standardized tool definition and discovery, lower complexity

2. **Enterprise Multi-Agent Systems**
   - Multiple agents collaborating across teams
   - Shared tool definitions via MCP servers
   - Centrally managed access controls and governance
   - **Why**: MCP provides standardization, governance features, enterprise-grade authentication

---

## Decision Framework

### Migration Paths

**Why Migrate to OpenClaw?**
- You need **multi-channel** access (Desktop + Mobile/Telegram)
- You require **granular control** over skill availability (gating)
- You want **built-in persistent memory** (memory-core, LanceDB) without building it yourself

**Why Stick with Claude Code?**
- You want **seamless, out-of-the-box experience** of desktop app
- You prefer **built-in MCP support** without extra configuration
- Your workflows are mostly **interactive and short-lived**

### Decision Checklist

Use this checklist to decide:

```
Needs Multi-Platform Support?
├─ Yes → Choose OpenClaw
└─ No → Continue

Needs Persistent Sessions?
├─ Yes → Choose OpenClaw
└─ No → Continue

Needs Team Collaboration?
├─ Yes → Choose OpenClaw
└─ No → Continue

Needs Cron/Webhook Automation?
├─ Yes → Choose OpenClaw
└─ No → Continue

Prefers Minimal Setup?
├─ Yes → Choose Claude Code
└─ No → Choose OpenClaw

Building Production System?
├─ Yes → Choose OpenClaw
└─ No → Either works
```

---

## Cross-Platform Development

See [Cross-Platform Skills](../patterns/04-cross-platform.md) for techniques to support both platforms from a single codebase.

---

## Summary

**For Single Developers**: Claude Code is excellent for local, interactive work. For production systems, OpenClaw provides the necessary infrastructure (persistence, automation, team collaboration).

**For Enterprise Teams**: OpenClaw is the clear choice with persistent sessions, automation triggers, and universal skill sharing.

**For Hybrid Environments**: Use both together, following best practices for naming, hook order, and memory coordination.

**The Honest Truth**: There is no magic bullet. Each platform has strengths and limitations. Choose based on your specific use case and requirements.

---

*Next: [Cross-Platform Skills](../patterns/04-cross-platform.md)*
