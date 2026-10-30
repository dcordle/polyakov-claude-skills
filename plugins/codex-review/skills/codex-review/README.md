# Codex Review Plugin

Cross-agent review: Claude Code implements, Codex (GPT) reviews.

> **IMPORTANT!** The plugin scripts keep all state (session, config, review log) in the `.codex-review/` directory **at the root of your project**, not next to themselves. The `config/` directory inside the plugin is only a template. Do not edit files in the plugin installation directory (`~/.claude/plugins/...`): they are overwritten on update.

## Installation

### Option A: via marketplace (recommended)

1. Add the repository as a marketplace (once):

```bash
# From a local directory
claude plugin marketplace add /path/to/polyakov-claude-skills

# Or from GitHub
claude plugin marketplace add github:artwist-polyakov/polyakov-claude-skills
```

2. Install the plugin:

```bash
claude plugin install codex-review@polyakov-claude-skills
```

### Option B: for a single session

```bash
claude --plugin-dir /path/to/polyakov-claude-skills/plugins/codex-review
```

### Dependencies

Make sure the `codex` CLI is installed:

```bash
npm install -g @openai/codex
```

## Project setup

### .gitignore

Add this to the project's `.gitignore` (or `.git/info/exclude`):

```
.codex-review/
```

The whole directory, including `notes/`, holds local review state and must not end up in Git.

If the project followed the earlier recommendation and already tracks `notes/`, after updating `.gitignore` remove the directory from the Git index only:

```bash
git rm -r --cached --ignore-unmatch .codex-review/
```

The local files are kept. Commit the staged deletions together with the new `.gitignore` rule.

### AGENTS.md (for Codex)

Add a section to the project's `AGENTS.md`:

```markdown
## Review Protocol

If you act as a reviewer (launched via the codex-review workflow):
- Give concrete, actionable feedback
- You may inspect the code/diff yourself
- Do not run scripts from skills/codex-review/: you are the reviewer
- Do not look into .codex-review/archive/: it holds artifacts of past sessions
- After the review, write the verdict to the path given in the review prompt (one word: APPROVED or CHANGES_REQUESTED)
```

### settings.local.json

Add permissions to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(bash */codex-review.sh:*)",
      "Bash(bash */codex-state.sh:*)",
      "Bash(codex exec:*)"
    ]
  }
}
```

### Configuration (optional)

Create `.codex-review/config.env` at the project root:

```bash
# Existing Codex session (or use init to create a new one)
# CODEX_SESSION_ID=sess_your_session_id

# Base option. Without this line, the model from the Codex settings
# or its current default is used.
CODEX_MODEL=gpt-5.5

# Current GPT-5.6 family: keep one option instead of the line above:
# CODEX_MODEL=gpt-5.6-sol    # maximum quality for complex reviews
# CODEX_MODEL=gpt-5.6-terra  # balanced everyday option
# CODEX_MODEL=gpt-5.6-luna   # fast, well-bounded tasks

# Usually: low, medium, high, xhigh; max is supported by GPT-5.6 models.
CODEX_REASONING_EFFORT=high

# Accelerated service tier. Off by default.
CODEX_FAST_MODE=false

CODEX_MAX_ITERATIONS=5
CODEX_YOLO=true

# Auto-review mode: block ExitPlanMode until Codex approves the plan,
# auto-run code review after implementation.
# AUTO_REVIEW=true

# Custom init procedure (optional, controls what Codex does during init)
# Reviewer role is always set automatically — this only adds init instructions.
# Example: make Codex explore the codebase before reviews begin:
# CODEX_REVIEWER_PROMPT="Explore the codebase areas relevant to the task. Understand the architecture, patterns, and conventions so you are prepared to review."

# Additional guidance for plan review phase (optional, appended to built-in focus areas)
# CODEX_PLAN_GUIDE="Verify backward compatibility with API v1 clients"

# Additional guidance for code review phase (optional, appended to built-in focus areas)
# CODEX_CODE_GUIDE="Check that all DB queries use parameterized statements"

# Severity calibration (default: true) — see "Severity and verdict" below.
# CODEX_SEVERITY_CALIBRATION=false
```

`CODEX_FAST_MODE=true` passes `service_tier="fast"` when the session is created and on every review. The mode takes effect if the selected model and account support it. With `false` or when the setting is absent, the plugin does not override the Codex service tier.

### Severity and verdict

The reviewer rates each finding as `critical`, `important` or `minor` and sorts the findings into three sections of the response: `## Blocking` (`critical` and `important` in the current work), `## Non-blocking` (`minor` in the current work) and `## Pre-existing` (defects in code that was touched but not changed, on the same scale, keeping the level).

`CHANGES_REQUESTED` is returned only when `## Blocking` is not empty. `APPROVED` with a list in `## Non-blocking` is a normal verdict.

From the third round on, the reviewer reports on the blockers of previous rounds and opens a new topic only at the `critical` level.

`CODEX_SEVERITY_CALIBRATION=false` restores the previous prompt, where the verdict can be held up by any remark.

## Usage

### Connecting an existing Codex session

