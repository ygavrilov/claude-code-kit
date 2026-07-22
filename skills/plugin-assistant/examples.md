# Plugin Examples

## Minimal Skill Plugin

```
my-first-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── hello/
        └── SKILL.md
```

```json
// .claude-plugin/plugin.json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

```markdown
<!-- skills/hello/SKILL.md -->
---
description: Greet the user with a personalized message
---

Greet the user named "$ARGUMENTS" warmly and ask how you can help today.
```

Invoke: `/my-first-plugin:hello Alex`

---

## Multi-Component Plugin

```
review-tools/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── pr-summary/
│       └── SKILL.md
├── agents/
│   └── code-reviewer.md
├── hooks/
│   └── hooks.json
└── README.md
```

```json
// .claude-plugin/plugin.json
{
  "name": "review-tools",
  "description": "Code review skills, a reviewer agent, and lint-on-save hooks",
  "version": "2.1.0",
  "author": { "name": "Team Platform" }
}
```

```json
// hooks/hooks.json
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

---

## Behavior-Changing Plugin (Default Agent)

A plugin that makes Claude Code run as a security reviewer by default when enabled.

```
security-mode/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   └── security-reviewer.md
└── settings.json
```

```json
// settings.json
{ "agent": "security-reviewer" }
```

---

## LSP Plugin

```
go-lsp/
├── .claude-plugin/
│   └── plugin.json
└── .lsp.json
```

```json
// .lsp.json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": { ".go": "go" }
  }
}
```

---

## Monitor Plugin

```
log-watcher/
├── .claude-plugin/
│   └── plugin.json
└── monitors/
    └── monitors.json
```

```json
// monitors/monitors.json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

---

## Marketplace (single self-hosted plugin)

One repo that is both the plugin and its marketplace. Plugin source is `./`.

```
rambo-plugins/                  # repo: jrambo/rambo-plugins
├── .claude-plugin/
│   ├── plugin.json             # the plugin manifest
│   └── marketplace.json        # the marketplace listing
└── skills/
    └── extraction/
        └── SKILL.md
```

```json
// .claude-plugin/plugin.json
{
  "name": "trautman-tools",
  "description": "Skills for extraction, recon, and after-action reports",
  "version": "1.0.0",
  "author": { "name": "Sam Trautman" }
}
```

```json
// .claude-plugin/marketplace.json
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

Install:

```
/plugin marketplace add jrambo/rambo-plugins
/plugin install trautman-tools@rambo-marketplace
```

---

## Marketplace (plugins in separate repos)

A standalone catalog repo pointing at plugins hosted elsewhere. Each entry uses the `github` source form.

```json
// .claude-plugin/marketplace.json
{
  "name": "rambo-marketplace",
  "owner": { "name": "John Rambo", "url": "https://github.com/jrambo" },
  "plugins": [
    {
      "name": "trautman-tools",
      "description": "Extraction and recon skills",
      "source": { "source": "github", "repo": "jrambo/trautman-tools" }
    },
    {
      "name": "teasle-lint",
      "description": "Strict lint-on-save hooks",
      "source": { "source": "github", "repo": "wteasle/teasle-lint" }
    }
  ]
}
```
