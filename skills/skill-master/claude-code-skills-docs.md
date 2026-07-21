# Claude Code Skills Documentation

Complete reference for creating Claude Code skills.

## Skill File Structure

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── template.md        # Template for Claude to fill in (optional)
├── examples/          # Example output (optional)
│   └── sample.md
└── scripts/           # Scripts Claude can execute (optional)
    └── helper.py
```

`SKILL.md` is required. Other files are optional — reference them from `SKILL.md` so Claude knows when to load them.

## Skill Locations

| Location   | Path                                          | Applies to                     |
|:-----------|:----------------------------------------------|:-------------------------------|
| Enterprise | See managed settings                          | All users in your organization |
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`     | All your projects              |
| Project    | `.claude/skills/<skill-name>/SKILL.md`       | This project only              |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`      | Where plugin is enabled        |

**`.claude/commands/` files still work** and support the same frontmatter. Skills are preferred since they support supporting files. If a command and skill share the same name, the skill takes precedence.

**Live change detection:** Adding, editing, or removing a skill takes effect within the current session without restarting. Creating a new top-level skills directory requires restart.

**Parent/nested discovery:** Skills load from `.claude/skills/` in the starting directory and every parent up to the repo root. Claude also discovers skills from nested `.claude/skills/` on demand (e.g. `packages/frontend/.claude/skills/`).

## How a Skill Gets Its Command Name

| Location                                         | Command name source              | Example                                          |
|:-------------------------------------------------|:---------------------------------|:-------------------------------------------------|
| `~/.claude/skills/` or `.claude/skills/`         | Directory name                   | `.claude/skills/deploy-staging/` → `/deploy-staging` |
| `.claude/commands/`                              | File name without extension      | `.claude/commands/deploy.md` → `/deploy`         |
| Plugin `skills/` subdirectory                    | Directory name, plugin-namespaced| `my-plugin/skills/review/` → `/my-plugin:review` |
| Plugin root `SKILL.md`                           | Frontmatter `name` field         | `name: review` → `/my-plugin:review`             |

The `name` frontmatter field sets the display label in skill listings — it does NOT change the invokable command, except for plugin-root skills.

## SKILL.md Structure

```yaml
---
name: explain-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works.
when_to_use: Also invoke when the user says "walk me through", "how does X work", or "explain this".
allowed-tools: Read, Grep, Glob
context: fork
agent: Explore
disable-model-invocation: false
user-invocable: true
argument-hint: [filename]
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow
3. **Walk through the code**: Explain step-by-step what happens
4. **Highlight a gotcha**: What's a common mistake?
```

## Frontmatter Reference

All fields are optional (description recommended).

| Field                      | Description                                                                                                           |
|:---------------------------|:----------------------------------------------------------------------------------------------------------------------|
| `name`                     | Display name in skill listings. Does NOT change invocation command (except plugin-root). Defaults to directory name.  |
| `description`              | What the skill does and when to use it. Combined with `when_to_use`, capped at 1,536 chars in skill listing.          |
| `when_to_use`              | Additional trigger phrases/examples for Claude. Appended to `description`. Counts toward 1,536-char cap.             |
| `argument-hint`            | Hint during autocomplete. Example: `[issue-number]` or `[filename] [format]`.                                        |
| `arguments`                | Named positional args for `$name` substitution. Space-separated string or YAML list.                                 |
| `disable-model-invocation` | `true` → only you can invoke. Claude can't trigger it, description not in context. Default: `false`.                 |
| `user-invocable`           | `false` → hide from `/` menu; only Claude can invoke. Default: `true`.                                               |
| `allowed-tools`            | Tools pre-approved (no prompt) for the turn that invokes the skill. Grant clears on your next message. Space- or comma-separated string or YAML list. |
| `disallowed-tools`         | Tools removed from Claude's pool while skill is active. Clears on next user message.                                 |
| `model`                    | Model override for this skill's turn. Reverts to session model on next prompt.                                       |
| `effort`                   | Effort level override: `low`, `medium`, `high`, `xhigh`, `max`. Reverts to session default after.                   |
| `context`                  | `fork` → run in isolated subagent context.                                                                            |
| `agent`                    | Subagent type when `context: fork`. Built-ins: `Explore`, `Plan`, `general-purpose`. Or custom agent name.           |
| `hooks`                    | Hooks scoped to this skill's lifecycle.                                                                               |
| `paths`                    | Glob patterns limiting when skill auto-activates. Comma-separated string or YAML list.                               |
| `shell`                    | Shell for `!` blocks: `bash` (default) or `powershell`.                                                              |

### Field Details

#### `description` + `when_to_use`

Both fields combined are truncated at 1,536 chars in the skill listing. Put the key use case first in `description`. Use `when_to_use` for additional trigger phrases.

#### `arguments` (named args)

Named args map to positional order:

```yaml
---
arguments: [issue, branch]
---
Fix issue $issue on branch $branch.
```

`/skill 42 main` → `$issue` = `42`, `$branch` = `main`.

#### `allowed-tools`

Format:
- Tool name: `Read`, `Write`, `Edit`, `Grep`, `Glob`
- Wildcarded: `Bash(git *)`, `Bash(npm *)`
- Specific: `Bash(git status)`, `Bash(ls -la)`

The grant covers only the turn that invokes the skill and clears on your next message; invoking the skill again re-applies it for that turn. To pre-approve tools for the whole session, add allow rules to your permission settings instead.

For project skills, takes effect after you accept the workspace trust dialog.

#### `disallowed-tools`

Block specific tools while skill runs. Useful for autonomous loops that should never prompt:

```yaml
disallowed-tools: AskUserQuestion
```

Restriction clears when user sends next message.

#### `context: fork`

Runs skill in isolated subagent. Skill content becomes the subagent's prompt. No access to conversation history.

**Use when:** self-contained task, isolation needed, explicit step-by-step instructions.

**Don't use when:** skill contains only guidelines (subagent receives no actionable task).

The Explore and Plan agents skip CLAUDE.md to keep context small. Other agent types load CLAUDE.md.

#### `disable-model-invocation` vs `user-invocable`

| Frontmatter                      | You can invoke | Claude can invoke | In context                                                   |
|:---------------------------------|:---------------|:------------------|:-------------------------------------------------------------|
| (default)                        | Yes            | Yes               | Description always in context, full skill loads when invoked |
| `disable-model-invocation: true` | Yes            | No                | Description NOT in context, full skill loads when you invoke |
| `user-invocable: false`          | No             | Yes               | Description always in context, full skill loads when invoked |

## Skill Content Lifecycle

When invoked, the rendered `SKILL.md` enters the conversation as a single message and stays for the rest of the session. This persistence applies to the skill's instructions, not its permissions: an `allowed-tools` grant clears on your next message. Claude Code does NOT re-read the skill file on later turns.

**Auto-compaction:** After context is summarized, the most recent invocation of each skill is re-attached (first 5,000 tokens each). All re-attached skills share a 25,000-token combined budget, filled starting from most-recently-invoked. Older skills may be dropped if many were invoked.

If a skill stops influencing behavior after compaction, re-invoke it.

## Dynamic Content and Substitutions

### String Substitutions

| Variable               | Description                                                                |
|:-----------------------|:---------------------------------------------------------------------------|
| `$ARGUMENTS`           | All arguments as typed                                                     |
| `$ARGUMENTS[N]`        | Specific argument by 0-based index                                         |
| `$N`                   | Shorthand for `$ARGUMENTS[N]` (`$0`, `$1`, etc.)                           |
| `$name`                | Named argument from `arguments` frontmatter (maps to positional order)     |
| `${CLAUDE_SESSION_ID}` | Current session ID                                                         |
| `${CLAUDE_EFFORT}`     | Current effort level: `low`, `medium`, `high`, `xhigh`, `max`             |
| `${CLAUDE_SKILL_DIR}`  | Directory containing this `SKILL.md`. Use to reference bundled scripts.    |

Multi-word args: wrap in quotes. `/skill "hello world" second` → `$0` = `hello world`, `$1` = `second`.

If `$ARGUMENTS` is absent from skill content, args are appended as `ARGUMENTS: <value>`.

### Shell Injection: `` !`command` ``

Runs BEFORE Claude sees content. Output replaces placeholder. This is preprocessing — Claude only sees the result.

```yaml
## Context
- Diff: !`gh pr diff`
- Status: !`gh pr checks`
```

Rules:
- `!` must appear at line start or after whitespace (not after another character)
- Output is NOT re-scanned for further placeholders
- Disable with `disableSkillShellExecution: true` in settings

**Multi-line injection** using fenced `` ```! `` blocks:

