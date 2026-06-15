# Snapshot Format

A snapshot is a **persistent, cross-session conclusion** about a task. It captures *what was decided* and *what comes next*, not a verbatim conversation log.

## File Format

```markdown
---
task: feature-002
date: 2026-06-15
agent: claude
---

## What Was Completed
- JWT middleware finished
- API routes created

## What Remains
- Integration tests are missing

## Next Entry Point
Fill in the integration tests in `src/auth/auth.test.ts`

## Key Decisions
Chose `jose` over `jsonwebtoken` for edge runtime compatibility.
```

## File Location and Naming

- Directory: `.acp/snapshots/`
- Filename: `{task-id}-{YYYY-MM-DD}.md`
- Multiple snapshots per task are allowed. The most recent (by date in filename) is the current source of truth.
- A new snapshot for the same task does **not** delete the prior one — the timeline is preserved.

## Frontmatter

| Field   | Type   | Required | Description                                  |
|---------|--------|----------|----------------------------------------------|
| `task`  | string | yes      | Task ID; must match an existing `tasks/*.yaml` entry. |
| `date`  | date   | yes      | ISO date `YYYY-MM-DD` of when this snapshot was written. |
| `agent` | string | yes      | Identifier of the agent that wrote the snapshot. |

## Body Sections (recommended headings)

The four sections above (`What Was Completed`, `What Remains`, `Next Entry Point`, `Key Decisions`) are conventional, not enforced. Use them or rename as long as the intent is clear:

- What landed in code/decisions during this slice.
- What is deliberately left undone.
- A single concrete action a future agent (or human) can take to resume work.
- Trade-offs, library choices, and rationale that must not be re-litigated.

## When to Write a Snapshot

An agent should write a snapshot when **any** of these triggers fires:

1. **Context exhaustion is imminent** — the agent is about to lose coherence on the task.
2. **A natural stopping point is reached** — a sub-goal is complete and verified.
3. **A non-trivial decision was made** — a library choice, an architecture pivot, a deliberately-accepted trade-off.
4. **The task transitions status** — moving from `doing` to `done` requires a snapshot.
5. **The user signals a pause or handoff** — "I'm switching to GPT", "we'll continue tomorrow".

## Snapshots vs. Sessions

|              | Snapshot                          | Session                                |
|--------------|-----------------------------------|----------------------------------------|
| Scope        | Cross-session conclusions         | Single-conversation summary            |
| Cadence      | Sparse, milestone-level           | Every conversation                     |
| Author       | Primarily the AI agent            | The AI agent or the user               |
| Stability    | Append-only, durable              | Append-only, lighter                   |
| Purpose      | Resume a task across cold starts  | Audit what happened in one interaction |

See [`session.md`](session.md) for the session format.
