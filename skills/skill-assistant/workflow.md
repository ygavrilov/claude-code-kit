# Skill Master — Workflow

## Process

### 1. Understand Requirements

Ask the user:

- **What should this skill do?**
- **Workflow or rules skill?** (Execute a process step-by-step, or specify how to perform a task / produce a class of artifact? See Step 3.)
- **When should it be invoked?** (Automatically by Claude, manually by user, or both?)
- **Should it run in isolation?** (Does it need `context: fork`?)
- **What tools does it need?**
- **Does it need arguments?**

### 2. Determine Frontmatter

**Required:**

- `name`: lowercase-with-hyphens, max 64 chars
- `description`: keyword-rich, drives automatic invocation (combined with `when_to_use`, capped at 1,536 chars)

**Invocation Control:**

- `disable-model-invocation: true` — user-triggered only (deploy, commit, side effects)
- `user-invocable: false` — Claude-only background knowledge

**Execution Context:**

- `context: fork` — isolated subagent
- `agent: Explore|Plan|general-purpose` — which subagent (requires `context: fork`)
- `effort: low|medium|high|xhigh|max` — override effort for this skill's turn

**Tools:**

- `allowed-tools` — e.g. `Read, Grep, Glob, Bash(git *)`
- `disallowed-tools` — block tools while skill active (clears on next user message)

**UX:**

- `argument-hint` — e.g. `[issue-number]`, `[filename] [format]`
- `arguments` — named args: `[issue, branch]` → use `$issue`, `$branch` in content
- `when_to_use` — extra trigger phrases appended to description
- `paths` — glob patterns limiting when skill auto-activates

### 3. Write Skill Content

Two practical patterns to classify the skill (decide this first — it drives frontmatter and structure):

**Workflow skill** — step-by-step instructions to _execute a process_. Numbered actionable steps, each verifiable. Often `context: fork` + `disable-model-invocation: true` (side effects, deploys). Examples: deploy, fix-issue, research-pattern.

**Rules skill** — a _specification_ for how to perform a task or produce a class of artifact. Declarative guidelines, conventions, constraints — no procedure to run. Inline, no `context: fork`. Examples: coding-standards, api-endpoint-spec.

> These two terms are a navigation lens. Official docs call the same things **Task skills** and **Reference skills** respectively — see [claude-code-skills-docs.md](claude-code-skills-docs.md) for the authoritative taxonomy.

**Dynamic mechanics** (cross-cut both patterns, not a type) — `$ARGUMENTS`, `$0`/`$1` for input; `` !`pwd` `` for shell injection; `[filename](filename)` markdown links for supporting files.

### 4. Choose Location

- **Personal** `~/.claude/skills/<name>/SKILL.md` — all projects
- **Project** `.claude/skills/<name>/SKILL.md` — current project only

Default to personal unless user specifies otherwise.

### 5. Create Supporting Files

Move content to dedicated files (see principles.md for structure rules):

- Workflow/process → `workflow.md`
- Examples/patterns → `examples.md`
- Reference docs → `reference.md` or domain-specific named files
- Scripts → `scripts/`

### 6. Validate

- [ ] Skill classified — workflow (process steps) or rules (spec);
- [ ] `name` is lowercase-with-hyphens
- [ ] `description` is clear and keyword-rich; `description` + `when_to_use` under 1,536 chars
- [ ] `allowed-tools` correct format
- [ ] `context: fork` has actionable content (not just guidelines)
- [ ] Arguments use `$ARGUMENTS`, `$N`, or named `$arg` (with `arguments:` frontmatter)
- [ ] `${CLAUDE_SKILL_DIR}` used for any bundled script paths
- [ ] Supporting files referenced with markdown links `[filename](filename)`

---

## Tips

1. Keyword-rich descriptions — use terms users say naturally
2. `context: fork` for isolation — research, analysis, generation
3. Minimal tools — only what the skill needs
4. Test both `/skill-name` and natural language invocation
5. `` !`pwd` `` sparingly — only for truly dynamic content
6. Argument hints — help users know what to pass
7. Clear names — the name becomes the `/command`

---

## Editing Existing Skills

1. Read the existing SKILL.md
2. Understand current structure and frontmatter
3. Ask what to change
4. Make targeted edits with Edit tool
5. Explain what changed and why

---

See [examples.md](examples.md) for sample skill definitions.
