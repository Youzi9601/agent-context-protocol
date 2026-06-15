# `task.yaml` Format

A task file lives in `.acp/tasks/{task-id}.yaml` and represents one unit of work.

## File Format

```yaml
id: feature-002
title: "Add OAuth login"
status: todo
assigned_agent: claude
created: 2026-06-15
updated: 2026-06-15
dependencies:
  - bug-001
context_refs:
  - snapshots/auth-design.md
files:
  - src/auth/*
notes: |
  Any additional context.
```

## Field Reference

| Field            | Type            | Required | Description                                                    |
|------------------|-----------------|----------|----------------------------------------------------------------|
| `id`             | string          | yes      | Unique task identifier (kebab-case). Also the filename stem.   |
| `title`          | string          | yes      | One-line human-readable title.                                  |
| `status`         | enum            | yes      | One of `todo`, `doing`, `done`. See [Status Lifecycle](#status-lifecycle). |
| `assigned_agent` | enum            | yes      | One of `claude`, `gpt`, `hermes`, `any`.                       |
| `created`        | date            | yes      | ISO date `YYYY-MM-DD` when the file was created.               |
| `updated`        | date            | yes      | ISO date `YYYY-MM-DD` of the last meaningful edit.             |
| `dependencies`   | list of strings | no       | Task IDs that must reach `done` before this task may begin.    |
| `context_refs`   | list of strings | no       | Paths relative to `.acp/` for snapshots/sessions to preload.   |
| `files`          | list of strings | no       | Glob patterns indicating the file scope an AI should focus on.  |
| `notes`          | string          | no       | Free-form multi-line notes (YAML `|` block scalar).            |

## Status Lifecycle

```
todo  ──►  doing  ──►  done
```

| Transition  | Trigger                                                  |
|-------------|----------------------------------------------------------|
| `todo → doing` | An agent begins actively working the task.              |
| `doing → done` | All acceptance criteria met; snapshot reflects state.   |
| `doing → todo` | Task is abandoned, blocked, or split.                   |
| `done → doing` | A regression or follow-up reopens the task.             |

An AI agent entering a workspace **must only act on tasks with `status: doing`**. Other statuses are out of scope for the current session unless the user explicitly redirects.

## `assigned_agent` Semantics

- `claude`, `gpt`, `hermes`: the task is tied to a specific agent ecosystem. Other agents should generally skip it unless asked.
- `any`: no agent preference. Any compliant ACP agent may pick up the task.

`assigned_agent` is advisory, not enforcement. It signals intent for routing and load balancing — it does not block other agents.

## Path Resolution

- `context_refs`: paths are **relative to `.acp/`**. Example: `snapshots/auth-design.md` resolves to `.acp/snapshots/auth-design.md`.
- `files`: glob patterns are **relative to the project root** (the directory that contains `.acp/`).

## Validation

A well-formed task file must:

1. Parse as valid YAML.
2. Have a unique `id` matching the filename stem (`tasks/foo-001.yaml` → `id: foo-001`).
3. Have `status` ∈ `{todo, doing, done}`.
4. Have `assigned_agent` ∈ `{claude, gpt, hermes, any}`.
5. Have `created` ≤ `updated` (lexically on ISO dates).
6. List any referenced `dependencies` IDs that exist as sibling task files; missing dependencies are warnings, not errors.
