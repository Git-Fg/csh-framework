# DEPRECATED Testing Documentation

**Status**: These documents are deprecated as of 2026-02-24.

## What Happened?

The testing framework was completely reimagined. The old focus on:

- Bash automation scripts for OpenClaw CLI
- CI/CD integration
- Code coverage and unit tests
- Automated test runners

...was replaced with **behavioral testing** using Claude Code as the primary testbed.

## Why the Change?

1. **Most valuable testing was behavioral**: Observing how agents use skills/hooks
2. **Claude Code provides superior test environment**: Safe, instant, controllable
3. **OpenClaw testing is still needed**: But as production integration, not development
4. **Scripts were theoretical**: The bash templates were docs, not runnable code
5. **Real focus**: Agent behavior > code coverage

## The New Framework

See the parent directory `06-testing/` for the new behavioral testing docs:

- `01-behavioral-testing-approach.md` - Philosophy and strategy
- `02-claude-code-testbed.md` - How to use Claude Code for testing
- `03-test-workflows.md` - Practical workflows
- `04-test-cases-templates.md` - Copy-paste test templates

## Migration

- Old automation scripts → Not applicable (behavioral testing doesn't use them)
- CI/CD integration → Separate from behavioral testing (still useful for code quality)
- OpenClaw CLI tests → Still valid for production integration (see 03-openclaw-testing.md)
- Unit tests → Still valuable for CLI tools separately

## Keep These Old Docs for Reference?

These docs are kept in archive for:
- Historical reference
- OpenClaw integration testing concepts (still partially valid)
- Example of what NOT to do (scripts that were never executable)

Do not use these for new plugin development. Use the new behavioral testing framework instead.

---

*Archive created: 2026-02-24*
