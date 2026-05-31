# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin bundling five skills.

- `plugin.json` — plugin metadata (name, version, author)
- `marketplace.json` — marketplace listing
- `README.md` — user-facing overview
- `skills/<name>/` — one directory per skill

## Skills

| Skill          | Purpose                                                  |
| -------------- | -------------------------------------------------------- |
| `skill-master`  | Create/edit Claude Code skills                          |
| `agent-master`  | Create/edit Claude Code subagents                       |
| `plugin-master` | Create/edit Claude Code plugins; package & distribute   |
| `llm-wiki`     | Reference knowledge on the LLM Wiki pattern              |
| `llm-wiki-ops` | Execute LLM wiki operations (init, ingest) on a codebase |

## Skill Directory Layout

Each `skills/<name>/` holds:

- `SKILL.md` — entrypoint: frontmatter + one-line role + markdown links to supporting files
- supporting files — one concern each (`workflow.md`, `principles.md`, `examples.md`, `*-docs.md`, etc.)

`SKILL.md` stays thin; content lives in supporting files, referenced with markdown links `[file.md](file.md)` (not `@import`). The `*-docs.md` files mirror official Claude Code documentation and are the authoritative reference for each skill — keep them in sync with upstream docs when editing.
