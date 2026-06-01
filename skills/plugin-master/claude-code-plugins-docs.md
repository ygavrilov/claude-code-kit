# Claude Code Plugins — Reference Documentation

Authoritative reference for creating Claude Code plugins.

## Standalone vs Plugin

| Approach | Skill names | Best for |
|----------|-------------|----------|
| **Standalone** (`.claude/`) | `/hello` | Personal workflows, project-specific config, quick experiments |
| **Plugin** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | Sharing, multiple projects, versioned releases, marketplace |

Start standalone for iteration; convert to a plugin to share.

---

## Plugin Manifest

`.claude-plugin/plugin.json` — the only file inside `.claude-plugin/`:

```json
{
  "name": "my-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

| Field | Purpose |
|-------|---------|
| `name` | Unique identifier + skill namespace. Skills become `/<name>:<skill>`. |
| `description` | Shown in plugin manager when browsing/installing. |
| `version` | Optional. Set it → users update only on bump. Omit + git-distributed → commit SHA, every commit = new version. |
| `author` | Optional. Attribution. |

Additional fields (`homepage`, `repository`, `license`) exist — see full manifest schema in upstream docs.

---

## Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json     ← manifest ONLY
├── skills/             ← all other dirs at ROOT
├── agents/
├── hooks/
├── .mcp.json
├── .lsp.json
├── monitors/
├── bin/
└── settings.json
```

**Common mistake:** never put `skills/`, `agents/`, `hooks/`, `commands/` inside `.claude-plugin/`. Only `plugin.json` goes there; everything else at plugin root.

| Directory / file | Location | Purpose |
|------------------|----------|---------|
| `.claude-plugin/plugin.json` | root | Manifest (optional if components use default locations) |
| `skills/` | root | Skills as `<name>/SKILL.md` directories |
| `commands/` | root | Skills as flat `.md` files (legacy — use `skills/` for new plugins) |
| `agents/` | root | Custom agent definitions (`<name>.md`) |
| `hooks/hooks.json` | root | Event handlers |
| `.mcp.json` | root | MCP server configs |
| `.lsp.json` | root | LSP server configs (code intelligence) |
| `monitors/monitors.json` | root | Background monitors |
| `bin/` | root | Executables added to Bash `PATH` while plugin enabled |
| `settings.json` | root | Default settings applied when plugin enabled |

All component dirs are optional. `skills/` alone is a valid plugin.

---

## Components

### Skills

`skills/<name>/SKILL.md`. Folder name → skill name, namespaced (`code-review/` in `my-plugin` → `/my-plugin:code-review`). Authored per skill rules — see skill-master. Run `/reload-plugins` after adding.

```
my-plugin/
└── skills/
    └── code-review/
        └── SKILL.md
```

### Agents

`agents/<name>.md`. Plugin subagents IGNORE `hooks`, `mcpServers`, `permissionMode` (security). Subfolders become part of the scoped identifier: `agents/review/security.md` → `my-plugin:review:security`. Authored per agent rules — see agent-master.

### Hooks

`hooks/hooks.json` — same format as the `hooks` object in `settings.json`. Hook command receives input as JSON on stdin (use `jq`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }]
      }
    ]
  }
}
```

### MCP Servers

`.mcp.json` at plugin root. Standard MCP server config schema.

### LSP Servers

`.lsp.json` — real-time code intelligence. Users must have the language server binary installed. Prefer official LSP plugins for common languages; create custom only for unsupported ones.

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": { ".go": "go" }
  }
}
```

### Background Monitors

`monitors/monitors.json` — watch logs/files/status, notify Claude as events arrive. Started automatically when plugin active. Each stdout line from `command` → a notification.

