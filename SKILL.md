---
name: agent-context-protocol
description: "Use for project continuity via `.acp/`: resume, task ID, agent switch, snapshot/session, edits under `.acp/`, or explicit ACP init/use. Skip: general Q&A, one-offs, opt-out."
license: MIT
compatibility: Requires `.acp/` workspace structure. Works with any LLM. No external packages required.
metadata:
  author: "Youzi9601 (柚子Youzi)"
  version: "0.2"
  triggers: resume, task-id, agent-switch, snapshot, session, explicit-acp
  spec_version: "0.2"
  supports_context_refs: true
  supports_memory_yaml: true
  supports_task_yaml: true
  supports_snapshot_md: true
  supports_session_md: true
---

# Agent Context Protocol -- SKILL

## When to Activate

ACTIVATE when ANY: resume context, task ID, agent switch, `.acp/` access, snapshot/session request, or "use ACP". Full: `spec/triggers.md`.

SKIP: "skip ACP"/"no ACP", unrelated Q&A, one-offs, opt-out.

When unsure, ask.

## READ Flow (run all 5 before any work)

1. Read `.acp/WORKSPACE.md`.
2. Read `.acp/memory.yaml`.
3. Read `.acp/tasks/` with `status: doing`.
4. Find the active task's latest session.
5. Load every `context_refs` snapshot. Then verify via `spec/verification.md`.

## Report

Use the `## Workspace Report` block from [`spec/READ.md`](spec/READ.md). Append `## Verification` (see [`spec/verification.md`](spec/verification.md)) only when a check failed.

No `.acp/`? Scaffold from `templates/.acp/` (read starter, write to project root). User can `cp -r` instead.

## Formats (terse)

- `task.yaml`: `id`, `title`, `status`, `assigned_agent`, `created`, `updated`; opt `dependencies`, `context_refs` (vs .acp/), `files` (vs root), `notes`.
- `snapshot.md`: FM `task`, `date`, `agent`. `{task-id}-{YYYY-MM-DD}.md`.
- `session.md`: FM `task`, ts+offset, `agent`. `{task-id}-{YYYY-MM-DD}-{seq}.md`.
- `memory.md`: on-demand reasoning layer.
- `context-assembly.yaml`: task-type → context files.
- `prompts/`: versioned prompt fragments; see `spec/prompts.md`.

Snapshot when context fragments, task done, or decision made. Session at conversation end. Both append-only.

## More

Triggers `spec/triggers.md` | Verify `spec/verification.md` | Gotchas `spec/gotchas.md` | Tests `spec/trigger-tests.md` | Rationale `spec/rationale.md` | Memory `spec/memory.md` | Context `spec/context-assembly.md` | Prompts `spec/prompts.md`.
