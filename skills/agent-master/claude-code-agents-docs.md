# Claude Code Subagents — Reference Documentation

## File Structure

Subagent files use YAML frontmatter + Markdown system prompt:

```markdown
---
name: my-agent
description: What this agent does and when to use it
tools: Read, Grep, Glob
model: sonnet
---

You are a specialist agent. Your system prompt goes here.

When invoked:
1. Step one
2. Step two
```

**Locations (priority high → low, higher wins on name conflict):**

| Location | Scope | Priority |
|----------|-------|----------|
| Managed settings | Organization-wide | 1 (highest) |
| `--agents` CLI flag (JSON) | Current session only | 2 |
| `.claude/agents/<name>.md` | Current project | 3 |
| `~/.claude/agents/<name>.md` | All your projects | 4 |
| Plugin `agents/` directory | Where plugin enabled | 5 (lowest) |

Subagents load at session start. After creating a file, run `/agents` or restart to load immediately. Agents created via `/agents` interface take effect immediately.

**Discovery & identity:**
- `.claude/agents/` and `~/.claude/agents/` scanned recursively — organize into subfolders (`agents/review/`, `agents/research/`). Subfolder path does NOT affect identity; identity comes only from `name` frontmatter. Keep `name` unique across the whole tree (duplicate names in one scope → one silently discarded).
- `--add-dir` directories grant file access only — NOT scanned for agents. Share via `~/.claude/agents/` or a plugin.
- Plugin `agents/` also scanned recursively, but subfolder becomes part of the scoped identifier: `agents/review/security.md` in plugin `my-plugin` → `my-plugin:review:security`.

**Plugin subagent restrictions:** plugin subagents ignore `hooks`, `mcpServers`, and `permissionMode` (security). To use those fields, copy the agent into `.claude/agents/` or `~/.claude/agents/`.

---

## Built-in Subagents

Available without creating anything — Claude delegates automatically, and they are valid `agent:` targets for forked skills.

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| `Explore` | Haiku | Read-only (no Write/Edit) | File discovery, code search. Skips CLAUDE.md + git status |
| `Plan` | Inherit | Read-only (no Write/Edit) | Codebase research during plan mode. Skips CLAUDE.md + git status |
| `general-purpose` | Inherit | All | Complex multi-step tasks needing exploration + action |
| `statusline-setup` | Sonnet | — | Configures status line (`/statusline`) |
| `claude-code-guide` | Haiku | — | Answers Claude Code feature questions |

Subagents cannot spawn other subagents (no nesting). Use chaining from the main conversation or Skills instead.

---

## Frontmatter Fields

### Required

| Field | Description |
|-------|-------------|
| `name` | Unique identifier. Lowercase letters and hyphens only. |
| `description` | When Claude should delegate to this agent. Keyword-rich. Include "Use proactively" to encourage auto-delegation. |

### Tool Access

| Field | Description |
|-------|-------------|
| `tools` | Allowlist — only these tools available. Omit to inherit all. |
| `disallowedTools` | Denylist — inherit all except these. Applied before `tools`. |

If both are set: `disallowedTools` applied first, then `tools` resolves against remainder. A tool in both is removed.

**Unavailable to subagents** (depend on main UI/session state — listing in `tools` has no effect): `Agent`, `AskUserQuestion`, `EnterPlanMode`, `ScheduleWakeup`, `WaitForMcpServers`, and `ExitPlanMode` (unless `permissionMode: plan`).

**Tool syntax examples:**
```yaml
tools: Read, Grep, Glob, Bash
tools: Read, Edit, Bash(git *), Bash(npm test)
disallowedTools: Write, Edit
```

**Agent spawning restriction** (only relevant when agent runs as main thread via `--agent`):
```yaml
tools: Agent(worker, researcher), Read, Bash   # only these subagent types
tools: Agent, Read, Bash                        # any subagent
# omit Agent entirely = cannot spawn subagents
```
(Tool `Task` renamed to `Agent` in v2.1.63; `Task(...)` still works as alias.)

### Model

| Value | Behavior |
|-------|----------|
| `inherit` | Same model as main conversation (default if omitted) |
| `sonnet` | claude-sonnet alias |
| `opus` | claude-opus alias |
| `haiku` | claude-haiku alias |
| Full ID | e.g. `claude-sonnet-4-6`, `claude-opus-4-7` |

