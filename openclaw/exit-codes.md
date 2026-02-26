---
title: "Exit Codes"
summary: "Standard exit codes for CLI tools in CSH Framework - success, errors, validation, and resource codes."
read_when:
  - You are designing CLI error handling
  - You want consistent exit codes
  - You need to handle errors properly
---

# Exit Codes

Standard exit codes for CLI tools.

## Standard Exit Codes

| Code | Meaning | Use For |
|------|---------|---------|
| 0 | Success | Normal completion |
| 1 | General error | Unexpected failures |
| 2 | Invalid arguments | Usage errors |
| 3 | Permission denied | Access issues |
| 4 | Resource not found | File/vault/memory not found |
| 5 | Invalid state | Not initialized, wrong mode |
| 6 | Validation failed | Input validation errors |
| 7 | Timeout | Operation timed out |
| 127 | Command not found | Missing dependencies |

## Examples

```bash
# Success
exit 0

# Usage error
echo "Usage: mytool <command>" >&2
exit 2

# Not found
echo "Error: File not found: $FILE" >&2
exit 4

# Validation
echo "Error: Invalid email format" >&2
exit 6
```

## Best Practices

1. **Use stderr** for error messages
2. **Be specific** - don't just exit 1
3. **Document** - add `--help` explaining codes
4. **Consistent** - same code = same error type