````markdown
## Environment
```!
node --version
npm --version
git status --short
```
````

### `${CLAUDE_SKILL_DIR}` for Bundled Scripts

Reference scripts relative to skill location regardless of working directory:

```yaml
allowed-tools: Bash(python3 *)
---
Run: python3 ${CLAUDE_SKILL_DIR}/scripts/analyze.py .
```

## Invocation Control

### `skillOverrides` Setting

Control skill visibility from settings without editing `SKILL.md`. The `/skills` menu writes this for you (Space to cycle states, Enter to save to `.claude/settings.local.json`).

```json
{
  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  }
}
```

| Value                   | Listed to Claude     | In `/` menu |
|:------------------------|:---------------------|:------------|
| `"on"` (default)        | Name and description | Yes         |
| `"name-only"`           | Name only            | Yes         |
| `"user-invocable-only"` | Hidden               | Yes         |
| `"off"`                 | Hidden               | Hidden      |

Plugin skills are not affected by `skillOverrides`.

## Add Supporting Files

Keep `SKILL.md` under 500 lines. Move detailed reference material, examples, and large docs to separate files in the skill directory, then link them from `SKILL.md` using **markdown links** so Claude knows what each contains and loads it on demand:

```markdown
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

Use markdown links (`[file.md](file.md)`), not `@file` imports — the `@` syntax is a CLAUDE.md/memory feature, not part of the skills standard. Markdown-linked files load on demand when relevant; they are not injected up front, which keeps context lean.

Bundled scripts are executed, not loaded — reference them with `${CLAUDE_SKILL_DIR}` (see above).

## Common Skill Patterns

### Reference Skills (Background Knowledge)

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

### Task Skills (Explicit Actions)

```yaml
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
4. Verify deployment succeeded
```

### Named Argument Skills

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
arguments: [component, from, to]
---

Migrate the $component component from $from to $2.
Preserve all existing behavior and tests.
```

