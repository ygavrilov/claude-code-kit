# LLM Wiki — Pattern

## Core Idea

Not RAG. RAG re-derives knowledge from raw documents at query time — no accumulation, no synthesis persisted. The LLM Wiki pattern is different: the LLM **incrementally builds and maintains a persistent wiki** sitting between raw sources and the user.

When a new source arrives, the LLM reads it, extracts key information, and integrates it into the existing wiki — updating entity pages, revising summaries, flagging contradictions, strengthening synthesis. Knowledge is compiled once and kept current.

**The wiki is a persistent, compounding artifact.** Cross-references already exist. Contradictions already flagged. Synthesis already reflects everything read. Gets richer with every source and every question.

## Three Layers

```
raw/          ← immutable source documents (LLM reads, never modifies)
wiki/         ← LLM-generated markdown files (LLM owns entirely)
WIKI.md       ← schema: structure, conventions, workflows (co-evolved by user + LLM)
```

**raw/** — curated source documents: articles, papers, transcripts, data files. Source of truth. Never modified.

**wiki/** — LLM-generated markdown: summaries, entity pages, concept pages, comparisons, overview, synthesis. LLM creates, updates, maintains cross-references, flags contradictions.

**WIKI.md** — configuration for the LLM. What the wiki covers, directory conventions, what each operation does, style rules. Analogous to CLAUDE.md for a codebase. User and LLM co-evolve this over time.

## Why This Works

The bottleneck in maintaining a knowledge base is not reading or thinking — it's bookkeeping: updating cross-references, keeping summaries current, noting when new data contradicts old, maintaining consistency across dozens of pages. Humans abandon wikis because maintenance burden grows faster than value.

LLMs don't get bored, don't forget to update a cross-reference, can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

Human's job: curate sources, direct analysis, ask good questions, think about what it means.
LLM's job: summarize, cross-reference, file, bookkeep, keep everything consistent.

## Working Style

In practice: LLM agent on one side, Obsidian on the other. LLM makes edits; user browses results in real time — following links, checking graph view, reading updated pages.

Obsidian = IDE. LLM = programmer. Wiki = codebase.

## Use Cases

- **Personal**: goals, health, self-improvement — journal entries, articles, podcast notes building a structured self-picture
- **Research**: going deep on a topic over weeks/months — papers, articles building a wiki with evolving thesis
- **Book reading**: filing each chapter, building pages for characters, themes, plot threads — like fan wikis (Tolkien Gateway) built personally as you read
- **Business/team**: internal wiki fed by Slack threads, meeting transcripts, project docs, customer calls
- **Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives**
