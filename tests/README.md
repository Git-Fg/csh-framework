# Testing Infrastructure

This directory contains the **automated test suite** that complements the manual behavioral testing.

## What's Tested Here

- ✅ **Skill parsing**: SKILL.md frontmatter validation
- ✅ **CLI validation**: Binary existence, --help, --json flags, exit codes
- ✅ **Hook configuration**: OpenClaw and Claude Code manifest validation
- ✅ **Plugin manifest**: Structure and required fields
- ✅ **Behavioral logging**: Test session record/export utilities

## What's NOT Tested Here

- ❌ Agent behavior in conversation (use Claude Code testbed)
- ❌ Multi-skill chaining (manual testing)
- ❌ Hook context injection (manual testing)
- ❌ Edge case scenarios (manual testing)

That's behavioral testing, see `docs/openclaw/` for that methodology.

---

## Structure

```
tests/
├── unit/                    # Unit tests for framework utilities
│   ├── skill-parser.test.ts
│   ├── cli-validator.test.ts
│   ├── behavioral-logger.test.ts
│   └── ...
├── integration/            # Integration tests
│   └── plugin-manifest.test.ts
├── fixtures/               # Test data
│   ├── valid-plugin/
│   └── invalid-plugin/
└── templates/              # Manual test templates
    └── test-session-template.md
```

---

## Running Tests

```bash
# Install dependencies first
npm install

# Run all tests
npm test

# Run only unit tests
npm run test:unit

# Run only integration tests
npm run test:integration

# Watch mode (auto-reload on changes)
npm run test:watch

# With coverage
npm run test:coverage
```

---

## Expected Coverage

Target: **80% statements, 75% branches, 80% functions, 80% lines**

Current coverage goals for:
- `src/skill-parser.ts`: 90%+
- `src/cli-validator.ts`: 85%+
- `src/hook-validator.ts`: 90%+
- `src/behavioral-logger.ts`: 95%+

---

## Writing New Tests

### Unit Test Pattern

```typescript
import { describe, it, expect } from 'vitest';
import { yourFunction } from '../../src/your-module';

describe('YourModule', () => {
  it('should handle valid input', () => {
    const result = yourFunction('input');
    expect(result).toEqual(expected);
  });

  it('should throw on invalid input', () => {
    expect(() => yourFunction(null)).toThrow('Expected error');
  });
});
```

### Integration Test Pattern

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { join } from 'path';
import { existsSync, mkdirSync, writeFileSync, rmSync } from 'fs';

describe('Plugin Manifest Integration', () => {
  const testDir = '/tmp/test-plugin';

  beforeAll(() => {
    mkdirSync(testDir, { recursive: true });
    // Setup test plugin structure
  });

  afterAll(() => {
    rmSync(testDir, { recursive: true, force: true });
  });

  it('should validate correct manifest', () => {
    // Test code
  });
});
```

---

## Test Quality Guidelines

1. **Isolated**: Each test should be independent, not rely on other tests' state
2. **Deterministic**: No random behavior or timing dependencies
3. **Clear**: Test name should explain what and why
4. **Fast**: Tests should complete in milliseconds, not seconds
5. **Coverage**: Aim for high coverage but focus on critical paths

---

## Behavioral Testing

For agent behavior validation (skills, hooks, chaining), see:

- `docs/openclaw/claude-code-testbed.md`
- `docs/openclaw/test-workflows.md`
- `tests/templates/test-session-template.md`

---

## CI/CD Integration

Example GitHub Actions workflow (to be added to `.github/workflows/test.yml`):

```yaml
- name: Install dependencies
  run: npm ci

- name: Run tests
  run: npm test

- name: Generate coverage
  run: npm run test:coverage

- name: Upload coverage
  uses: codecov/codecov-action@v3
```

---

*Automated tests complement manual behavioral testing. Both are required for production-ready plugins.*
