# Context Assembly

*(Pending — Phase 2)*

## Purpose

Phase 1's READ flow is **explicit**: every step the agent must take is enumerated in `spec/READ.md`. That works for humans and for tooling that can write code. It is brittle for agents that operate at a higher level — e.g. one model that simply "opens" a workspace and asks, "what should I read?"

Phase 2 introduces a **`context-assembly` declaration** per project. The project author writes a small ACP-internal manifest that answers:

> Given a task of type T, which files under `.acp/` should be loaded into the model context, and in what order?

## Outline (proposed)

```yaml
# .acp/context-assembly.yaml
defaults:
  - WORKSPACE.md
  - memory.yaml

tasks:
  bug-fix:
    - WORKSPACE.md
    - memory.yaml
    - tasks/${task-id}.yaml
    - snapshots/${task-id}-*.md          # latest by filename date
    - sessions/${task-id}-${today}-*.md  # today's sessions

  feature:
    - WORKSPACE.md
    - memory.md
    - tasks/${task-id}.yaml
    - context_refs (from task.yaml)
```

The agent upon entering a workspace may **either** run the explicit READ flow **or** consult `context-assembly.yaml` to derive the same set of files. The two paths must produce the same end state.

## Open Questions

- Should the assembly manifest be a separate file or a top-level section in `memory.yaml`?
- Caching: how often should assembly re-run?
- Token budgeting: assembly can't return more than ~25% of the model's context window — is the agent or the manifest responsible for trimming?

These questions are deferred until Phase 2 design.
