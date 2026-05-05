# WIKI.md — Schema Template

Use this template when running `init`. Fill in repo-specific details.

---

```markdown
# WIKI.md — [Repo Name] Wiki

## Codebase

- **Repo**: [path or URL]
- **Stack**: [language, framework, version]
- **Package manager**: [composer / npm / pip / etc.]
- **Last ingested**: [YYYY-MM-DD]

## Directory Layout

```
wiki/
  index.md          ← page catalog
  log.md            ← operation log (append-only)
  overview.md       ← stack, architecture, module map
  entities/         ← one page per domain entity/model
  flows/            ← one page per key operation traced end-to-end
  integrations.md   ← external dependencies and third-party APIs
  queries/          ← filed answers to one-off questions
```

## Entity Types

[Fill in after ingest — list domain entities discovered:]
- [Entity]: [what it represents]

## Key Entry Points

[Fill in after ingest:]
- Routes file: [path]
- Main controllers: [paths]
- Auth: [how authentication works]

## Per-Ingest Workflow

When re-ingesting after significant code changes:
1. Re-run Steps 1–6 of ingest workflow
2. Update changed entity pages
3. Update flows that changed
4. Update overview.md with any architectural changes
5. Append to log.md

## Focus Areas

[Note any areas to investigate more deeply on next pass:]
-

## Notes

[Domain-specific observations, quirks, known tech debt worth tracking]
```
