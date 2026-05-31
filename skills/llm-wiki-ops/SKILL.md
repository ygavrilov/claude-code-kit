---
name: llm-wiki-ops
description: Execute LLM wiki operations on a codebase. Use when initializing a new wiki structure or analyzing and ingesting a code repository into a wiki. Supports init (create wiki scaffold) and ingest (analyze codebase, extract stack/structure/entities/flows, write wiki pages).
context: fork
argument-hint: "[init|ingest] [repo-path]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(mkdir -p *), Bash(find *), Bash(ls *), Bash(grep -r *), Bash(wc *), Bash(cat *)
---

Execute LLM wiki operation: **$ARGUMENTS**

Current directory: !`pwd`

Follow the workflow for the requested operation:

- [workflows.md](workflows.md) — init and ingest workflows, step by step
- [schema-template.md](schema-template.md) — WIKI.md schema template to fill in