Model resolution order (highest wins):
1. `CLAUDE_CODE_SUBAGENT_MODEL` env var
2. Per-invocation model parameter
3. Subagent `model` frontmatter
4. Main conversation model

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Standard permission checking with prompts |
| `acceptEdits` | Auto-accept file edits and common filesystem commands in working dir |
| `auto` | Background classifier reviews commands and protected-directory writes |
| `dontAsk` | Auto-deny permission prompts (explicitly allowed tools still work) |
| `bypassPermissions` | Skip all permission prompts — use with caution |
| `plan` | Read-only exploration mode |

**Parent override rules:**
- If parent uses `bypassPermissions` or `acceptEdits` → takes precedence, cannot be overridden
- If parent uses `auto` → subagent inherits `auto`, `permissionMode` in frontmatter is ignored

### Memory (Persistent Across Sessions)

| Scope | Location | Use when |
|-------|----------|----------|
| `user` | `~/.claude/agent-memory/<name>/` | Knowledge applicable across all projects |
| `project` | `.claude/agent-memory/<name>/` | Project-specific, shareable via version control |
| `local` | `.claude/agent-memory-local/<name>/` | Project-specific, not in version control |

When `memory` is set:
- System prompt gains instructions for reading/writing to memory directory
- First 200 lines or 25KB of `MEMORY.md` injected into context
- `Read`, `Write`, `Edit` tools auto-enabled for memory management

**Best practice:** Include in system prompt:
```
Update your agent memory as you discover codepaths, patterns, library
locations, and key architectural decisions.
```

### Other Fields

| Field | Type | Description |
|-------|------|-------------|
| `maxTurns` | integer | Maximum agentic turns before agent stops |
| `skills` | list | Skill names to preload into context at startup. Full skill content injected — not just made available |
| `mcpServers` | list | MCP servers for this agent. Inline definitions (scoped to agent) or string references (shared from session) |
| `hooks` | map | Lifecycle hooks scoped to this agent |
| `background` | bool | Always run as background task. Default: false |
| `effort` | string | `low`, `medium`, `high`, `xhigh`, `max` — overrides session effort level |
| `isolation` | string | `worktree` — run in isolated git worktree copy, auto-cleaned if no changes made |
| `color` | string | Display color: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `initialPrompt` | string | Auto-submitted as first user turn when agent runs as main session (via `--agent`). Commands and skills processed. Prepended to user-provided prompt. |

---

## MCP Servers

```yaml
mcpServers:
  # Inline definition — scoped to this agent only
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # String reference — reuses already-configured server
  - github
```

Inline definitions use same schema as `.mcp.json` (stdio, http, sse, ws). They connect when agent starts, disconnect when it finishes. Use inline to keep server tools out of the main conversation context.

---

## Hooks in Frontmatter

Hooks run while the agent is active (subagent or main session via `--agent`):

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
  Stop:
    - hooks:
        - type: command
          command: "./scripts/cleanup.sh"
```

`Stop` hooks in frontmatter are automatically converted to `SubagentStop` events when running as subagent.

**Hook input:** JSON via stdin. Extract with jq:
```bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
```

**Exit codes:**
- `0` — allow
- `2` — block (stderr returned to Claude as error message)

---

## Project-Level Hooks for Subagent Events

In `settings.json`, respond to subagent lifecycle:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [{ "type": "command", "command": "./scripts/setup-db.sh" }]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [{ "type": "command", "command": "./scripts/cleanup.sh" }]
      }
    ]
  }
}
```

---

## Skills Preloading

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

Full skill content injected at startup. Subagents don't inherit skills from parent — must list explicitly. Cannot preload skills with `disable-model-invocation: true`.

---

## Invocation Patterns

| Pattern | Example | Effect |
|---------|---------|--------|
| Natural language | "Use the code-reviewer agent" | Claude decides whether to delegate |
| @-mention | `@"code-reviewer (agent)"` | Guarantees that agent runs |
| Session-wide | `claude --agent code-reviewer` | Entire session uses agent's system prompt |
| Settings default | `{ "agent": "code-reviewer" }` in `.claude/settings.json` | Default for every session in project |

**@-mention syntax:**
- Local: `@agent-<name>`
- Plugin: `@agent-<plugin-name>:<agent-name>`

