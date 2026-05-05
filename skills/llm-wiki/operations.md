# LLM Wiki — Operations

## Ingest

Drop a source into `raw/`, tell the LLM to process it.

Flow:
1. LLM reads source
2. Discusses key takeaways with user (optional but recommended for first-time)
3. Writes summary page to `wiki/` (e.g., `wiki/sources/article-title.md`)
4. Updates `wiki/index.md` — adds entry with link, one-line summary, category
5. Updates relevant entity and concept pages across the wiki (one source may touch 10–15 pages)
6. Appends entry to `wiki/log.md`

Preference: ingest one source at a time with user involvement. Batch ingestion possible but reduces synthesis quality. Document preferred workflow in `WIKI.md`.

## Query

Ask questions against the wiki.

Flow:
1. LLM reads `wiki/index.md` to find relevant pages
2. Reads relevant pages
3. Synthesizes answer with citations to wiki pages

**Key insight**: good answers get filed back as new wiki pages. A comparison, analysis, or discovered connection is valuable — don't let it disappear into chat history. This compounds the knowledge base just like ingested sources.

Output formats depending on question: markdown page, comparison table, Marp slide deck, matplotlib chart.

## Lint

Periodic health check.

LLM checks for:
- Contradictions between pages (newer source supersedes older claim)
- Orphan pages — no inbound links
- Important concepts mentioned but lacking their own page
- Missing cross-references between related pages
- Stale claims (newer sources have updated information)
- Data gaps that could be filled with a web search
- Suggests new questions to investigate, new sources to look for

Run lint when the wiki grows or after a large batch of ingests.

## Init

Initialize a new wiki.

Creates:
```
raw/           ← drop source documents here
wiki/
  index.md     ← catalog of all wiki pages (LLM updates on every ingest)
  log.md       ← append-only chronological record
WIKI.md        ← schema for this wiki (customize for your domain)
```

`WIKI.md` is the most important step — it tells the LLM how THIS wiki is structured, what pages to create for each source, naming conventions, and what to do in each operation. Start from the template and customize for the domain.

## Index and Log

**index.md** — content-oriented catalog. Every wiki page listed with link, one-line summary, optional metadata. Organized by category. LLM reads this first when answering queries to find relevant pages. Works well at ~100 sources / ~hundreds of pages without embedding-based RAG.

**log.md** — chronological append-only record. Each entry starts with consistent prefix for grep-ability:
```
## [2026-04-02] ingest | Article Title
## [2026-04-02] query | user question summary
## [2026-04-02] lint | health check summary
```

Useful tip: `grep "^## \[" wiki/log.md | tail -5` gives last 5 log entries.
