---
name: agent-context-protocol
description: Use when a project has `.acp/` and the user resumes work, references a task ID, switches agents, asks for snapshot/session, or edits under `.acp/`. Triggers on "this project", "the repo", "continue", "where were we". Skip general Q&A, no-`.acp/` projects, one-off file ops, creative writing, "skip ACP". MIT.
---

# Agent Context Protocol -- SKILL v0.1

## When to Activate

ACTIVATE only when ALL true:

1. Project has `.acp/` at its root.
2. User did not opt out.

DO NOT activate when: user asks general Q&A with no project; no `.acp/`; user says "skip ACP" / "no ACP"; task is a one-off file op. When unsure, ask. Full: `spec/triggers.md`.

## READ Flow (run all 5 before any work)

1. Read `.acp/WORKSPACE.md`.
2. Read `.acp/memory.yaml`.
3. Read `.acp/tasks/` with `status: doing`.
4. Find latest session for active task (highest seq today).
5. Load every `context_refs` snapshot.

Then self-check via `spec/verification.md`. Only proceed if all 5 items pass.

## Report

```
**Project:** <name>
**Active task:** <ID + title, or "none">
**Where we left off:** <1-3 bullets>
**Suggested next action:** <one sentence>
**Open questions:** <bullets or "none">
```

No `.acp/`? Tell user: `cp -r templates/.acp/ .`.

## Formats (terse)

- `task.yaml`: `id`, `title`, `status`, `assigned_agent`, `created`, `updated`; opt `dependencies`, `context_refs` (vs .acp/), `files` (vs root), `notes`. Full: `spec/task.md`.
- `snapshot.md`: durable conclusion. FM `task`, `date`, `agent`. `{task-id}-{YYYY-MM-DD}.md`. Full: `spec/snapshot.md`.
- `session.md`: convo summary. FM `task`, ISO ts+offset, `agent`. `{task-id}-{YYYY-MM-DD}-{seq}.md`. Full: `spec/session.md`.

Snapshot when context fragments, task done, or decision made. Session at conversation end. Both append-only.

## More

Triggers `spec/triggers.md` | Verify `spec/verification.md` | Gotchas `spec/gotchas.md` | Tests `spec/trigger-tests.md` | Rationale `spec/rationale.md`.

ACP-SKILL v0.1.
