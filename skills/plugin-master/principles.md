# Plugin Structure Principles

## Manifest Is the Identity

`.claude-plugin/plugin.json` defines the plugin: `name`, `description`, `version`, `author`. The `name` is the skill namespace — every skill becomes `/<name>:<skill>`. Nothing else belongs inside `.claude-plugin/`.

## Components Live at the Plugin Root

The single most common mistake: putting `skills/`, `agents/`, `hooks/`, `commands/` inside `.claude-plugin/`. They must sit at the **plugin root**, beside `.claude-plugin/`. Only `plugin.json` goes inside `.claude-plugin/`.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json     ← manifest only
├── skills/             ← root, NOT in .claude-plugin/
├── agents/
└── hooks/
```

## One Plugin, Coherent Purpose

A plugin bundles related functionality shipped and versioned together. If components serve unrelated purposes, split into separate plugins. Namespacing means all skills share the plugin name — keep that name meaningful.

## Defer Component Authoring

A plugin is a package. The skills, agents, and hooks inside it are authored by their own rules — use [skill-master](../skill-master/SKILL.md) and [agent-master](../agent-master/SKILL.md). Plugin-master handles the manifest, directory layout, packaging, and distribution, not the internals of each component.

## Minimal Components

Include only the directories the plugin actually uses. An empty `agents/` or `hooks/` adds noise. Each component directory is optional — `skills/` alone is a valid plugin.

## Version Deliberately

Set `version` (semver) so users receive updates only when you bump it. Omit it and a git-distributed plugin treats every commit as a new version (commit SHA). Bump `version` on each meaningful release.

## Plugins Are Standalone First

Prefer `.claude/` standalone config for personal/project work and quick iteration. Convert to a plugin when sharing across teams or projects, versioning, or distributing via a marketplace. Short names (`/deploy`) vs namespaced (`/my-plugin:deploy`) is the trade-off.
