# Skill Examples

Grouped by the two practical patterns (see [workflow.md](workflow.md) Step 3). Official taxonomy in [claude-code-skills-docs.md](claude-code-skills-docs.md).

## Workflow Skills (execute a process)

### Manual-Only Task

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
context: fork
allowed-tools: Bash(git *), Bash(npm *), Bash(ssh *)
---

Deploy to production:
1. Run test suite
2. Build application
3. Push to deployment target
```

### Argument-Based

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
argument-hint: [issue-number]
allowed-tools: Bash(gh *), Read, Write, Edit
---

Fix GitHub issue #$ARGUMENTS:

1. Fetch issue details: `gh issue view $ARGUMENTS`
2. Understand requirements
3. Implement fix
4. Write tests
5. Create commit
```

### Research Subagent

```yaml
---
name: research-pattern
description: Research a code pattern across the codebase
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze implementations
3. Identify common patterns and variations
4. Summarize findings with file references
```

## Rules Skills (spec for a task / class of artifact)

### Automatic Convention Reference

```yaml
---
name: coding-standards
description: Coding standards and conventions for this project
allowed-tools: Read
---
Follow these conventions:
    - Use TypeScript strict mode
    - Write tests for all public APIs
    - Document complex logic
```

### Spec for a Class of Artifact

```yaml
---
name: api-endpoint-spec
description: Specification every REST API endpoint in this project must follow. Use when adding or editing an endpoint, route handler, or controller.
allowed-tools: Read
---
Every API endpoint must conform to this spec:

**Naming & routing**
- Plural nouns, kebab-case paths: `/order-items`, not `/getOrderItem`
- HTTP verb carries the action — no verbs in the path

**Validation**
- Validate the request body/params at the boundary before any logic
- Reject unknown fields; never trust client input

**Responses**
- Success: `2xx` with the resource as JSON
- Errors: consistent envelope `{ error: { code, message } }`, correct status (`400` client, `404` missing, `409` conflict, `500` server)

**Errors**
- No leaking internals or stack traces in responses
- Log the full error server-side with a request id
```