Usage: `/migrate-component SearchBar React Vue`

### Dynamic Context Skills

```yaml
---
name: analyze-pr
description: Analyze pull request changes
allowed-tools: Bash(gh *)
---

## Current PR State
- Diff: !`gh pr diff`
- Status checks: !`gh pr checks`

Analyze the above and provide recommendations.
```

### Research Skills (Subagent)

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

### Path-Scoped Skills

```yaml
---
name: frontend-conventions
description: Frontend coding conventions
paths: packages/frontend/**, src/components/**
---

Follow these frontend conventions:
- Use named exports
- Co-locate test files
```

Only auto-activates when working with files matching the paths.

## Extended Thinking

Include `ultrathink` anywhere in skill content to request deeper reasoning when the skill runs.

## Troubleshooting

### Skill not triggering

1. Check description includes keywords users say naturally
2. Verify skill appears in `/skills` or "What skills are available?"
3. Try invoking directly with `/skill-name`
4. Make description more specific if triggering too often

### Skill descriptions cut short

Budget scales at 1% of model context window. When overflow occurs, least-used skill descriptions are dropped first.

Remedies:
- Raise budget: `skillListingBudgetFraction: 0.02` in settings (or `SLASH_COMMAND_TOOL_CHAR_BUDGET` env var for fixed char count)
- Set low-priority skills to `"name-only"` in `skillOverrides`
- Trim `description`/`when_to_use` — combined cap is 1,536 chars per skill; put key use case first
- Configurable cap: `maxSkillDescriptionChars` in settings

### Skill stops influencing behavior after compaction

Re-invoke the skill. After auto-compaction, older skill invocations may be dropped if the 25K combined budget is exceeded.
