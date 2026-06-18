# `.acp/` Folder Structure

The `.acp/` directory lives at the root of a project and is the **single source of truth** for the agent workspace. It contains only plain text files — no executable code, no binaries.

## Canonical Layout

```
.acp/
├── WORKSPACE.md             Project overview, goal, current focus
├── memory.yaml              Long-term memory: stack, style, constraints, key decisions
├── memory.md                Long-form knowledge: decisions, conventions, pitfalls
├── context-assembly.yaml    Task-type → context-file mapping (optional)
├── tasks/                   One task file per unit of work (empty at init)
│   └── .gitkeep
├── sessions/               Per-conversation summaries (Markdown)
│   └── .gitkeep
├── snapshots/              Cross-session checkpoints (Markdown)
│   └── .gitkeep
└── prompts/                Versioned prompt fragments for repeatable workflows
```

## File Roles

| Path                      | Role                                                                  | Read on entry? | Written by                 |
|---------------------------|-----------------------------------------------------------------------|----------------|----------------------------|
| `WORKSPACE.md`            | Project goal, tech stack, current focus, important links.             | yes            | user / agent               |
| `memory.yaml`             | Long-term conventions — style, stack, constraints, decisions.         | yes            | user / agent               |
| `memory.md`               | Long-form reasoning: decisions, conventions, recurring pitfalls. | on-demand      | user / agent               |
| `context-assembly.yaml`  | Task-type → context-file mapping. Optional.                    | no (consulted if present) | user / agent   |
| `tasks/*.yaml`            | Discrete units of work with explicit lifecycle status.                | yes (filter to `doing`) | user / agent      |
| `sessions/*.md`           | Per-conversation summaries; light, append-only.                       | yes (latest for current task) | user / agent |
| `snapshots/*.md`          | Cross-session conclusions, durable state, next-entry-point.           | yes (per task `context_refs`)  | agent              |
| `prompts/*.md`         | Versioned prompt fragments for repeatable workflows.    | no (consulted after READ) | user                       |

A reference `example-task.yaml` lives at `examples/example-task.yaml` in the ACP repo (not under `templates/`) so it is never copied into a fresh workspace.

## Design Constraints

- **Human-readable.** Every file must be openable in `cat`, `less`, or any text editor without decoding.
- **Parsable.** YAML files must round-trip through standard YAML parsers. Markdown files must render in any CommonMark viewer.
- **No executables.** No scripts, no compiled artifacts, no machine state.
- **Single directory.** All ACP state lives under `.acp/`. Config files elsewhere in the repo are out of scope.
- **Grep-friendly.** Names use kebab-case; field names are stable. This is a deliberate choice so agents can `grep` for skills without parsing structured indexes.

## Versioning

The folder structure itself has a version, declared in [`SKILL.md`](../SKILL.md) header. An ACP-SKILL reader must tolerate the absence of the `prompts/` directory — it is additive, not required.

Backwards compatibility rules:

- Missing `memory.yaml` → fall back to defaults (no constraints, no language).
- Missing `WORKSPACE.md` → treat the project name as the directory name; goal is unknown.
- Empty `tasks/` → report "no active task" and wait for input.

## Minimum Schemas

The `templates/.acp/` directory ships with a starter shape for `WORKSPACE.md` and `memory.yaml`. The subsections below declare the **minimum** an ACP-compliant workspace must contain for each file. A workspace MAY carry additional fields or sections; this list states the floor, not the ceiling.

### `WORKSPACE.md` — minimum sections

The file MUST contain at least one `# ` heading (the project title) followed by these H2 sections, in any order:

| Section           | Purpose                                                                  | Required? | Fallback if missing |
|-------------------|--------------------------------------------------------------------------|-----------|---------------------|
| `## Goal`         | One-paragraph description of what the project solves.                    | yes       | report "Goal unknown; project name only" and ask the user |
| `## Tech Stack`   | Primary language, framework, build tooling.                              | yes       | report "Stack unknown" in `memory.yaml` block |
| `## Current Focus`| The single highest-priority task ID the workspace is converging on.      | yes       | treat as no active task (see `gotchas.md#2`) |
| `## Important Links` | URLs to repo, docs, deployment — at least one bullet.                  | no        | omit silently; not enforced in v0.1 |

Any other H2 sections are allowed. Agents MUST NOT refuse to read the file because of extra sections; they SHOULD ignore unknown sections rather than error out. The starter at `templates/.acp/WORKSPACE.md` is the canonical example.

### `memory.yaml` — minimum keys

The file MUST be a valid YAML mapping. Within the mapping, these keys are required at the levels shown:

| Key path                  | Type         | Required? | Notes                                                        |
|---------------------------|--------------|-----------|--------------------------------------------------------------|
| `project.name`            | string       | yes       | Human-readable project name. May be empty string at init.    |
| `project.language`        | string       | yes       | May be empty string at init.                                 |
| `project.framework`       | list[string] | yes       | May be empty list `[]` at init.                              |
| `project.timezone`        | string       | yes       | IANA tz, e.g. `Asia/Taipei`. Used by session/snapshot filename dates. May be empty string at init but agents MUST warn if unset and a session/snapshot is being written. |
| `coding_style`            | list[string] | yes       | May be empty list at init.                                   |
| `test_framework`          | string       | yes       | May be empty string at init.                                 |
| `constraints`             | list[string] | yes       | May be empty list at init. Hard limits (license, runtime).   |
| `key_decisions`           | list[string] | yes       | May be empty list at init. Library/pivot rationale log.     |

Empty strings and empty lists are **explicitly tolerated at init**. An agent reading an init-stage `memory.yaml` (everything empty) MUST proceed with defaults rather than refuse.

The starter at `templates/.acp/memory.yaml` is the canonical example. Projects MAY add additional top-level keys for project-specific facts (e.g. `on_call`, `deploy_target`) as long as the eight keys above remain present (possibly empty).

### `memory.yaml` — optional `related:` field

For monorepo awareness, `memory.yaml` MAY contain a top-level `related:` list. Each entry declares a sibling `.acp/` workspace and its relationship to this one.

| Key path   | Type   | Required? | Notes                                                    |
|------------|--------|-----------|----------------------------------------------------------|
| `related[].path`         | string | no        | Relative path to a sibling `.acp/` directory.           |
| `related[].relationship` | string | no        | Free-form label: `sibling`, `dependency`, etc.           |

One-hop only. No recursive traversal. See `spec/context-assembly.md` for cross-workspace behavior.

### `memory.md` — minimum sections

`memory.md` is optional at init. When present, it MUST contain these five H2 sections (sections may be empty):

| Section                     | Required? | Notes                                                    |
|-----------------------------|-----------|----------------------------------------------------------|
| `## Architecture`           | yes       | Long-form prose describing system structure.            |
| `## Conventions Beyond Code Style` | yes | Non-linter conventions, rituals, expectations.   |
| `## Recurring Pitfalls`     | yes       | Footguns: **Symptom** / **Cause** / **Fix**.            |
| `## Tribal Knowledge`       | yes       | Business-domain assumptions for new contributors.       |
| `## Decision Index`         | yes       | One entry per significant decision with rationale.      |

All five section headers are required. Agents MUST NOT refuse to read the file because of extra sections.`
