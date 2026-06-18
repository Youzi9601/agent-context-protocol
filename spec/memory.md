# Shared Knowledge Base (`memory.md`)

## Purpose

`memory.yaml` holds structured key/value conventions (stack, style, constraints) that an agent must always consider.

`memory.md` supplements `memory.yaml` with long-form, narrative knowledge that does not fit in a flat key/value format.

## Design Principle

`memory.md` is **on-demand**. It is not loaded on every READ. An agent consults it when relevant — e.g., when encountering a pattern described in a pitfall entry, or when making a decision that intersects with documented architecture.

This keeps token usage predictable. `memory.md` is not a replacement for `memory.yaml`, `snapshot.md`, or `session.md`. It is a **reasoning trace** layer: it records why decisions were made, what constraints apply, and what the project has learned.

## Must / Must Not

`memory.md` MUST contain:
- Decisions and the reasoning behind them
- Conventions beyond code style
- Recurring pitfalls and their fixes
- Stable project-level knowledge

`memory.md` MUST NOT become:
- A general knowledge dumping ground
- Unstructured notes
- A replacement for snapshots (durable conclusions) or sessions (conversation summaries)

## File Location

`.acp/memory.md` at the project root.

## Structure

```markdown
# Memory: <Project Name>

## Architecture
(Long-form prose. Diagrams in ASCII or Mermaid. Explain the system structure and key design choices.)

## Conventions Beyond Code Style
(Code review culture, deployment rituals, on-call expectations, naming conventions not enforced by linters.)

## Recurring Pitfalls
(One entry per known footgun. Format: **Symptom** / **Cause** / **Fix**.)

## Tribal Knowledge
(Business-domain assumptions baked into the code. What a new contributor must know to avoid missteps.)

## Decision Index
(One entry per significant decision. Format: **Decision** / **Reason** / **Alternatives considered** / **Link to snapshot or ADR**.)
```

## Section Requirements

All five section headers are required. Sections may be empty (write `None yet` or `- none`).

## Decision Index

Each entry in Decision Index should point to the relevant `snapshot.md` file (durable conclusion) or an external ADR document. This creates a trace from the current project state back to the reasoning that produced it.

Example entry:

```markdown
## Decision Index

- **Use JWT for auth, not sessions**
  Reason: Stateless API; multiple mobile clients; no server-side session store needed.
  Alternatives considered: OAuth2 client credentials (overkill), session cookies (stateful).
  Link: `snapshots/auth-2026-06-01.md`
```

## Loading Behavior

`memory.md` is **on-demand**. In the READ flow, after loading the active task, an agent may consult `memory.md` when:

- The task type in `context-assembly.yaml` includes `memory.md` in its context list
- The agent encounters a pattern described in Recurring Pitfalls
- The agent is about to make a decision that intersects with documented Architecture or Decision Index
- The agent cannot determine next actions from snapshots alone

Do NOT load `memory.md` on every READ. It is not part of the default reconstruction path.