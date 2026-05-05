# WIKI.md — Schema Template

Use this template when running `init`. Fill in domain-specific details based on user's answers.

---

```markdown
# WIKI.md — [Domain Name] Wiki

## Domain

[What this wiki covers. Scope: what's in, what's out.]

## Directory Layout

```
raw/              ← source documents (immutable)
raw/assets/       ← locally downloaded images
wiki/
  index.md        ← page catalog (updated on every ingest)
  log.md          ← operation log (append-only)
  sources/        ← one page per ingested source
  entities/       ← [describe entity types for this domain]
  concepts/       ← [describe concept types for this domain]
  comparisons/    ← comparison and analysis pages
  queries/        ← filed answers to important questions
  overview.md     ← evolving high-level synthesis
```

## Entity Types

[List the kinds of entities this wiki tracks. Examples:]
- People: researchers, authors, practitioners
- Organizations: companies, labs, institutions
- Products: tools, models, frameworks
- [Add domain-specific types]

## Per-Source Workflow

When ingesting a new source:
1. Write summary page to `wiki/sources/<slug>.md`
2. Update index.md
3. Create/update entity pages for significant entities
4. Create/update concept pages for key ideas
5. Flag contradictions with existing pages
6. Update overview.md if synthesis changes materially
7. Append to log.md

## Style Rules

- Summary pages: ~300–500 words. Sections: Overview, Key Points, Entities, Concepts, Cross-References.
- Entity pages: factual, neutral, cite sources. Update with each new mention.
- Concept pages: definition first, then how it appears across sources.
- Cross-links: use [[wiki-link]] syntax. Link every mentioned entity/concept that has a page.
- Frontmatter on all pages:
  ```yaml
  ---
  type: source|entity|concept|comparison|query
  date: YYYY-MM-DD
  tags: []
  ---
  ```

## Frontmatter Tags

[List tag vocabulary for this domain so Dataview queries stay consistent]
- [tag-1]
- [tag-2]

## Notes

[Any domain-specific notes, exceptions, or preferences that evolved over time]
```
