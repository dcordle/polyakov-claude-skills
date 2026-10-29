---
name: agent-deck
description: |
  Manage AI agent sessions through the agent-deck CLI.
  Triggers (RU): "запусти агента" (launch an agent), "запусти саб-агента" (launch a sub-agent), "создай сессию" (create a session),
  "проверь сессию" (check the session), "проверь статус" (check the status), "покажи вывод агента" (show agent output), "что агент ответил" (what the agent replied).
  Triggers (EN): "launch sub-agent", "create sub-agent", "start session",
  "check session", "show agent output".
---

# Agent Deck CLI

Terminal session manager for AI agents. Lets you launch child Claude sessions, monitor them, and collect their results.

## Launching a sub-agent

**Triggers:** "запусти агента" (launch an agent), "запусти саб-агента" (launch a sub-agent), "launch sub-agent"

### Simple launch (CLI commands)

```bash
# Create a session
agent-deck add -t "Name" -c claude /path/to/workdir

# Create as a child session of the current agent
agent-deck add -t "Name" --parent "Parent" -c claude /path/to/workdir

# Start it
agent-deck session start "Name"

# Send the task
agent-deck session send "Name" "Your task..."
```

### Automatic launch (script)

```bash
scripts/launch-subagent.sh "Name" "Prompt" [--mcp exa] [--wait]
```

The script automatically:
- Detects the current session and profile
- Creates a child session
- Waits for Claude to initialize
- Sends the prompt

### Ways to get the result

| Mode | Command | When to use |
|-------|---------|-------------------|
| **Fire & forget** | (without --wait) | Default. Say: "Ask me when it's ready" |
| **On-demand** | `agent-deck session output "Name"` | When the user asks |
| **Blocking** | `--wait` | An immediate result is needed |

---

## Checking status

**Triggers:** "проверь сессию" (check the session), "проверь статус" (check the status), "check session"

```bash
agent-deck status                      # All sessions (summary)
agent-deck session show "Name"         # Details of a specific session
agent-deck session show -json "Name"   # JSON format
agent-deck session current             # Current session (the one we are working in)
agent-deck session current --json      # Current session as JSON
```

**Statuses:**
- `●` running
- `◐` waiting for input (waiting)
- `○` idle
- `✕` error

---

## Getting the result

**Triggers:** "покажи вывод агента" (show agent output), "что агент ответил" (what the agent replied), "show agent output"

```bash
agent-deck session output "Name"
```

---

## Attaching MCP

```bash
agent-deck mcp list                        # Available MCP servers
agent-deck mcp attach "Name" exa           # Attach MCP to a session
agent-deck session restart "Name"          # REQUIRED after attaching!
```

### Recommended MCP

| Task | MCP servers |
|--------|-------------|
| Web search | `exa`, `firecrawl` |
| Code documentation | `context7` |
| Complex reasoning | `sequential-thinking` |

---

## Session management

```bash
# Lifecycle
agent-deck session start "Name"
agent-deck session stop "Name"
agent-deck session restart "Name"

# List all sessions
agent-deck ls
agent-deck ls -json

# Remove a session
agent-deck rm "Name"
```

---

## Important rules

1. **Flags before arguments:** `session show -json name` (not `session show name -json`)
2. **After mcp attach** you must run `session restart` to apply the changes
3. **Avoid polling** results from other agents: it can interfere with the target session
4. **Session identification:** you can use the name, the ID (6+ characters) or the path

---

## Usage examples

### Launching a research agent

```bash
# Create an agent for web research
agent-deck add -t "Researcher" -c claude --mcp exa /tmp/research
agent-deck session start "Researcher"
agent-deck session send "Researcher" "Find information about the latest AI trends"
```

### Checking readiness

```bash
# Check the status
agent-deck session show "Researcher"

# If the status is ◐ (waiting), the agent has finished; get the result:
agent-deck session output "Researcher"
```
