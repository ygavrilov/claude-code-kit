# llm-wiki-ops — Workflows

Dispatch on first argument:
- `init` → run **Init Workflow**
- `ingest <path>` → run **Ingest Workflow** with the given source path

---

## Init Workflow

Initialize a new wiki in the current directory.

### Steps

1. **Check for existing wiki** — look for `WIKI.md`, `wiki/`, `raw/`. If found, warn user before proceeding.

2. **Create directory structure**:
   ```
   raw/
   raw/assets/
   wiki/
   wiki/sources/
   wiki/entities/
   wiki/concepts/
   wiki/comparisons/
   wiki/queries/
   ```

3. **Create `wiki/index.md`**:
   ```markdown
   # Wiki Index

   ## Sources
   <!-- source pages listed here -->

   ## Entities
   <!-- entity pages listed here -->

   ## Concepts
   <!-- concept pages listed here -->

   ## Comparisons & Queries
   <!-- comparison/query pages listed here -->
   ```

4. **Create `wiki/log.md`**:
   ```markdown
   # Wiki Log

   <!-- append-only, newest entries at bottom -->
   <!-- format: ## [YYYY-MM-DD] operation | description -->
   ```

5. **Ask user about domain** — what is this wiki about? What kinds of entities and concepts will it track? Use answers to customize `WIKI.md`.

6. **Create `WIKI.md`** — use the schema template from `@schema-template.md`, fill in domain-specific details from user's answers.

7. **Append init entry to `wiki/log.md`**:
   ```
   ## [YYYY-MM-DD] init | Wiki initialized
   ```

8. **Report** what was created, remind user to drop sources into `raw/` and run `/llm-wiki-ops ingest <path>`.

---

## Ingest Workflow

Process a source document into the wiki. Source path is the second argument (e.g., `ingest raw/article.md`).

### Steps

1. **Read `WIKI.md`** — understand this wiki's domain, conventions, entity types, and per-source workflow.

2. **Read `wiki/index.md`** — understand existing pages and current wiki state.

3. **Read the source file** at the given path. If path not provided, ask user which file in `raw/` to process.

4. **Discuss key takeaways with user** — what are the most important points? What entities and concepts appear? What contradicts or extends existing wiki content? (Skip this step if user requests batch mode.)

5. **Write source summary page** to `wiki/sources/<slug>.md`:
   - Slug from source title/filename
   - Sections: Overview, Key Points, Entities Mentioned, Concepts, Cross-References, Raw Source link
   - Add YAML frontmatter: `type: source`, `date: YYYY-MM-DD`, `tags: [...]`

6. **Update `wiki/index.md`** — add entry under Sources with link and one-line summary.

7. **Update entity pages** — for each significant entity mentioned:
   - If page exists: update with new information from this source, note the source, flag any contradictions
   - If page doesn't exist: create `wiki/entities/<name>.md` with initial content
   - Add entry to `wiki/index.md` under Entities if new

8. **Update concept pages** — same pattern as entities, under `wiki/concepts/`

9. **Update `wiki/overview.md`** if it exists — revise synthesis to reflect new source if it materially changes the picture.

10. **Append ingest entry to `wiki/log.md`**:
    ```
    ## [YYYY-MM-DD] ingest | <source title>

    Pages created: wiki/sources/<slug>.md, wiki/entities/<X>.md
    Pages updated: wiki/entities/<Y>.md, wiki/concepts/<Z>.md
    Key finding: <one sentence>
    ```

11. **Report** — list all pages created and updated, highlight any contradictions found with existing content.
