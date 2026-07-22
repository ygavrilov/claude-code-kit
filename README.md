# Claude Code Kit

A Claude Code marketplace shipping the **claude-code-kit** plugin: five skills for building and managing Claude Code skills, agents, plugins, and LLM wikis. Opinionated, based on official docs.

## Skills

| Skill           | Purpose                                                  |
| --------------- | --------------------------------------------------------- |
| `skill-master`  | Create/edit Claude Code skills                            |
| `agent-master`  | Create/edit Claude Code subagents                         |
| `plugin-master` | Create/edit plugins; package & distribute                 |
| `llm-wiki`      | Design, set up, and maintain an LLM wiki knowledge base    |
| `llm-wiki-ops`  | Run LLM wiki ops (init, ingest) on a codebase              |

## Install

Add the marketplace, then install the plugin:

```
/plugin marketplace add ygavrilov/claude-code-kit
/plugin install claude-code-kit@ygavrilov-claude-code-kit
```

## Usage

Invoke a skill directly:

```
/skill-master
/agent-master
/plugin-master
/llm-wiki
/llm-wiki-ops
```

If another installed plugin has a same-named skill, qualify it: `/claude-code-kit:skill-master`.

Or describe the task in natural language — Claude invokes the matching skill automatically.

## Structure

```
.claude-plugin/
  plugin.json                 # Plugin metadata (name, version, author)
  marketplace.json            # Marketplace listing
skills/
  skill-master/                # SKILL.md + supporting docs (one per skill, same layout)
  agent-master/
  plugin-master/
  llm-wiki/
  llm-wiki-ops/
docs-upstream/                # Snapshots of official Claude Code docs, diffed nightly for drift
scripts/
  check-docs-freshness.sh     # Nightly job: diffs docs-upstream/ against live docs, opens a task on drift
  com.claude-code-kit.docs-freshness.plist   # launchd schedule for the freshness check (macOS)
.githooks/
  pre-commit                  # Blocks commits touching skills/ without a plugin.json version bump
CLAUDE.md                     # Repo guidance for Claude Code when working in this repo
```

## Maintaining this repo

These are dev-only, not needed to use the plugin:

1. **Version bump on skill changes.** `.githooks/pre-commit` blocks any commit touching `skills/` unless `.claude-plugin/plugin.json`'s `version` is bumped. Not auto-installed — after cloning: `git config core.hooksPath .githooks`.
2. **Docs drift check.** `scripts/check-docs-freshness.sh` diffs `docs-upstream/` snapshots (sub-agents, skills, plugins, plugins-reference) against the live Claude Code docs. On drift, it opens a task in the linked personal-os `tasks/open/` — this script is wired to this author's machine/workflow, not portable as-is.

## Author

Yevgeniy Gavrilov