If you already have a live session with Codex (for example, you discussed the architecture), put its id into `.codex-review/config.env`:

```bash
CODEX_SESSION_ID=sess_your_id
```

Alternatively, via the CLI: `bash scripts/codex-state.sh set session_id sess_your_id`

After that, the `plan` and `code` commands will send reviews to this session via `resume`.

### Creating a new session

```
"Используем workflow с codex ревьювером. Задачи: #23, #10" (We use the workflow with the Codex reviewer. Tasks: #23, #10)
```

Claude calls `init`, which creates a Codex session. By default init is lightweight (Codex confirms it is ready). With `CODEX_REVIEWER_PROMPT` in config.env, init runs a custom procedure (for example, exploring the codebase). The reviewer role is set automatically. Then `plan` and `code` send reviews to this session via `resume`.

### Workflow

1. **Init** - Claude creates a Codex session (`init`)
2. **Plan Review** - Claude describes the plan, Codex reviews it (`plan`)
3. **Implementation** - Claude implements according to the approved plan
4. **Code Review** - Claude describes the changes, Codex reviews them (`code`)
5. **Done** - the result goes to the user

### State management

```bash
bash scripts/codex-state.sh show          # Current state
bash scripts/codex-state.sh dir           # Path to the current branch's state directory
bash scripts/codex-state.sh reset         # Reset iterations
bash scripts/codex-state.sh reset --full  # Full reset
bash scripts/codex-state.sh set session_id <value>  # Set manually
bash scripts/codex-state.sh set phase implementing  # Update the phase
bash scripts/codex-state.sh set iteration 2        # Roll back the round counter
```

Writable fields: `session_id`, `phase`, `iteration`, `max_iterations`,
`reviews_completed`, `last_review_status`, `last_review_timestamp`,
`task_description`. An unknown field name, a non-integer counter value, and a value
containing a quote, backslash, newline or tab is an error with return
code 1, and the state is not changed.

## Structure of .codex-review/

A directory with per-branch isolation is created at the root of the main repo (not a worktree). All of its content is local:

```
.codex-review/
├── config.env                  # shared local project settings
├── .gitkeep
├── archive/                    # local archive of all sessions
│   └── {timestamp}/            # artifacts of one session (branch in summary.json)
├── feat-auth/                  # per-branch state (branch name, / → -)
│   ├── state.json              # transient state
│   ├── STATUS.md               # auto-generated status for Claude
│   ├── verdict.txt             # latest verdict from Codex
│   ├── last_response.txt       # latest Codex response
│   ├── current_session.txt     # binding of the verdict to the Claude session
│   ├── codex-init.log          # session initialization log
│   ├── codex-init.request.md   # task text sent to the session
│   ├── codex-init.prompt.md    # prompt that Codex received
│   ├── codex-{phase}-{N}.log   # review iteration logs
│   ├── codex-{phase}-{N}.request.md  # text sent for the iteration
│   ├── codex-{phase}-{N}.prompt.md   # prompt that Codex received
│   ├── plan.md                 # copy of the last plan sent for review
│   └── notes/                  # local log of the current review
│       ├── .gitkeep
│       ├── plan-review-1.md
│       └── code-review-1.md
└── feat-ui/                    # another branch: full isolation
    └── ...
```

## CLAUDE.md

Add this to the project's CLAUDE.md (one-time setup):

```markdown
## Codex Review
Check for `.codex-review/*/STATUS.md` — if a STATUS.md exists for the current branch, read it before starting work (an active review is in progress).
```

`STATUS.md` is created and updated automatically by the plugin scripts in the branch's state directory (path: `codex-state.sh dir`). The presence of the file means a review is active, its absence means no review is running or it has finished.

## Git Worktree Support

The plugin works transparently from git worktrees:

- `.codex-review/` is always resolved to the main repository root via `git rev-parse --git-common-dir`
- Review state is isolated per branch — each branch gets its own subdirectory (e.g. `.codex-review/feat-auth/`)
- Multiple worktrees on different branches can run reviews in parallel without conflicts
- `config.env` is shared across all branches (project-level settings)
- No additional setup required

## Auto-Review Mode

When `AUTO_REVIEW=true` in `.codex-review/config.env`, the plugin enforces automated review:

- **Plan phase**: a plugin hook blocks `ExitPlanMode` until Codex approves the plan **AND** the approval was issued in the current Claude session. The hook binds each plan review to the Claude session that ran it, so a stale verdict from a previous task or session cannot silently auto-approve a new plan. If the verdict is missing, stale, not approved, or from a different session, the hook denies the exit and instructs Claude to load the codex-review skill and run plan review first.
- **Code phase**: after implementation, Claude automatically sends code for review and iterates until approved.

No additional configuration needed — the hook is declared in `plugin.json` and auto-registered when the plugin is enabled.

## Anti-recursion

The plugin is protected against recursive invocation at 3 levels:

1. **Env guard** - `CODEX_REVIEWER=1` when calling codex exec; if the script is called with this variable, it exits with 1
2. **Prompt context** - the skill path in the prompt, for orientation
3. **AGENTS.md** - an instruction for Codex about the reviewer role