---

## Foreground vs Background

| | Foreground | Background |
|-|------------|------------|
| Execution | Blocks until complete | Concurrent with main conversation |
| Permission prompts | Passed through to user | Pre-approved before launch, then auto-denied |
| AskUserQuestion | Passed through | Fails silently, agent continues |

Force background: ask Claude "run this in the background" or press `Ctrl+B`.

Disable all background: `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`.

---

## Disable Specific Agents

In `settings.json`:
```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

Or via CLI: `claude --disallowedTools "Agent(Explore)"`

---

## Fork Subagents (Experimental)

Enable: `CLAUDE_CODE_FORK_SUBAGENT=1`

A fork inherits the full conversation context instead of starting fresh — same system prompt, tools, model, message history. Use when a named agent would need too much background re-explanation.

| | Fork | Named Subagent |
|-|------|----------------|
| Context | Full conversation history | Fresh — only what you pass |
| System prompt | Same as main session | From agent's definition file |
| Model | Same as main session | From agent's `model` field |
| Prompt cache | Shared with main session | Separate cache |

Start a fork manually: `/fork draft unit tests for the parser changes so far`

Limitations: fork cannot spawn further forks.

---

## CLI-Defined Agents (`--agents`)

Pass JSON at launch — ephemeral, session-only, not saved to disk. Good for testing/automation. `prompt` = system prompt (equivalent to markdown body).

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on quality, security, best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

Accepts same fields as file frontmatter: `description`, `prompt`, `tools`, `disallowedTools`, `model`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `skills`, `initialPrompt`, `memory`, `effort`, `background`, `isolation`, `color`.

---

## What Loads at Startup

A non-fork subagent starts fresh — does NOT see conversation history, prior skills, or files already read. Initial context contains:

- **System prompt** — agent's own prompt + environment details (NOT the full Claude Code system prompt)
- **Task message** — delegation prompt Claude writes at handoff
- **CLAUDE.md + memory hierarchy** — all levels the main conversation loads. *Explore and Plan skip this.*
- **Git status** — snapshot from parent session start. *Explore and Plan skip this.*
- **Preloaded skills** — full content of any skill in `skills:` field

Implication: if a rule must reach the agent (e.g. "ignore `vendor/`"), restate it in the delegation prompt — agents don't inherit your conversation. Explore/Plan are the only agents that skip CLAUDE.md + git; no setting changes this.

---

## Resume Subagents

Each invocation = new instance, fresh context. To continue prior work, ask Claude to resume — the resumed agent retains full history (tool calls, results, reasoning).

```text
Use the code-reviewer subagent to review the auth module
[completes]
Continue that review and now analyze the authorization logic
```

Transcripts persist per-session at `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`, unaffected by main-conversation compaction. Cleaned up per `cleanupPeriodDays` (default 30).

---

## Subagent vs Skill vs Main Conversation

| Use | When |
|-----|------|
| **Main conversation** | Frequent back-and-forth, shared context across phases, quick targeted change, latency matters |
| **Subagent** | Verbose output you don't need in main context, enforce tool restrictions, self-contained work returning a summary |
| **Skill** (`context: fork` inverse) | Reusable prompt/workflow that should run in main context, not isolated. See skill-master. |

Subagent = isolated context, own system prompt. Forked skill = inject skill content into a chosen agent. Both share the same underlying system.

---

## Example Agents

### Code Reviewer (Read-Only)
```yaml
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code clarity and readability
- Well-named functions and variables
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation
- Test coverage
- Performance considerations

Provide feedback by priority: Critical → Warnings → Suggestions.
Include specific fix examples.
```

### Debugger (Read + Write)
```yaml
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Focus on fixing the underlying issue, not the symptoms.
```

### Data Scientist
```yaml
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Write optimized queries. Include comments for complex logic. Provide data-driven recommendations.
```

### DB Reader (Hook-Validated)
```yaml
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries only.
You cannot modify data. If asked to INSERT, UPDATE, DELETE, or modify schema, explain you only have read access.
```

Validation script (`./scripts/validate-readonly-query.sh`):
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
if [ -z "$COMMAND" ]; then exit 0; fi
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2
fi
exit 0
```

Make executable: `chmod +x ./scripts/validate-readonly-query.sh`
