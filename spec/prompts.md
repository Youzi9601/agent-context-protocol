# Prompt Library (`prompts/`)

## Purpose

`prompts/` is a **versioned library of reusable prompt fragments** for repeatable agent workflows.

A prompt fragment is a partial prompt — not a complete instruction set. It covers one workflow: the steps, expected output, and conventions for that task type. The agent composes the fragment with workspace context (WORKSPACE, task, memory) at runtime.

## Why a Prompt Library

Without a prompt library, each agent run is a fresh interpretation of the task. A `review` task run by Agent A and Agent B may produce different report formats, different section headings, and different severity scales. The `prompts/` library fixes this by declaring: "this is how we do a review."

## Design Principle

`prompts/` is **agent-consulted, not agent-required**. A missing `prompts/` directory does not block ACP operation. The explicit READ path always works. Prompts are an additive optimization for consistency and workflow repeatability.

## File Location

`.acp/prompts/` at the project root. Optional. If absent, the agent uses its own interpretation of the task type.

## Naming Convention

```
{workflow-type}.md
```

`workflow-type` is a free-form kebab-case string. Common types: `code-review`, `bug-report`, `pr-summary`, `onboarding`, `release-notes`.

## File Format

Each prompt file is Markdown with YAML frontmatter:

```yaml
---
version: "1.0"
workflow: code-review
description: Standard review template for pull requests in this project
loaded_by:
  - context-assembly (task type: review)
notes: |
  Add project-specific review criteria below the fixed sections.
---
```

### Frontmatter Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | yes | Semantic version of this prompt fragment. Bump on breaking changes. |
| `workflow` | string | yes | Identifier matching the filename without `.md`. |
| `description` | string | no | One-line description of when to use this fragment. |
| `loaded_by` | list[string] | no | Which ACP mechanisms load this file. |
| `notes` | string | no | Human notes on customization (not read by agent). |

### Body Sections

The body contains any Markdown content. Convention: start with an H1 matching the workflow type, then organize in numbered steps or conventional sections the project uses consistently.

There is **no fixed body schema**. The project defines its own conventions. The agent reads the body and follows the structure declared within.

## How It Relates to Explicit READ

`prompts/` is consulted **after** the explicit READ flow:

1. Agent runs explicit READ (WORKSPACE → memory.yaml → active task → latest session → context_refs)
2. Agent identifies task type (from `task.yaml` or inferred from context)
3. Agent checks: does `.acp/prompts/{task-type}.md` exist?
   - If yes: loads it, merges the fragment structure into workspace understanding
   - If no: proceeds with agent's own interpretation
4. Agent continues with workspace report or action

## Must / Must Not

`prompts/` files MUST:
- Be Markdown with valid frontmatter
- Describe one specific workflow
- Be human-readable without tooling

`prompts/` files MUST NOT become:
- Complete system prompts (agent adds its own reasoning layer)
- Executable code or scripts
- Required for ACP to function (optional, additive)
- Duplicates of `context-assembly.yaml` behavior (different layer: one is context-loading, one is prompt structure)

## Versioning

Each `prompts/` file is independently versioned. A breaking change to `code-review.md` bumps its `version` field. Agents should warn if the loaded version differs from the version last seen in a session, but must not refuse to use an older version.

## Example: Code Review

File: `.acp/prompts/code-review.md`

```yaml
---
version: "1.0"
workflow: code-review
description: Standard PR review template
loaded_by:
  - context-assembly (task type: review)
---

# Code Review — {pr-number}

## Scope
- Files changed: `{file-list}`
- Lines added/removed: `{delta}`

## Findings

### Critical
(none — or list each with file:line reference)

### Major
-

### Minor
-

## Recommendation
Merge / Request changes / Block

## Notes
-
```

## Conformance

An ACP agent that supports `prompts/`:
- Loads `prompts/` fragments after the explicit READ flow
- Merges the fragment structure with workspace understanding
- Does not refuse operation if `prompts/` is absent
- Does not substitute the fragment for its own reasoning

See `spec/verification.md` for self-check procedures.