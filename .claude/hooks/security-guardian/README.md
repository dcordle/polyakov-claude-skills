# Security Guardian

A system of protective hooks for Claude Code that makes it safe to work in `--dangerously-skip-permissions` mode.

## Protection philosophy

**Main principle: directory boundaries matter more than filename patterns.**

| Level | Protection | Reliability |
|-------|------------|-------------|
| **1. Directory boundaries** | Any operation outside the project → block | High |
| **2. Path allowlist** | The user explicitly adds permitted directories | High |
| **3. Name patterns** | An extra layer for typical secrets inside the project | Medium |

## Installation

### 1. Install dependencies

```bash
cd .claude/hooks/security-guardian
uv sync
# or
pip install -e .
```

### 2. Add the hooks to settings.json

Copy the contents of `settings.template.json` into your `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Read|Write|Edit|Glob|Grep|NotebookEdit",
        "hooks": [
          {
            "type": "command",
            "command": "uv run \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-guardian/main.py\" 2>/dev/null || python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-guardian/main.py\"",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

### 3. Adjust the configuration (optional)

Edit `config/security_config.yaml` to fit your needs:

```yaml
directories:
  allowed_paths:
    - "${HOME}/Documents/shared-libs"
    - "/tmp/claude"
```

## Protections

### Directory boundaries (PRIMARY)
- Reading/writing outside the project → block
- Symlinks pointing outside the project → block (via realpath)
- Relative paths like `../../../etc` → block

### Git operations
- `git push --force` → block (suggest `--force-with-lease`)
- `git reset --hard` → confirm
- `git branch -D` → confirm
- `git clean -fd` → confirm (but `--dry-run` is allowed)

### Bypass attempts
- `eval` → block
- `$VAR` as a command → block
- `curl | bash` → block
- `sh -c "..."` → block

### File downloads
- Executable files (.py, .sh, .exe) → give the command to the user
- Archives → allowed, but unpacking is controlled

### Secrets
- `.env` files → reading is blocked
- `.env.example` → allowed

## Testing

```bash
uv run -m pytest
```

## Structure

```
security-guardian/
├── main.py               # Entry point
├── pyproject.toml        # Dependencies
├── config/
│   └── security_config.yaml
├── parsers/
│   ├── bash_parser.py    # AST parsing via bashlex
│   └── path_parser.py    # Path resolution
├── checks/
│   ├── directory_check.py    # Directory boundaries (PRIMARY)
│   ├── git_check.py          # Git operations
│   ├── deletion_check.py     # File deletion
│   ├── bypass_check.py       # Bypass attempts
│   ├── download_check.py     # File downloads
│   ├── unpack_check.py       # Archive unpacking
│   ├── execution_check.py    # chmod +x
│   └── secrets_check.py      # Secrets
├── handlers/
│   ├── bash_handler.py
│   ├── read_handler.py
│   ├── write_handler.py
│   └── glob_grep_handler.py
└── messages/
    └── guidance.py       # Hints for Claude
```

## Logging

Logs are saved to `~/.claude/logs/security-guardian/`.

They contain only metadata (command, path, reason for blocking), not file contents.
