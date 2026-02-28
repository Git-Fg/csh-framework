---
title: "Skills Hub-and-Spoke Pattern"
summary: "Formal documentation for the Hub-and-Spoke architecture for managing complex sets of agent instructions."
read_when:
  - You have more than 3-4 specialized workflows
  - You want to reduce baseline token usage
  - You are refactoring a large monolithic skill
---

# Skills Hub-and-Spoke Pattern

The Hub-and-Spoke pattern is the primary CSH mechanism for managing high-complexity plugins without overwhelming the agent's context window.

## Core Component: The Hub Skill
The "Hub" is the primary skill loaded by the agent (usually the root `SKILL.md`). It behaves like a **Router**.

### Requirements
1. **Minimal Context**: Should not contain specific code or detailed steps.
2. **Clear Routing**: Uses bullet points matching specific intents to specialized skills.
3. **Strong Language**: Tells the agent it MUST invoke a spoke to proceed.

```markdown
# MyPlugin: Main Support

I can assist with advanced cloud operations. Do NOT attempt these manually.
You MUST choose a specialized workflow:

- **Deployment**: use skill 'cloud-deploy-flow'
- **Cost Analysis**: use skill 'cloud-cost-audit'
- **Permissions**: use skill 'cloud-iam-manager'
```

## Core Component: The Spoke Skill
A "Spoke" is a targeted instruction file usually kept in a `references/` subdirectory.

### Requirements
1. **High Detail**: Contains the actual CLI commands, flags, and edge cases.
2. **Self-Contained**: Focuses on exactly one outcome.
3. **Execution-Ready**: Written so the agent can follow it step-by-step.

## Implementation Details

### File Structure
```
plugin/
└── skills/
    ├── SKILL.md                 # The HUB
    └── references/
        ├── deploy.md            # Spoke A
        ├── audit.md             # Spoke B
        └── iam.md               # Spoke C
```

### Loading Strategy
The Hub is loaded at session start or on detection. The Spokes are *only* loaded when the agent explicitly types `use skill 'spoke-name'`.

## Benefits
- **High Recall**: Agents are 95%+ likely to follow instructions when they are short and focused (spokes).
- **Infinite Scalability**: You can add dozens of spokes without degrading performance of the main session.
- **Cheap**: Baseline context stays low.
