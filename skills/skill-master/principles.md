# Skill Structure Principles

## SKILL.md Must Be Skinny

SKILL.md contains only:
- Frontmatter
- One-line role statement
- Markdown links to supporting files (e.g. `[workflow.md](workflow.md)`)

Everything else lives in dedicated external files referenced via markdown links (`[workflow.md](workflow.md)`). Claude reads them on demand when the link is relevant — this keeps context load minimal and content organized.

## One File Per Concern

| Content | File |
|---------|------|
| Creation workflow, steps, tips | `workflow.md` |
| Examples and patterns | `examples.md` |
| API/field reference docs | `reference.md` or domain-named files |
| Scripts | `scripts/` |

Never mix concerns. A workflow file should not contain examples. An examples file should not contain tips.

## Reference Files Are Authoritative

Supporting files (`[workflow.md](workflow.md)`, `[examples.md](examples.md)`, etc.) are loaded on demand when Claude follows the link. They must be self-contained and accurate — do not duplicate content between files.

## Minimal Tools

Grant only the tools the skill actually needs. Never add tools "just in case."

## Descriptions Drive Invocation

The `description` field is the primary signal Claude uses to decide when to invoke a skill automatically. Make it keyword-rich and specific. Include "Use when..." or "Use proactively when..." phrasing for automatic triggers.
