# Plugin Master — Workflow

## Process

### 1. Understand Requirements

Ask the user:

- **What does this plugin do?** (Coherent purpose — bundle related functionality)
- **Which components?** (skills / agents / hooks / MCP servers / LSP servers / monitors / default settings)
- **New plugin, or converting existing `.claude/` config?**
- **Distribution?** (Local only, team marketplace, community submission)
- **Versioning?** (Explicit semver, or git commit SHA)

### 2. Create Manifest

Plugin root holds `.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "description": "Shown in the plugin manager",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

- `name` — unique, lowercase-with-hyphens; becomes skill namespace `/my-plugin:<skill>`
- `description` — what users see when browsing
- `version` — optional but recommended (semver); omit → git SHA per commit
- `author` — optional, for attribution

### 3. Add Components (all at plugin root)

| Directory / file | Holds |
|------------------|-------|
| `skills/<name>/SKILL.md` | Skills (model- or user-invoked) |
| `agents/<name>.md` | Subagent definitions |
| `hooks/hooks.json` | Event handlers |
| `.mcp.json` | MCP server configs |
| `.lsp.json` | LSP server configs |
| `monitors/monitors.json` | Background monitors |
| `bin/` | Executables added to Bash `PATH` while enabled |
| `settings.json` | Default settings (`agent`, `subagentStatusLine` only) |

Author the components themselves with [skill-master](../skill-master/SKILL.md) (skills) and [agent-master](../agent-master/SKILL.md) (agents). Defer to [claude-code-plugins-docs.md](claude-code-plugins-docs.md) for full schemas.

### 4. Test Locally

```bash
claude --plugin-dir ./my-plugin     # load without installing
/reload-plugins                      # pick up edits without restart
/my-plugin:skill-name                # try a skill
/agents                              # confirm agents appear
```

`--plugin-dir` also accepts a `.zip`; `--plugin-url` loads a hosted `.zip`. A local `--plugin-dir` plugin overrides an installed one of the same name for the session.

### 5. Validate

```bash
claude plugin validate    # same check the community review pipeline runs
```

Debug checklist if something is missing:
1. Components at plugin root, NOT inside `.claude-plugin/`
2. Each component tested individually
3. `/reload-plugins` after edits

### 6. Share

- Add `README.md` (install + usage)
- Choose versioning strategy
- Distribute via a [marketplace](claude-code-plugins-docs.md); private repo to keep internal
- Optional: submit to `claude-community` marketplace via the in-app form

---

## Converting Standalone → Plugin

1. Create `my-plugin/.claude-plugin/plugin.json` manifest
2. Copy `.claude/commands`, `.claude/agents`, `.claude/skills` into the plugin root
3. Move `hooks` object from `.claude/settings.json` into `my-plugin/hooks/hooks.json` (same format)
4. Test with `claude --plugin-dir ./my-plugin`
5. Remove originals from `.claude/` to avoid duplicates (plugin version takes precedence when loaded)

---

## Editing Existing Plugins

1. Read `.claude-plugin/plugin.json` and scan component directories
2. Confirm components sit at root, not inside `.claude-plugin/`
3. Ask what to change
4. Make targeted edits with the Edit tool
5. Bump `version` if it's a meaningful release; remind user to run `/reload-plugins`

### Validate

- [ ] `.claude-plugin/plugin.json` valid JSON with `name` + `description`
- [ ] `name` lowercase-with-hyphens
- [ ] Components at plugin root, only `plugin.json` inside `.claude-plugin/`
- [ ] No empty component directories
- [ ] `version` set (or intentional SHA-based versioning)
- [ ] Supporting files referenced with markdown links

---

See [examples.md](examples.md) for sample plugin manifests and layouts.
