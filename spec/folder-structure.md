# `.acp/` Folder Structure

The `.acp/` directory lives at the root of a project and is the **single source of truth** for the agent workspace. It contains only plain text files — no executable code, no binaries.

## Canonical Layout

```
.acp/
├── WORKSPACE.md        Project overview, goal, current focus
├── memory.yaml         Long-term memory: stack, style, constraints, key decisions
├── tasks/              One task file per unit of work
│   └── example-task.yaml
├── sessions/           Per-conversation summaries (Markdown)
│   └── .gitkeep
├── snapshots/          Cross-session checkpoints (Markdown)
│   └── .gitkeep
└── prompts/            (Phase 3, optional) Versioned prompt library
```

## File Roles

| Path                | Role                                                                  | Read on entry? | Written by                 |
|---------------------|-----------------------------------------------------------------------|----------------|----------------------------|
| `WORKSPACE.md`      | Project goal, tech stack, current focus, important links.             | yes            | user / agent               |
| `memory.yaml`       | Long-term conventions — style, stack, constraints, decisions.         | yes            | user / agent               |
| `tasks/*.yaml`      | Discrete units of work with explicit lifecycle status.                | yes (filter to `doing`) | user / agent      |
| `sessions/*.md`     | Per-conversation summaries; light, append-only.                       | yes (latest for current task) | user / agent |
| `snapshots/*.md`    | Cross-session conclusions, durable state, next-entry-point.           | yes (per task `context_refs`)  | agent              |
| `prompts/*.md`      | (Phase 3) Versioned prompt fragments for repeatable workflows.        | no             | user                       |

## Design Constraints

- **Human-readable.** Every file must be openable in `cat`, `less`, or any text editor without decoding.
- **Parsable.** YAML files must round-trip through standard YAML parsers. Markdown files must render in any CommonMark viewer.
- **No executables.** No scripts, no compiled artifacts, no machine state.
- **Single directory.** All ACP state lives under `.acp/`. Config files elsewhere in the repo are out of scope.
- **Grep-friendly.** Names use kebab-case; field names are stable. This is a deliberate choice so agents can `grep` for skills without parsing structured indexes.

## Versioning

The folder structure itself has a version, declared in [`SKILL.md`](../SKILL.md) header. A ACP-SKILL v0.1 reader **must** tolerate the absence of the Phase 3 `prompts/` directory without warning.

Backwards compatibility rules:

- Missing `memory.yaml` → fall back to defaults (no constraints, no language).
- Missing `WORKSPACE.md` → treat the project name as the directory name; goal is unknown.
- Empty `tasks/` → report "no active task" and wait for input.
