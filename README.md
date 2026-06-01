# Claude Code Masters

A Claude Code marketplace shipping the **claude-code-masters** plugin: five skills for building and managing Claude Code skills, agents, plugins, and LLM wikis. Opinionated, based on official docs.

## Skills

| Skill           | Purpose                                       |
| --------------- | --------------------------------------------- |
| `skill-master`  | Create/edit Claude Code skills                |
| `agent-master`  | Create/edit Claude Code subagents             |
| `plugin-master` | Create/edit plugins; package & distribute     |
| `llm-wiki`      | Reference knowledge on the LLM wiki pattern   |
| `llm-wiki-ops`  | Run LLM wiki ops (init, ingest) on a codebase |

## Install

Add the marketplace, then install the plugin:

```
/plugin marketplace add ygavrilov/claude-code-skill-master-plugin
/plugin install claude-code-masters@ygavrilov-claude-code-masters
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

Or describe the task in natural language — Claude invokes the matching skill automatically.

## Structure

```
.claude-plugin/
  plugin.json          # Plugin metadata
  marketplace.json     # Marketplace listing
skills/
  skill-master/        # SKILL.md + supporting docs (one per skill)
  agent-master/
  plugin-master/
  llm-wiki/
  llm-wiki-ops/
```

## Author

Yevgeniy Gavrilov
