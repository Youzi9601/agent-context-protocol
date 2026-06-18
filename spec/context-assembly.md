# Context Assembly

## Purpose

The READ flow is **explicit** and **universal**: every step the agent must take is enumerated in `spec/READ.md`. That works for tooling and humans who can follow steps. It is sufficient for basic reconstruction.

However, different task types need different context. A `bug-fix` task may need recent error snapshots and session history. A `feature` task may need architecture documentation and memory.md. Loading everything on every READ is token-inefficient; loading only the default set may miss task-relevant context.

`context-assembly.yaml` is an **optional manifest** that maps task types to the files an agent should load. It allows the project author to declare: "given task type T, load these files."

## Must / Must Not

`context-assembly.yaml` MUST:
- Map task types to file patterns under `.acp/`
- Be declarative (no execution logic)
- Remain optional — explicit READ is always valid

`context-assembly.yaml` MUST NOT become:
- A retrieval engine or semantic search system
- A ranking or scoring system
- A dependency graph
- A required component

## File Location

`.acp/context-assembly.yaml` at the project root. Optional. If absent, the agent uses explicit READ.

## Format

```yaml
defaults:
  - WORKSPACE.md
  - memory.yaml

tasks:
  <task-type>:
    - <file-path or glob-pattern>
    - ...
```

### Built-in Variables

| Variable | Meaning |
|----------|---------|
| `${task-id}` | The task ID from the active task's `task.yaml` |
| `${today}` | Today's date in `YYYY-MM-DD` format |

Variables are substituted by the agent at runtime.

### Task Types

Task types are free-form strings that describe the kind of work. Common types: `bug-fix`, `feature`, `refactor`, `docs`, `review`.

A task may specify its type in `task.yaml` under `task_type:`. If absent, the agent uses the `defaults` list.

## Example

```yaml
defaults:
  - WORKSPACE.md
  - memory.yaml

tasks:
  bug-fix:
    - WORKSPACE.md
    - memory.yaml
    - tasks/${task-id}.yaml
    - snapshots/${task-id}-*.md
    - sessions/${task-id}-${today}-*.md

  feature:
    - WORKSPACE.md
    - memory.yaml
    - memory.md
    - tasks/${task-id}.yaml
    - snapshots/${task-id}-*.md
    - sessions/${task-id}-${today}-*.md
    - related:../../*/.acp  # cross-workspace awareness (optional)
```

## How It Relates to Explicit READ

The explicit READ flow (from `spec/READ.md`) is the **canonical** ACP entry path. `context-assembly.yaml` is an **optional optimization** layered on top:

- Agent B enters workspace
- Agent B runs the first 4 steps of explicit READ (WORKSPACE → memory.yaml → active task → latest session)
- Agent B then checks: does `context-assembly.yaml` exist?
  - If yes: load files according to the task type in the active task
  - If no: use explicit READ output as-is
- Agent B continues with workspace report

Both paths produce equivalent reconstruction output. The only difference is which files are loaded as context.

## Equivalence Guarantee

Any workspace report produced using `context-assembly.yaml` must answer the same three questions as explicit READ alone:
- What is happening now?
- Why is it happening?
- What should happen next?

`context-assembly.yaml` must not produce a workspace that omits information recoverable via explicit READ.

## Snapshot Ordering

When `context-assembly.yaml` specifies a glob pattern for snapshots, the agent loads all matches sorted by filename date (descending). The most recent snapshot is the one with the latest date in the filename.

The convention `{task-id}-latest.md` may be used as an explicit alias for the most recent snapshot by date — see `spec/snapshot.md`.

## Token Budget

The agent applies judgment. The manifest declares intent; the agent decides what actually fits in context. If loading the specified files exceeds available context, the agent drops files from the bottom of the list first (least relevant).