```json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

### Default Settings

`settings.json` at plugin root applies defaults when the plugin is enabled. Only `agent` and `subagentStatusLine` keys supported; unknown keys silently ignored. `settings.json` takes priority over `settings` in `plugin.json`.

```json
{ "agent": "security-reviewer" }
```

Setting `agent` activates one of the plugin's custom agents as the main thread (its system prompt, tools, model).

---

## Skills-Directory Plugins

`claude plugin init my-tool` scaffolds `~/.claude/skills/my-tool/` with a `.claude-plugin/plugin.json` + starter `SKILL.md`. Loads automatically next session as `my-tool@skills-dir` — no marketplace or install step. Project-scope versions require accepting the workspace-trust dialog.

---

## Testing Locally

```bash
claude --plugin-dir ./my-plugin              # load without installing
claude --plugin-dir ./my-plugin.zip          # .zip archive (v2.1.128+)
claude --plugin-url https://.../my-plugin.zip # hosted .zip, session only
claude --plugin-dir ./one --plugin-dir ./two # multiple at once
```

`/reload-plugins` picks up edits without restart (reloads skills, agents, hooks, plugin MCP + LSP servers).

A local `--plugin-dir` plugin overrides an installed marketplace plugin of the same name for the session — except plugins force-enabled/disabled by managed settings.

Verify: skills via `/plugin-name:skill-name`, agents in `/agents`, hooks fire as expected.

---

## Validate

```bash
claude plugin validate
```

Same check run by the community review pipeline. Debug order:
1. Components at plugin root, not inside `.claude-plugin/`
2. Test each component individually
3. `/reload-plugins` after edits

---

## Distribution

1. Add `README.md` (install + usage)
2. Choose versioning: explicit `version` or git SHA
3. Distribute via a [plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces); private repo keeps it internal
4. Test with others before wide release

### Marketplace Manifest

A marketplace is a repo with `.claude-plugin/marketplace.json` listing one or more plugins. Same dir as `plugin.json`.

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "rambo-marketplace",
  "description": "Survival tools forged in the field",
  "owner": {
    "name": "John Rambo",
    "email": "rambo@hopereborn.example"
  },
  "plugins": [
    {
      "name": "trautman-tools",
      "description": "Skills for extraction, recon, and after-action reports",
      "source": "./",
      "category": "productivity"
    }
  ]
}
```

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | yes | Marketplace identifier. Used in install: `plugin@marketplace-name`. |
| `owner` | **yes** | **Object**, not string. `name` + (`email` or `url`). Omitting it → `owner: Invalid input: expected object, received undefined`. |
| `plugins` | yes | Array of plugin entries. |
| `description` | no | Shown when browsing the marketplace. |
| `$schema` | no | Editor validation/autocomplete. |

**Plugin entry fields:** `name`, `description`, `source`, optional `category`. Per-plugin `version`/`author` live in each plugin's own `plugin.json` — not here.

### Plugin `source` — the install gotcha

- **Same repo as the marketplace** → `"source": "./"`. The plugin lives at the marketplace repo root.
- **Different repo** → object form:
  ```json
  "source": { "source": "github", "repo": "jrambo/trautman-tools" }
  ```

Pointing the `github` form at the marketplace's OWN repo breaks install — use `"./"` for self-hosted plugins.

### Installing from a Marketplace

```
/plugin marketplace add jrambo/rambo-plugins      # owner/repo (or https URL, or local path)
/plugin install trautman-tools@rambo-marketplace  # plugin-name@marketplace-name
```

### Community Marketplaces

- `claude-plugins-official` — curated by Anthropic, available in every install. No application process.
- `claude-community` — public submissions after review. Users add: `/plugin marketplace add anthropics/claude-plugins-community`.

Submit via in-app form (claude.ai/settings/plugins/submit or platform.claude.com/plugins/submit). Run `claude plugin validate` first. Approved plugins pinned to a commit SHA; catalog syncs nightly.

---

## Convert Standalone → Plugin

| Standalone (`.claude/`) | Plugin |
|-------------------------|--------|
| One project only | Shareable via marketplaces |
| `.claude/commands/` | `plugin-name/commands/` |
| Hooks in `settings.json` | `hooks/hooks.json` |
| Manual copy to share | `/plugin install` |

Steps: create manifest → copy `commands`/`agents`/`skills` to plugin root → move `hooks` object into `hooks/hooks.json` → test with `--plugin-dir` → remove `.claude/` originals (plugin takes precedence when loaded).
