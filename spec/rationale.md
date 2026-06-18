# Design Rationale

This document explains the **why** behind ACP's choices. New contributors should read it before proposing changes; existing contributors should link back to it when defending a design in PR review.

## Why `tasks/` Exists

The alternative to a structured task list is "let the agent figure it out from session history." That fails because:

- A model has no concept of "this conversation ended with a half-finished feature." The model only knows "this conversation ended."
- Different agents prioritize differently. A shared notion of "the current active task" is a coordination primitive, not a UI nicety.
- Tasks are **commit-able**. `git log` on `tasks/*.yaml` doubles as a project history. Free-form recap emails are not.

The `task.yaml` schema is **flat on purpose**. Nested task hierarchies (epics → stories → subtasks) tend to drift toward project-management tooling rather than agent context, which is what ACP concerns itself with. If a project needs issue trees, point at Linear or GitHub Issues and mirror task IDs in ACP.

## Why `sessions/` Exists

Sessions capture the **flow of an interaction**, not its conclusions. They answer: "what did the agent and user try, what came out, what was left ambiguous?" — an audit-trail question.

We separate sessions from snapshots because:

- Sessions are cheap and writeable on every turn end. They don't need curation.
- Snapshots require a deliberate summary step. They get out of date if written too often.
- Reading ten sessions is a worse cold-start than reading one snapshot. So sessions are write-often, read-sometimes; snapshots are write-sometimes, read-on-entry.

## Why `snapshots/` Exists

A snapshot is the **durable conclusion** plus the **next action**. It is the only artifact an agent strictly needs to resume work after a context reset.

We require sections like "Next Entry Point" because:

- Without it, the cold-start agent produces another LLM-driven reinterpretation of the situation — never the same result twice.
- With it, an agent's first action is deterministic: read the snapshot, do that one thing.

Snapshots are **append-only**. We do not rewrite old snapshots because:

- A future contributor might want to know what the agent believed on day N, not just day N+M.
- Rewrites are how knowledge is lost silently in versioned text systems.

## Why Markdown and YAML

Some competitors use SQLite, JSON, or proprietary binary blobs. We deliberately pick Markdown + YAML because:

- **Grep.** `rg "TODO" .acp/` is a universally-understood operation. JSON also works for that, but Markdown is greppable *and* renderable in GitHub's diff viewer.
- **Diff.** Git diff on `.acp/tasks/feature-002.yaml` is legible to a human. Git diff on `tasks.db/feature-002/row-3` is not.
- **Edit.** Any text editor — Vim, VS Code, Notepad — can edit these files. A database requires a tool. The convention should not require installing anything.
- **LLM training distribution.** Markdown and YAML are heavily represented in pretraining corpora. Models structure their responses in these formats with high accuracy.
- **No parsers.** Markdown is the only one of the two that requires a parser, and the parser (CommonMark) has converged. YAML has stable specs.

What Markdown and YAML are **not** good at:

- Large structured queries (use SQLite / vector DB).
- Concurrent writes from multiple agents (ACP assumes single-writer or human-coordinated writes).
- Binary content (kept outside `.acp/`).

If a project's needs outgrow the medium, ACP is the wrong tool — escalate to a database.

## Why a Convention, Not Software

ACP could have been a CLI, a server, a VS Code extension. We made it a convention for three reasons:

1. **It runs where the agent runs.** A convention in `.acp/` survives moving from Claude Code to GPT to Hermes to a custom in-house agent. A CLI ties to a host.
2. **It survives trust boundaries.** A user can `git diff` the entire ACP workspace and read every byte before trusting it. They cannot `git diff` a daemon.
3. **It costs nothing to adopt.** `cp -r templates/.acp/ .` and you are done. There is no install, no upgrade, no rate limit.

The downside is the agent must do the work of following the convention, and a non-compliant agent will silently produce garbage. That is the trade-off. We mitigate it with `spec/trigger-tests.md` (so the user can detect non-compliance) and `spec/verification.md` (so the agent self-checks).

## Why the Folder Is Named `.acp/`

- The leading `.` keeps it inert in casual `ls` and signals "this directory is system-managed."
- Short, lowercase, matches the protocol name. Mirrors conventions like `.git/`, `.vscode/`.
- Easy to grep: `find . -path '*/.acp/*.yaml'`.

We considered `_acp/`, `acp/`, `.agent/` and rejected each:

- `_acp/` is not hidden and breaks the convention of dot-prefixed meta directories.
- `acp/` (no dot) pollutes `ls` and gets confused with an actual source folder.
- `.agent/` is too generic — it doesn't tell future agents which protocol.

## Why ACP Has No `scripts/`, `references/`, or `assets/` Subdirectories

Anthropic's official Claude Skills layout allows `scripts/`, `references/`, `assets/` alongside `SKILL.md`. ACP does not. This is deliberate, not an oversight.

The defaults are designed for skills that **do work** — call APIs, render documents, run code. ACP does none of those. Its "work" is **convention compliance**, and a convention that ships with executable code is no longer a convention: it is a software project with hidden side effects.

Three consequences follow:

1. **No `scripts/`.** A script under `scripts/` would need a runtime (Python, Node, Bash). Different agents run on different runtimes; introducing one is a hostile act against portability. Any "automation" that becomes important belongs in `spec/context-assembly.md`, which lets the project author describe assembly in YAML — and the agent does the work.
2. **No `references/`.** ACP's references are the spec files themselves (`spec/*.md`). Progressive disclosure is achieved by **citing** the right spec file from `SKILL.md`, not by separating references into a sidecar folder. See [Why the SKILL.md Is So Short](#why-the-skillmd-is-so-short).
3. **No `assets/`.** Assets only make sense for skills that produce artifacts (PDFs, icons, slide decks). ACP produces text files the agent writes itself; bundling templates is the only asset needed, and those live under `templates/` at the repo root, not under the skill directory.

If a project genuinely needs scripts (for instance, to auto-generate a `WORKSPACE.md` from git history), they belong **inside that project's own tooling**, not inside ACP. ACP stays inert.

## Why the SKILL.md Is So Short

The widely-cited guidance for skill metadata — roughly: metadata ~100 tokens, full instructions <5k, bundled resources on demand — is what `SKILL.md` follows here. SKILL.md is the metadata layer: it tells the agent that ACP exists and how to discover deeper docs at `spec/`. Putting the entire spec there would burn the user's context window before the agent has even seen the project.

## Future Rationale Questions

These are deliberately deferred:

- Why no sqlite index? — see above, defer until projects need it.
- Why no shared global `.acp/`? — see `spec/memory.md` for the current approach.
- Why not extend MCP? — see `spec/overview.md`. The answer is layers: MCP for tools, ACP for memory. They compose.
