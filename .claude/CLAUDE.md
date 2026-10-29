# Rules for the polyakov-claude-skills project

## plugin.json: manifest format

When creating or editing `.claude-plugin/plugin.json`:

- **`author`**: MUST be an object: `{"name": "Polyakov"}`. A string causes a validation error.
- **`skills`**: NOT a valid field. Do not add it. Skills are discovered automatically from the `skills/` directory.
- Reference format:
  ```json
  {
    "name": "plugin-name",
    "version": "1.0.0",
    "description": "...",
    "author": {
      "name": "Polyakov"
    }
  }
  ```

## marketplace.json

- When adding a new plugin, you must add an entry to `.claude-plugin/marketplace.json` → `plugins[]`, otherwise the plugin will not be visible through `/plugin install`.

## Skill file structure

Every skill must follow the official file structure:

```
your-skill-name/
├── SKILL.md           # Required: the main skill file
├── scripts/           # Optional: executable code
├── references/        # Optional: extra documentation
└── assets/            # Optional: templates, etc.
```

- **Extra documentation** (detailed guides, references) goes ONLY in `references/`, NOT in the skill root next to SKILL.md.
- In SKILL.md, link to extra documents like this: `[doc.md](references/doc.md)`.
- SKILL.md should contain only the core instructions (progressive disclosure).

## Shell scripts for the Claude cloud sandbox

Skill scripts run in the Claude cloud sandbox. It has restrictions that break standard bash approaches. The rules below come from practice.

### Shell and compatibility

- Shebang: `#!/bin/sh`, not `#!/bin/bash`. Bash may be missing in the sandbox.
- No bashisms: `[[ ]]` → `[ ]`, `${BASH_SOURCE[0]}` → `$0`, `source` → `.`, `local` → variables prefixed with `_funcname_`.
- No external `source common.sh`: inline everything into the script. The sandbox may not allow reading neighboring files.
- Reference: `plugins/fal-ai-image/skills/fal-ai-image/scripts/` (commit `6410d65`).

### The stdout buffer is the main trap

- The sandbox limits stdout size (~4-64 KB, the exact value is in issue #48).
- If a script prints more than the limit, it fails **silently**: "Error running command", with no output and no stderr.
- Symptom: small scripts (quota.sh, ~200 bytes of output) work, big ones (top_requests.sh with ~50 KB of JSON) do not.

### Solution: a temp file instead of a variable

```sh
# BAD: API response in a variable, echo | grep breaks on large JSON
result=$(curl -s ...)
echo "$result" | grep ...

# GOOD: response in a file, grep reads the file directly
TMPFILE="${TMPDIR:-/tmp}/result_$$.json"
trap 'rm -f "$TMPFILE"' EXIT
curl -s ... | tr -d '\n\r' > "$TMPFILE"
grep ... "$TMPFILE"
```

- `tr -d '\n\r'` normalizes the JSON into one line for safe parsing with grep/sed.
- `trap cleanup EXIT` guarantees the file is removed.
- The `$$` variable (PID) makes the file name unique.

### Checklist for creating a new script

1. `#!/bin/sh` + `set -e`
2. Inline config: `[ -f "$CONFIG" ] && . "$CONFIG"`
3. API response → temp file, not a variable
4. All data reading is grep/sed on a file, not `echo "$var" | ...`
5. Minimal stdout: tables of ≤20 lines, the rest goes to CSV/a file
6. No `local`, `[[ ]]`, `source`, or bash arrays
