---
name: llm-wiki-ops
description: Execute LLM wiki operations. Use when initializing a new wiki structure or ingesting a source document into an existing wiki. Supports init (create wiki scaffold) and ingest (process source into wiki pages).
context: fork
argument-hint: [init|ingest] [source-path]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(mkdir -p *), Bash(find *), Bash(ls *), Bash(grep *)
---

Execute LLM wiki operation: **$ARGUMENTS**

Current directory: !`pwd`

@workflows.md

@schema-template.md
