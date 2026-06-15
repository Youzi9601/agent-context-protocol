# ACP Overview

## Problem Statement: The Agent Cold Start

When an AI agent — Claude, GPT, Hermes, or any other — opens a project, it has **no memory of prior work**. The user either re-explains the entire context from scratch, or the agent wanders directionless until it produces something coherent enough to react to.

This is the **Agent Cold Start Problem**. It is the dominant tax on long-running, multi-agent, multi-session work.

ACP (Agent Context Protocol) is a convention spec that solves it by establishing **a single, predictable layout** for the artifacts an agent needs to resume work: goal, current task, latest session, durable conclusions.

## Design Principles

| Principle            | Meaning                                                                 |
|----------------------|-------------------------------------------------------------------------|
| **LLM Native**       | The protocol's primary reader is a language model, not a library. Files are written for the model's strengths. |
| **Provider Agnostic**| No assumptions about the model's vendor, tool API, or pricing tier. ACP works identically in Claude, GPT, Hermes, and beyond. |
| **File First**       | The workspace state lives in files under version control, not in a database, not in a vendor cloud. |
| **Human Readable**   | Every artifact must be openable in any text editor and understandable without tooling. |

## Relationship to Other Tools

ACP is opinionated about one narrow thing: **how agents find prior context**. It deliberately does not reinvent adjacent layers.

| Layer                        | Owned by                    | ACP's stance                                      |
|------------------------------|-----------------------------|---------------------------------------------------|
| Tool calls, function calling | MCP (Model Context Protocol)| Borrow — agents use MCP for tools, ACP for memory. |
| Code execution, shells       | User / sandbox              | Out of scope.                                     |
| Persistent knowledge graphs  | Mem0, Letta, Zep, etc.      | Compatible. ACP uses Markdown/YAML; graph layers can ingest it. |
| Vector / RAG                 | Vendor-specific             | Compatible. Snapshots and sessions are text — embeddable as-is. |
| Project management UI        | Linear, Jira, GitHub Issues | Compatible. Tasks may mirror tickets if desired.  |

ACP **borrows MCP's naming and philosophy** but solves a different layer: MCP gives an agent hands (tools), ACP gives an agent memory (workspace state).

## Versioning

`ACP-SKILL` versions follow `vMAJOR.MINOR`. Backwards compatibility rules:

- **Major bump** → file layout or required frontmatter fields change. Older agents must refuse on detection.
- **Minor bump** → additive only (new optional fields, new sections). Older agents still work.

Current version: see [`SKILL.md`](../SKILL.md) header.

## Roadmap

### Phase 1 (v0.1) — Workspace Conventions
- `.acp/` folder layout
- `task.yaml`, `snapshot.md`, `session.md` formats
- READ flow on entry
- `SKILL.md` install file for individual agents

### Phase 2 (v0.2) — Shared Knowledge
- `memory.md`: shared knowledge base spec (long-form Markdown supplement to `memory.yaml`)
- `context-assembly.md`: automatic context assembly rules (which snapshots to bundle for which task types)
- Cross-workspace `.acp/` linking for monorepos

(Phase 2 specs are scaffolded in `spec/memory.md` and `spec/context-assembly.md`; contents are pending.)

### Phase 3 (v0.3) — Prompt Library (Optional)
- `prompts/` directory for versioned, reusable prompt fragments
- Migration helpers from common stores (Notion, Linear exports)
- Conformance test suite — a small repo that contains a reference implementation of the READ flow

---

ACP is small on purpose. Its value is not in the number of files it prescribes, but in the predictability it gives every agent that walks into a `.acp/` directory.
