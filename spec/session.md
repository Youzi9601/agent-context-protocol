# Session Format

A session file is a **lightweight snapshot of a single conversation**. It records *what happened* in that interaction, in human-readable prose — not the entire transcript.

## File Format

```markdown
---
task: feature-002
date: 2026-06-15T14:30:00+08:00
agent: claude
---

## Session Summary
(Focus of this conversation, in 1–3 paragraphs.)

## Outputs
- Created `src/auth/jwt.ts`
- Modified `src/api/login.ts`

## Open Questions
- Refresh token strategy is not yet decided.
```

## File Location and Naming

- Directory: `.acp/sessions/`
- Filename: `{task-id}-{YYYY-MM-DD}-{seq}.md`
- `seq` is a zero-padded counter for sessions on the same date (`00`, `01`, `02`, ...).
  - Example: `feature-002-2026-06-15-00.md`, `feature-002-2026-06-15-01.md`.
- Multiple sessions per task per day are expected and normal.

## Frontmatter

| Field   | Type            | Required | Description                                            |
|---------|-----------------|----------|--------------------------------------------------------|
| `task`  | string          | yes      | Task ID; must match an existing `tasks/*.yaml` entry.  |
| `date`  | ISO 8601 date-time | yes    | Full timestamp including timezone offset.             |
| `agent` | string          | yes      | Identifier of the agent that ran the session.          |

## Body Sections (suggested)

- **Session Summary** — narrative of what was attempted, discovered, or decided.
- **Outputs** — concrete artifacts created or modified (files, commits, PRs).
- **Open Questions** — items that surfaced during the session but were not resolved.

These headings are conventional. The body is free-form Markdown.

## When to Write a Session File

Either the AI agent or the user writes a session file when **the conversation ends** — voluntarily or otherwise. Common triggers:

1. The user signs off ("that's it for today").
2. The agent detects context fragmentation and offers to close out.
3. A hard cap (context window, rate limit, network drop) is approaching.
4. A clean milestone is reached, even mid-day.

Session files are cheap and disposable; when in doubt, write one.

## Sessions vs. Snapshots

A **session** answers: "What happened in this conversation?"
A **snapshot** answers: "What is the durable state of this task and what should I do next?"

Sessions are write-often, read-rarely (audit trail). Snapshots are write-sometimes, read-on-entry (cold-start lever). Both may exist for the same task on the same day — they serve different roles.

See [`snapshot.md`](snapshot.md) for the snapshot format.
