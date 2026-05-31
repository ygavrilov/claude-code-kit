# llm-wiki-ops — Workflows

Dispatch on first argument:
- `init` → run **Init Workflow**
- `ingest <repo-path>` → run **Ingest Workflow** with the given repo path

---

## Init Workflow

Initialize a new wiki in the current directory.

### Steps

1. **Check for existing wiki** — look for `WIKI.md`, `wiki/`, `raw/`. If found, warn user before proceeding.

2. **Create directory structure**:
   ```
   wiki/
   wiki/entities/
   wiki/flows/
   wiki/integrations/
   wiki/queries/
   ```

3. **Create `wiki/index.md`**:
   ```markdown
   # Wiki Index

   ## Overview
   <!-- link to wiki/overview.md once created -->

   ## Entities
   <!-- domain model pages -->

   ## Flows
   <!-- key operation/flow pages -->

   ## Integrations
   <!-- external dependencies pages -->

   ## Queries
   <!-- filed answers to important questions -->
   ```

4. **Create `wiki/log.md`**:
   ```markdown
   # Wiki Log

   <!-- append-only, newest entries at bottom -->
   <!-- format: ## [YYYY-MM-DD] operation | description -->
   ```

5. **Ask user about the codebase** — what repo will this wiki cover? Any known tech stack or focus areas?

6. **Create `WIKI.md`** — use the schema template from [schema-template.md](schema-template.md), fill in what's known.

7. **Append init entry to `wiki/log.md`**:
   ```
   ## [YYYY-MM-DD] init | Wiki initialized
   ```

8. **Report** what was created, prompt user to run `/llm-wiki-ops ingest <repo-path>`.

---

## Ingest Workflow

Analyze a codebase and build wiki pages from the code itself — not documentation. The source of truth is the code.

Repo path is the second argument (e.g., `ingest ../my-api`). If not provided, use current directory.

### Step 1 — Identify Technology Stack

Detect language, framework, package manager, runtime.

Look for (check in order, stop when found):
- `composer.json` → PHP, identify framework from `require` (laravel/*, cakephp/cakephp, symfony/*, etc.)
- `package.json` → Node.js, identify framework from dependencies
- `requirements.txt` / `pyproject.toml` / `setup.py` → Python
- `go.mod` → Go
- `Cargo.toml` → Rust
- `pom.xml` / `build.gradle` → Java/Kotlin
- `Gemfile` → Ruby

Read the dependency file. Extract:
- Language + version
- Framework + version
- Key libraries (auth, ORM, queue, HTTP client, testing)
- Dev dependencies worth noting

**CakePHP specifics** (if detected):
- Check `composer.json` for cakephp version
- Project root will have `src/`, `config/`, `templates/`, `tests/`
- `src/Model/Table/` = ORM tables (one per DB table)
- `src/Model/Entity/` = entity classes
- `src/Controller/` = controllers
- `config/routes.php` = route definitions
- `config/app.php` = app config

### Step 2 — Map Project Structure

Get counts and locations of key file types. Run these (adapt paths to framework):

```bash
find src/ -name "*.php" | grep -c ""             # total PHP files
find src/Controller -name "*Controller.php" | wc -l
find src/Model/Table -name "*Table.php" | wc -l
find src/Model/Entity -name "*.php" | wc -l
find tests/ -name "*Test.php" | wc -l
ls config/
```

List directories at depth 2 to understand module/feature organization:
```bash
find . -maxdepth 2 -type d | grep -v vendor | grep -v node_modules | grep -v .git
```

Note: total file counts, which directories are largest, evident module structure.

### Step 3 — Extract Domain Model

Read the actual model/entity files to understand what data the app manages.

For CakePHP:
- List all `*Table.php` files — each is a DB table
- Read 3–5 Table files: look for `$this->belongsTo`, `$this->hasMany`, `$this->belongsToMany` to map relationships
- Read corresponding Entity files for field definitions, virtual fields, hidden fields
- Check migrations directory if present for schema history

For other stacks: read ORM models, schema files, migration files.

Extract:
- Full list of entities (DB tables / models)
- Relationships between them
- Key fields per entity (not exhaustive — focus on business-significant ones)

### Step 4 — Map API / Entry Points

Find what operations the system exposes.

For CakePHP:
```bash
cat config/routes.php
grep -r "public function " src/Controller/ | grep -v "initialize\|beforeFilter\|afterFilter"
```

For REST APIs: look for route files, controller actions, middleware.

Extract:
- Route groups / prefixes (admin, api, public)
- Controller → action list
- Any API authentication patterns (JWT, session, API key)

### Step 5 — Trace Key Flows

Pick 2–3 most important operations (ask user if unclear, or infer from entity names and controller actions — e.g., the core CRUD of the main entity, the auth flow).

For each flow:
- Start at the route/entry point
- Read the controller action
- Follow calls into services, models, utilities
- Note what DB tables are touched, what external calls are made

Goal: understand the path of a real request through the system.

### Step 6 — Map External Dependencies / Integrations

```bash
grep -r "Http\|Guzzle\|curl\|fetch\|axios" src/ --include="*.php" -l
grep -r "getenv\|env(" src/ --include="*.php" | grep -v test | head -30
cat .env.example 2>/dev/null || cat .env 2>/dev/null
```

Look for: HTTP clients, third-party API calls, queue systems, cache drivers, email providers, payment processors, external auth (OAuth, SSO).

### Step 7 — Write Wiki Pages

Write all pages to `wiki/`. Use [[wiki-link]] for cross-references.

**`wiki/overview.md`** — top-level synthesis:
- Tech stack summary
- Architecture pattern (MVC, layered, etc.)
- Module/feature map with file counts
- Entry points
- Link to all other wiki pages

**`wiki/entities/<entity-name>.md`** — one per domain entity:
- What it represents
- Key fields
- Relationships (links to related entity pages)
- Which controllers/flows use it
- Notable business rules found in code

**`wiki/flows/<flow-name>.md`** — one per traced flow:
- Entry point (route + controller action)
- Step-by-step path through the code with file references
- DB tables touched
- External calls made

**`wiki/integrations.md`** — all external dependencies:
- Each integration: what it is, how it's used, where in code

**Update `wiki/index.md`** — add all created pages with links and one-line summaries.

**Append to `wiki/log.md`**:
```
## [YYYY-MM-DD] ingest | <repo-name>

Stack: <framework> <version>
Entities: <count> (see wiki/entities/)
Flows traced: <list>
Pages created: <count>
```

### Step 8 — Report to User

- What was found (stack, entity count, routes count)
- What pages were created
- Any gaps: flows not traced, integrations unclear, areas needing deeper investigation
- Suggest follow-up queries worth filing as wiki pages
