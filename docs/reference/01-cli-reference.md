# CLI Reference

Commands and conventions for framework CLI tools.

---

## Command Design Principles

### The 5-10 Command Sweet Spot

| Category | Count | Example Commands |
|----------|-------|------------------|
| Core | 5-10 | init, install, status, help, version |
| Resource | 5-10 | create, list, get, update, delete |
| Action | 5-10 | start, stop, run, build, deploy |

### Command Hierarchy

```
framework <noun> <verb>

framework plugin list
framework skill create
framework hook register
```

---

## Standard Commands

### init

Initialize a new plugin or project.

```bash
framework init my-plugin
```

| Flag | Description |
|------|-------------|
| `--template` | Use template (default: skeleton) |
| `--force` | Overwrite existing |

### install

Install a plugin or skill.

```bash
framework install <plugin-name>
framework install ./local-plugin
```

### list

List available plugins, skills, or hooks.

```bash
framework list plugins
framework list skills
framework list hooks
```

### status

Show framework or plugin status.

```bash
framework status
framework status my-plugin
```

### help

Show help for command.

```bash
framework help <command>
framework <command> --help
```

---

## Output Formats

### Default (Human-Readable)

```
Plugins:
- my-plugin (1.0.0) [enabled]
- other-plugin (2.1.0) [disabled]
```

### JSON (`--json`)

```bash
framework list plugins --json
```

```json
{
  "plugins": [
    {"name": "my-plugin", "version": "1.0.0", "enabled": true},
    {"name": "other-plugin", "version": "2.1.0", "enabled": false}
  ]
}
```

---

## Exit Codes

| Code | Meaning | Example |
|------|---------|---------|
| 0 | Success | Command completed |
| 1 | Generic error | Unexpected failure |
| 2 | Invalid usage | Missing required argument |
| 3 | Config error | Invalid configuration |
| 4 | Not found | Resource doesn't exist |
| 5 | Permission denied | Access issue |
| 127 | Unknown command | Command not found |

---

## Configuration

### Config File Location

- `~/.csh/config` — Global config
- `./.cshrc` — Project config

### Config Format

```yaml
plugins:
  enabled:
    - my-plugin
  disabled: []

skills:
  auto_load: true
  directories:
    - ~/.csh/skills
    - ./.skills

hooks:
  log_level: info
```

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CSH_HOME` | Framework home (default: ~/.csh) |
| `CSH_CONFIG` | Config file path |
| `CSH_DEBUG` | Enable debug output |

---

## Common Patterns

### List with Filter

```bash
framework list plugins --enabled
framework list skills --tag database
```

### Verbose Output

```bash
framework status --verbose
# or
framework status -v
```

### Dry Run

```bash
framework run --dry-run
```
