---
title: "Complete Plugin Hooks Reference - OpenClaw API"
summary: "Comprehensive reference for all 22 OpenClaw plugin hooks with when/how to use each one. Verified against source code v2026.2.25."
read_when:
  - You need to implement OpenClaw plugin hooks
  - You are choosing which plugin hooks to use
  - You want to understand hook parameters and capabilities
---

# Complete Plugin Hooks Reference - OpenClaw API

> ⚠️ **IMPORTANT**: This document covers **OpenClaw plugin hooks** (registered via `api.on()` in plugins). For **Claude Code hooks**, see `hooks.md` in this directory.

> **Verification**: All 22 hooks listed below are verified against OpenClaw v2026.2.25 source code at `/usr/lib/node_modules/openclaw/dist/plugin-sdk/plugins/types.d.ts`.

---

## Table of Contents

- [Quick Reference: All 22 Hooks](#quick-reference-all-22-hooks)
- [Hook Categories](#hook-categories)
  - [Agent Lifecycle Hooks](#agent-lifecycle-hooks)
  - [Compaction & Reset Hooks](#compaction--reset-hooks)
  - [Tool Hooks](#tool-hooks)
  - [Message Hooks](#message-hooks)
  - [Session Hooks](#session-hooks)
  - [Subagent Hooks](#subagent-hooks)
  - [Gateway Hooks](#gateway-hooks)
- [When to Use Which Hook](#when-to-use-which-hook)
- [Registration Examples](#registration-examples)
- [Best Practices](#best-practices)
- [Priority System](#priority-system)

---

## Quick Reference: All 22 Hooks

| Category | Hook | Can Modify | Purpose |
|-----------|--------|------------|----------|
| **Agent** | `before_model_resolve` | ✅ Override model/provider | Model selection override |
| **Agent** | `before_prompt_build` | ✅ Inject context | Modify system prompt, prepend context |
| **Agent** | `before_agent_start` | ✅ Override model/provider/context | Legacy hook (before agent starts) |
| **Agent** | `llm_input` | ❌ Read-only | Observe LLM input before inference |
| **Agent** | `llm_output` | ❌ Read-only | Observe LLM output + usage stats |
| **Agent** | `agent_end` | ❌ Read-only | Observe final agent state |
| **Compaction** | `before_compaction` | ❌ Read-only | Observe before compaction |
| **Compaction** | `after_compaction` | ❌ Read-only | Observe after compaction |
| **Reset** | `before_reset` | ❌ Read-only | Observe before session reset |
| **Tool** | `before_tool_call` | ✅ Modify params / block execution | Control tool execution |
| **Tool** | `after_tool_call` | ❌ Read-only | Observe tool results |
| **Tool** | `tool_result_persist` | ✅ Modify message | Transform tool results before persistence |
| **Message** | `message_received` | ❌ Read-only | Observe inbound messages |
| **Message** | `message_sending` | ✅ Modify / cancel | Control outbound messages |
| **Message** | `message_sent` | ❌ Read-only | Observe sent messages |
| **Message** | `before_message_write` | ✅ Block / modify | Filter messages before transcript write |
| **Session** | `session_start` | ❌ Read-only | Observe session start |
| **Session** | `session_end` | ❌ Read-only | Observe session end |
| **Subagent** | `subagent_spawning` | ✅ Influence binding | Control subagent spawn |
| **Subagent** | `subagent_delivery_target` | ✅ Set origin | Control delivery to channel |
| **Subagent** | `subagent_spawned` | ❌ Read-only | Observe after spawn |
| **Subagent** | `subagent_ended` | ❌ Read-only | Observe subagent termination |
| **Gateway** | `gateway_start` | ❌ Read-only | Observe gateway start |
| **Gateway** | `gateway_stop` | ❌ Read-only | Observe gateway stop |

**Legend**:
- ✅ **Can Modify**: Can return values that change behavior
- ❌ **Read-Only**: Can observe but cannot modify

---

## Hook Categories

### Agent Lifecycle Hooks (6)

Hooks that fire during agent execution, allowing you to observe or modify the agent's inputs, prompts, and outputs.

#### before_model_resolve

**Trigger**: Before the model is selected for this agent run.

**Can Modify**: ✅ **YES** - Can override model or provider.

**Return Type**:
```typescript
{
  modelOverride?: string,      // e.g., "llama3.3:8b"
  providerOverride?: string     // e.g., "ollama"
}
```

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string,
  workspaceDir?: string,
  messageProvider?: string
}
```

**When to Use**:
- ✅ **Force specific model**: Override default model for certain tasks
- ✅ **Provider routing**: Route to different provider based on context
- ✅ **Cost optimization**: Select cheaper model for simple queries

**Example**:
```typescript
api.on('before_model_resolve', async (event) => {
  if (event.prompt.toLowerCase().includes('quick')) {
    return { modelOverride: 'llama3.3:8b' };  // Cheaper model
  }
});
```

#### before_prompt_build

**Trigger**: After session messages are loaded, but before the system prompt is assembled.

**Can Modify**: ✅ **YES** - Can inject system prompt or prepend context.

**Return Type**:
```typescript
{
  systemPrompt?: string,      // Replace entire system prompt
  prependContext?: string      // Add before system prompt
}
```

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string,
  workspaceDir?: string,
  messageProvider?: string
}
```

**Event Data**:
```typescript
{
  prompt: string,              // User prompt for this run
  messages: unknown[]         // Session messages (prepared for this run)
}
```

**When to Use**:
- ✅ **Context injection**: Add project-specific context to every run
- ✅ **System prompt modification**: Add rules or guidelines
- ✅ **Conditional instructions**: Add instructions based on prompt content

**Example**:
```typescript
api.on('before_prompt_build', async (event) => {
  return {
    prependContext: `
## Project Context
- Current branch: main
- Sprint: Authentication refactor
- Use TypeScript, not JavaScript
`
  };
});
```

#### before_agent_start

**Trigger**: Before the agent starts processing the prompt.

**Can Modify**: ✅ **YES** - Legacy hook that can modify multiple aspects.

**Return Type**:
```typescript
{
  modelOverride?: string,
  providerOverride?: string,
  systemPrompt?: string,
  prependContext?: string
}
```

**When to Use**:
- ⚠️ **Legacy compatibility**: Prefer `before_model_resolve` and `before_prompt_build` for new code.
- ✅ **Combined overrides**: Need to modify model, provider, AND prompt together.

**Note**: This is a legacy compatibility hook. Use `before_model_resolve` and `before_prompt_build` for clarity.

#### llm_input

**Trigger**: Just before the LLM is called with the input.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string
}
```

**Event Data**:
```typescript
{
  runId: string,
  sessionId: string,
  provider: string,
  model: string,
  systemPrompt?: string,
  prompt: string,
  historyMessages: unknown[],
  imagesCount: number
}
```

**When to Use**:
- ✅ **Usage tracking**: Log all LLM invocations for analytics
- ✅ **Cost monitoring**: Track prompt size and image count
- ✅ **Debugging**: See exactly what's sent to the LLM

**Example**:
```typescript
api.on('llm_input', async (event) => {
  console.log(`[LLM] Provider: ${event.provider}, Model: ${event.model}`);
  console.log(`[LLM] Prompt length: ${event.prompt.length}, Images: ${event.imagesCount}`);
});
```

#### llm_output

**Trigger**: Just after the LLM returns a response.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string
}
```

**Event Data**:
```typescript
{
  runId: string,
  sessionId: string,
  provider: string,
  model: string,
  assistantTexts: string[],
  lastAssistant?: unknown,
  usage?: {
    input?: number,
    output?: number,
    cacheRead?: number,
    cacheWrite?: number,
    total?: number
  }
}
```

**When to Use**:
- ✅ **Cost tracking**: Calculate token costs from usage stats
- ✅ **Quality monitoring**: Analyze output for patterns
- ✅ **Performance metrics**: Track LLM response times

**Example**:
```typescript
api.on('llm_output', async (event) => {
  const cost = calculateCost(event.usage);
  console.log(`[COST] Session ${event.sessionId}: $${cost.toFixed(2)}`);
});
```

#### agent_end

**Trigger**: After the agent finishes processing (after all tool calls and output).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string,
  workspaceDir?: string,
  messageProvider?: string
}
```

**Event Data**:
```typescript
{
  messages: unknown[],
  success: boolean,
  error?: string,
  durationMs?: number
}
```

**When to Use**:
- ✅ **Final logging**: Log session completion status
- ✅ **Error handling**: Catch and log agent failures
- ✅ **Duration tracking**: Measure agent response times

**Example**:
```typescript
api.on('agent_end', async (event) => {
  console.log(`[AGENT] Session ${event.sessionId} finished in ${event.durationMs}ms`);
  if (!event.success) {
    console.error(`[AGENT] Error: ${event.error}`);
  }
});
```

---

### Compaction & Reset Hooks (3)

Hooks that fire when the session context is compacted (to save tokens) or reset.

#### before_compaction

**Trigger**: Before compaction starts (when context window is full).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string,
  workspaceDir?: string,
  messageProvider?: string
}
```

**Event Data**:
```typescript
{
  messageCount: number,          // Total messages before any truncation
  compactingCount?: number,     // Messages being fed to compaction LLM
  tokenCount?: number,          // Current token count
  messages?: unknown[],         // Full message list
  sessionFile?: string           // Path to session JSONL (can read async)
}
```

**When to Use**:
- ✅ **Warning**: Notify user before context is compressed
- ✅ **Save critical context**: Write important information to disk before compaction
- ✅ **Async processing**: Process messages in parallel with compaction LLM

**Example**:
```typescript
api.on('before_compaction', async (event) => {
  if (event.sessionFile) {
    // Read and archive important messages asynchronously
    const messages = readFileSync(event.sessionFile, 'utf-8');
    const critical = extractCriticalInfo(messages);
    writeFileSync('/tmp/pre-compact-backup.txt', critical);
  }
});
```

#### after_compaction

**Trigger**: After compaction completes.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**: Same as `before_compaction`.

**Event Data**:
```typescript
{
  messageCount: number,
  compactedCount: number,
  tokenCount?: number,
  sessionFile?: string           // Path to session JSONL (can read async)
}
```

**When to Use**:
- ✅ **Recovery**: Re-inject critical context that was lost
- ✅ **Logging**: Log compaction statistics
- ✅ **Cleanup**: Remove temporary files created before compaction

**Example**:
```typescript
api.on('after_compaction', async (event) => {
  return {
    prependContext: `
[RESTORED CONTEXT]
Critical information from before compaction has been restored.
Compacted ${event.compactedCount} messages.
`
  };
});
```

#### before_reset

**Trigger**: Before a session is reset (via `/reset` or other means).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  sessionId?: string,
  workspaceDir?: string,
  messageProvider?: string
}
```

**Event Data**:
```typescript
{
  sessionFile?: string,           // Path to session file
  messages?: unknown[],          // Messages being reset
  reason?: string               // Reason for reset
}
```

**When to Use**:
- ✅ **Cleanup**: Remove temporary files before reset
- ✅ **Logging**: Log reset events for audit trail
- ✅ **State saving**: Save important state before it's lost

**Example**:
```typescript
api.on('before_reset', async (event) => {
  if (event.reason) {
    console.log(`[RESET] Reason: ${event.reason}`);
  }
  // Clean up temp files
  cleanupTempFiles();
});
```

---

### Tool Hooks (3)

Hooks that control or observe tool execution.

#### before_tool_call

**Trigger**: Before a tool is executed.

**Can Modify**: ✅ **YES** - Can modify tool parameters or block execution.

**Return Type**:
```typescript
{
  params?: Record<string, unknown>,      // Modified tool parameters
  block?: boolean,                       // Block execution
  blockReason?: string                    // Reason for blocking
}
```

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  toolName: string
}
```

**Event Data**:
```typescript
{
  toolName: string,
  params: Record<string, unknown>
}
```

**When to Use**:
- ✅ **Security**: Block dangerous tool calls (`rm -rf`, `drop table`)
- ✅ **Validation**: Validate tool inputs before execution
- ✅ **Transformation**: Add defaults or transform parameters
- ✅ **Routing**: Redirect to alternative tools based on parameters

**Example**: Block dangerous writes
```typescript
api.on('before_tool_call', async (event, ctx) => {
  if (event.toolName === 'write' || event.toolName === 'edit') {
    const vaultPaths = ['/root/mastervault', '/root/clawvault'];
    const filePath = event.params?.file_path || '';
    if (vaultPaths.some(p => filePath.includes(p))) {
      return {
        block: true,
        blockReason: 'Use ClawVault CLI, not direct file edits'
      };
    }
  }
});
```

**Example**: Modify parameters
```typescript
api.on('before_tool_call', async (event) => {
  if (event.toolName === 'Bash' && event.params?.command?.includes('npm')) {
    return {
      params: {
        ...event.params,
        command: event.params.command + ' --legacy-peer-deps'
      }
    };
  }
});
```

#### after_tool_call

**Trigger**: After a tool execution completes (success or failure).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  toolName: string
}
```

**Event Data**:
```typescript
{
  toolName: string,
  params: Record<string, unknown>,
  result?: unknown,
  error?: string,
  durationMs?: number
}
```

**When to Use**:
- ✅ **Logging**: Log all tool executions
- ✅ **Error handling**: Catch and log tool failures
- ✅ **Trigger follow-up**: Run post-processing after successful tools

**Example**:
```typescript
api.on('after_tool_call', async (event) => {
  if (event.error) {
    console.error(`[TOOL] ${event.toolName} failed: ${event.error}`);
  } else {
    console.log(`[TOOL] ${event.toolName} succeeded in ${event.durationMs}ms`);
  }
});
```

#### tool_result_persist

**Trigger**: Right before a tool result is written to the session transcript.

**Can Modify**: ✅ **YES** - Can modify the result message.

**Return Type**:
```typescript
{
  message?: AgentMessage      // Modified tool result message
}
```

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string,
  toolName?: string,
  toolCallId?: string
}
```

**Event Data**:
```typescript
{
  toolName?: string,
  toolCallId?: string,
  message: AgentMessage,           // The tool result about to be persisted
  isSynthetic?: boolean            // True if synthesized by guard/repair step
}
```

**When to Use**:
- ✅ **Data redaction**: Remove sensitive information from tool results
- ✅ **Size reduction**: Remove non-essential fields to save tokens
- ✅ **Sanitization**: Clean up tool output before storage

**Example**:
```typescript
api.on('tool_result_persist', async (event) => {
  const message = event.message;
  if (message?.content) {
    // Remove API keys, passwords from content
    const sanitized = redactSecrets(message.content);
    return { message: { ...message, content: sanitized } };
  }
});
```

---

### Message Hooks (4)

Hooks that control or observe message flow through the gateway.

#### message_received

**Trigger**: When an inbound message is received from any channel.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  channelId: string,           // e.g., "telegram", "whatsapp"
  accountId?: string,
  conversationId?: string
}
```

**Event Data**:
```typescript
{
  from: string,                  // Sender identifier (phone number, user ID)
  content: string,               // Message content
  timestamp?: number,            // Unix timestamp
  metadata?: Record<string, unknown>  // Provider-specific data
}
```

**When to Use**:
- ✅ **Intent detection**: Analyze user messages to trigger workflows
- ✅ **Routing**: Route messages to different handlers
- ✅ **Logging**: Audit log of all inbound messages
- ✅ **Detection**: Detect "remember this" or similar patterns

**Example**:
```typescript
api.on('message_received', async (event) => {
  const msg = event.content?.toLowerCase() || '';
  if (msg.includes('remember') || msg.includes('save')) {
    return {
      prependContext: '[MANDATORY] User wants to remember something. Use memory-capture skill.'
    };
  }
});
```

#### message_sending

**Trigger**: Before an outbound message is sent to the channel.

**Can Modify**: ✅ **YES** - Can modify content or cancel sending.

**Return Type**:
```typescript
{
  content?: string,      // Modified message content
  cancel?: boolean        // Cancel the send
}
```

**Context Available**:
```typescript
{
  channelId: string,
  accountId?: string,
  conversationId?: string
}
```

**Event Data**:
```typescript
{
  to: string,                    // Recipient identifier
  content: string,               // Message content
  metadata?: Record<string, unknown>
}
```

**When to Use**:
- ✅ **Content modification**: Add disclaimers or formatting
- ✅ **Duplicate suppression**: Prevent sending duplicate messages
- ✅ **Filtering**: Cancel messages based on content
- ✅ **Routing**: Redirect to different channel

**Example**:
```typescript
api.on('message_sending', async (event) => {
  // Cancel duplicate messages
  if (isDuplicate(event.content)) {
    return { cancel: true };
  }

  // Add footer
  return {
    content: event.content + '\n\n---\nSent via OpenClaw'
  };
});
```

#### message_sent

**Trigger**: After an outbound message is successfully sent (or failed).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  channelId: string,
  accountId?: string,
  conversationId?: string
}
```

**Event Data**:
```typescript
{
  to: string,                    // Recipient identifier
  content: string,               // Message content that was sent
  success: boolean,              // Whether send succeeded
  error?: string                 // Error message if failed
}
```

**When to Use**:
- ✅ **Audit logging**: Log all outbound messages
- ✅ **Error handling**: Catch and handle send failures
- ✅ **Analytics**: Track message delivery rates

**Example**:
```typescript
api.on('message_sent', async (event) => {
  if (!event.success) {
    console.error(`[MESSAGE] Failed to send to ${event.to}: ${event.error}`);
    // Trigger retry or alert
  }
});
```

#### before_message_write

**Trigger**: Before a message is written to the session transcript.

**Can Modify**: ✅ **YES** - Can block or modify the message.

**Return Type**:
```typescript
{
  block?: boolean,        // Block writing to transcript
  message?: AgentMessage   // Modified message
}
```

**Context Available**:
```typescript
{
  agentId?: string,
  sessionKey?: string
}
```

**Event Data**:
```typescript
{
  message: AgentMessage,     // The message about to be written
  sessionKey?: string,
  agentId?: string
}
```

**When to Use**:
- ✅ **Filtering**: Block certain messages from being persisted
- ✅ **Redaction**: Remove sensitive info before storage
- ✅ **Sanitization**: Clean up messages before transcript

**Example**:
```typescript
api.on('before_message_write', async (event) => {
  const message = event.message;
  if (message?.content?.includes('API_KEY')) {
    return {
      block: true  // Don't write secrets to transcript
    };
  }
});
```

---

### Session Hooks (2)

Hooks that observe session lifecycle events.

#### session_start

**Trigger**: When a new session begins.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionId: string
}
```

**Event Data**:
```typescript
{
  sessionId: string,
  resumedFrom?: string      // Previous session ID if resumed
}
```

**When to Use**:
- ✅ **Initialization**: Set up session-specific state
- ✅ **Context loading**: Load initial context for new sessions
- ✅ **Resume handling**: Handle session resumption differently

**Example**:
```typescript
api.on('session_start', async (event) => {
  console.log(`[SESSION] Started: ${event.sessionId}`);
  if (event.resumedFrom) {
    console.log(`[SESSION] Resumed from: ${event.resumedFrom}`);
  }
});
```

#### session_end

**Trigger**: When a session ends.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  agentId?: string,
  sessionId: string
}
```

**Event Data**:
```typescript
{
  sessionId: string,
  messageCount: number,
  durationMs?: number
}
```

**When to Use**:
- ✅ **Cleanup**: Clean up session resources
- ✅ **Logging**: Log session statistics
- ✅ **Checkpointing**: Save final state

**Example**:
```typescript
api.on('session_end', async (event) => {
  console.log(`[SESSION] Ended: ${event.sessionId}`);
  console.log(`[SESSION] Duration: ${event.durationMs}ms, Messages: ${event.messageCount}`);
});
```

---

### Subagent Hooks (4)

Hooks that control or observe subagent spawning and termination. **These are NEW in OpenClaw 2026.2+**.

#### subagent_spawning

**Trigger**: Before a subagent is spawned.

**Can Modify**: ✅ **YES** - Can influence thread binding and status.

**Return Type**:
```typescript
{
  status: "ok",
  threadBindingReady?: boolean
}
| {
  status: "error",
  error: string
}
```

**Context Available**:
```typescript
{
  runId?: string,
  childSessionKey?: string,
  requesterSessionKey?: string
}
```

**Event Data**:
```typescript
{
  childSessionKey: string,
  agentId: string,
  label?: string,
  mode: "run" | "session",
  requester?: {
    channel?: string,
    accountId?: string,
    to?: string,
    threadId?: string | number
  },
  threadRequested: boolean
}
```

**When to Use**:
- ✅ **Thread binding**: Control whether subagent binds to messaging thread
- ✅ **Pre-spawn context**: Set up context before subagent starts
- ✅ **Quality gate**: Prevent spawn if conditions not met

**Example**:
```typescript
api.on('subagent_spawning', async (event) => {
  if (event.threadRequested) {
    return {
      status: 'ok',
      threadBindingReady: true  // Allow thread binding
    };
  }
});
```

#### subagent_delivery_target

**Trigger**: Before subagent output is delivered to the messaging channel.

**Can Modify**: ✅ **YES** - Can set delivery origin.

**Return Type**:
```typescript
{
  origin?: {
    channel?: string,
    accountId?: string,
    to?: string,
    threadId?: string | number
  }
}
```

**Context Available**:
```typescript
{
  runId?: string,
  childSessionKey?: string,
  requesterSessionKey?: string
}
```

**Event Data**:
```typescript
{
  childSessionKey: string,
  requesterSessionKey?: string,
  requesterOrigin?: {
    channel?: string,
    accountId?: string,
    to?: string,
    threadId?: string | number
  },
  childRunId?: string,
  spawnMode?: "run" | "session",
  expectsCompletionMessage: boolean
}
```

**When to Use**:
- ✅ **Routing control**: Override delivery destination
- ✅ **Channel selection**: Choose which account/channel to use
- ✅ **Thread management**: Control thread binding for delivery

**Example**:
```typescript
api.on('subagent_delivery_target', async (event) => {
  if (event.childSessionKey.includes('research')) {
    // Route research results to a different channel
    return {
      origin: {
        channel: 'discord',
        accountId: 'research-bot'
      }
    };
  }
});
```

#### subagent_spawned

**Trigger**: After a subagent has been spawned.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  runId?: string,
  childSessionKey?: string,
  requesterSessionKey?: string
}
```

**Event Data**:
```typescript
{
  runId: string,
  childSessionKey: string,
  agentId: string,
  label?: string,
  mode: "run" | "session",
  requester?: {
    channel?: string,
    accountId?: string,
    to?: string,
    threadId?: string | number
  },
  threadRequested: boolean
}
```

**When to Use**:
- ✅ **Tracking**: Track all spawned subagents
- ✅ **Logging**: Log subagent spawn events
- ✅ **Monitoring**: Observe subagent startup times

**Example**:
```typescript
api.on('subagent_spawned', async (event) => {
  console.log(`[SUBAGENT] Spawned: ${event.childSessionKey} (${event.agentId})`);
});
```

#### subagent_ended

**Trigger**: After a subagent terminates (any outcome).

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  runId?: string,
  targetSessionKey?: string
}
```

**Event Data**:
```typescript
{
  targetSessionKey: string,
  targetKind: "subagent" | "acp",
  reason: string,
  sendFarewell?: boolean,
  accountId?: string,
  runId?: string,
  endedAt?: number,
  outcome?: "ok" | "error" | "timeout" | "killed" | "reset" | "deleted",
  error?: string
}
```

**When to Use**:
- ✅ **Result handling**: Process subagent output
- ✅ **Error handling**: Catch and handle subagent failures
- ✅ **Cleanup**: Clean up subagent resources

**Example**:
```typescript
api.on('subagent_ended', async (event) => {
  console.log(`[SUBAGENT] Ended: ${event.targetSessionKey}`);
  console.log(`[SUBAGENT] Outcome: ${event.outcome}, Reason: ${event.reason}`);
  if (event.outcome === 'error') {
    console.error(`[SUBAGENT] Error: ${event.error}`);
  }
});
```

---

### Gateway Hooks (2)

Hooks that observe gateway lifecycle events.

#### gateway_start

**Trigger**: When the gateway starts up.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  port?: number
}
```

**Event Data**:
```typescript
{
  port: number
}
```

**When to Use**:
- ✅ **Initialization**: Set up gateway-wide state
- ✅ **Plugin startup**: Initialize plugin services
- ✅ **Health checks**: Run diagnostic checks

**Example**:
```typescript
api.on('gateway_start', async (event) => {
  console.log(`[GATEWAY] Started on port ${event.port}`);
  // Initialize plugin services
  initializeServices();
});
```

#### gateway_stop

**Trigger**: When the gateway shuts down.

**Can Modify**: ❌ **NO** - Read-only observation hook.

**Context Available**:
```typescript
{
  port?: number
}
```

**Event Data**:
```typescript
{
  reason?: string
}
```

**When to Use**:
- ✅ **Cleanup**: Clean up resources before shutdown
- ✅ **Save state**: Persist important data
- ✅ **Graceful shutdown**: Close connections cleanly

**Example**:
```typescript
api.on('gateway_stop', async (event) => {
  console.log(`[GATEWAY] Stopping: ${event.reason || 'normal'}`);
  // Cleanup resources
  cleanupResources();
  saveState();
});
```

---

## When to Use Which Hook

### Decision Tree

```
Need to...
├─ Select different model?
│  └─> before_model_resolve
│
├─ Inject context into prompt?
│  └─> before_prompt_build
│
├─ Track LLM usage/costs?
│  └─> llm_output (observe after)
│
├─ Control tool execution?
│  ├─> before_tool_call (modify/block)
│  ├─> after_tool_call (observe)
│  └─> tool_result_persist (transform before save)
│
├─ Filter messages?
│  ├─> message_received (detect intent)
│  ├─> message_sending (modify/cancel)
│  ├─> message_sent (observe)
│  └─> before_message_write (block/modify)
│
├─ Manage subagents?
│  ├─> subagent_spawning (control spawn)
│  ├─> subagent_delivery_target (control delivery)
│  ├─> subagent_spawned (observe)
│  └─> subagent_ended (observe result)
│
├─ Session lifecycle?
│  ├─> session_start (initialize)
│  └─> session_end (cleanup)
│
├─ Compaction?
│  ├─> before_compaction (warn/save)
│  ├─> after_compaction (restore)
│  └─> before_reset (cleanup)
│
└─ Gateway lifecycle?
   ├─> gateway_start (initialize)
   └─> gateway_stop (cleanup)
```

---

## Registration Examples

### Basic Hook Registration

```typescript
// src/plugin.ts
import type { Plugin } from 'openclaw';

export default function register(api: Plugin) {
  console.log('[MyPlugin] Loading...');

  // Register a hook
  api.on('before_tool_call', async (event, ctx) => {
    console.log(`Tool called: ${ctx.toolName}`);
    // Hook logic here
  });
}
```

### Hook with Priority

```typescript
// Lower priority = higher precedence
api.on({ name: 'before_tool_call', priority: 100 }, async (event, ctx) => {
  // This runs after priority 0-99 hooks
  // But before priority 101+ hooks
});
```

### Multiple Events

```typescript
// Register same handler for multiple events
api.on(['message_received', 'message_sent'], async (event, ctx) => {
  console.log(`Message event: ${event.type}`);
});
```

---

## Best Practices

### 1. Fast Hooks

Hooks run in the hot path. Keep them under 100ms where possible.

**Bad**:
```typescript
api.on('before_tool_call', async (event) => {
  await longRunningTask();  // Blocks tool call
});
```

**Good**:
```typescript
api.on('before_tool_call', async (event) => {
  void longRunningTask();  // Fire and forget
});
```

### 2. Idempotent Hooks

Hooks should handle being called multiple times safely.

**Bad**:
```typescript
api.on('before_compaction', async (event) => {
  writeFileSync('/tmp/state.txt', JSON.stringify(event));
});
```

**Good**:
```typescript
api.on('before_compaction', async (event) => {
  if (!existsSync('/tmp/state.txt')) {
    writeFileSync('/tmp/state.txt', JSON.stringify(event));
  }
});
```

### 3. Error Handling

Wrap risky operations in try/catch. Don't throw.

**Bad**:
```typescript
api.on('before_tool_call', async (event) => {
  const result = JSON.parse(userInput);  // Can throw
  return { params: result };
});
```

**Good**:
```typescript
api.on('before_tool_call', async (event) => {
  try {
    const result = JSON.parse(userInput);
    return { params: result };
  } catch (err) {
    api.logger.error(`[Hook] Parse error: ${err}`);
    return {};  // Don't block on parse error
  }
});
```

### 4. Use Priority Wisely

Only use priority when order matters. Lower number = higher priority.

**Example**: Security validation should run first
```typescript
// Security hook - runs first (priority 0)
api.on({ name: 'before_tool_call', priority: 0 }, validateSafety);

// Feature hook - runs after security (priority 100)
api.on({ name: 'before_tool_call', priority: 100 }, addFeatures);
```

---

## Priority System

Hooks of the same type are executed in priority order:

- **Priority 0-9**: Security, validation (runs first)
- **Priority 10-99**: Core features, transformations
- **Priority 100+**: Observations, logging (runs last)

**Priority 0** = highest priority, **999** = lowest priority.

If not specified, hooks execute in registration order.

---

## Summary

| Category | Hooks | Purpose |
|-----------|--------|---------|
| **Agent Lifecycle** | 6 | Model selection, prompt injection, observation |
| **Compaction/Reset** | 3 | Context management, cleanup |
| **Tools** | 3 | Tool control, transformation |
| **Messages** | 4 | Message flow control, filtering |
| **Sessions** | 2 | Session lifecycle observation |
| **Subagents** | 4 | Subagent orchestration |
| **Gateway** | 2 | Gateway lifecycle |
| **Total** | **22** | Complete coverage |

---

## Related Docs

- [OpenClaw Hook Events](./hook-events.md) - Claude Code hooks (different system)
- [Plugin Development Guide](../../guides/02-plugin-development.md) - How to build plugins
- [Internal Hooks Reference](../automation/hooks.md) - Gateway hooks (HOOK.md format)
