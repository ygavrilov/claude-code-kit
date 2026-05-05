# LLM Wiki — Conventions

## Directory Structure

```
project-wiki/
├── raw/                    ← immutable source documents
│   └── assets/             ← locally downloaded images
├── wiki/
│   ├── index.md            ← page catalog (LLM-maintained)
│   ├── log.md              ← operation log (append-only)
│   ├── sources/            ← one page per ingested source
│   ├── entities/           ← people, organizations, products
│   ├── concepts/           ← ideas, themes, frameworks
│   ├── comparisons/        ← comparison pages (from queries)
│   └── overview.md         ← high-level synthesis (evolves over time)
└── WIKI.md                 ← schema (customize per domain)
```

Subdirectory structure is flexible — adapt to domain. Document chosen structure in `WIKI.md`.

## Page Naming

- Lowercase with hyphens: `entity-name.md`, `concept-theme.md`
- Source pages: slug from title: `wiki/sources/why-machines-learn.md`
- Entity pages: `wiki/entities/john-von-neumann.md`
- Dated queries filed as pages: `wiki/queries/2026-04-02-comparison-approaches.md`

## Cross-Linking

Use relative Obsidian-style wiki links: `[[entity-name]]` or `[[concept-theme|display text]]`.

Every entity or concept mentioned in a page should link to its own page if one exists. If no page exists yet, note it as a candidate for creation.

## index.md Format

```markdown
# Wiki Index

## Sources
- [[sources/article-title]] — one-line summary · 2026-04-01
- [[sources/paper-name]] — one-line summary · 2026-03-28

## Entities
- [[entities/person-name]] — role/relevance
- [[entities/org-name]] — what it is

## Concepts
- [[concepts/theme]] — brief definition

## Comparisons & Queries
- [[comparisons/2026-04-02-X-vs-Y]] — what was compared
```

## log.md Format

Append-only. Each entry:
```markdown
## [YYYY-MM-DD] operation | description

Brief notes on what was done, what pages were created/updated, key findings.
```

Grep pattern for last 5 entries: `grep "^## \[" wiki/log.md | tail -5`

## WIKI.md (Schema)

The schema tells the LLM how THIS wiki operates. Key sections:
- **Domain** — what this wiki covers, scope
- **Directory layout** — which subdirectories exist and what goes where
- **Per-source workflow** — what pages to create/update when ingesting a source
- **Entity types** — what kinds of entities to track
- **Style rules** — tone, length of summaries, what to emphasize
- **Frontmatter** — any YAML frontmatter to add to pages (for Dataview queries)

## Optional Frontmatter (for Dataview)

Add YAML to pages for Obsidian Dataview queries:
```yaml
---
type: source
date: 2026-04-01
source-count: 1
tags: [machine-learning, theory]
---
```

## Tooling Tips

- **Obsidian Web Clipper**: browser extension → converts articles to markdown for `raw/`
- **Download images locally**: Obsidian Settings → Files and links → set attachment folder to `raw/assets/` → hotkey for "Download attachments for current file"
- **Obsidian graph view**: best way to see wiki shape — hubs, orphans, clusters
- **Marp**: markdown slide decks — Obsidian plugin, useful for presentations from wiki content
- **Dataview**: Obsidian plugin — queries over frontmatter for dynamic tables/lists
- **qmd**: local markdown search engine (BM25/vector + LLM re-ranking), has CLI and MCP server — useful when wiki exceeds ~hundreds of pages
- The wiki is a git repo — version history, branching, collaboration for free
