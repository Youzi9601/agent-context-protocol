# Agent Context Protocol (ACP)

> A workspace convention so any AI agent resumes a project instantly, without the user re-explaining the goal, status, or rationale.

ACP is **not a software tool, not a CLI, not a server**. It is a plain-text convention under `.acp/` at the project root, in Markdown and YAML, designed to be discovered by any large language model.

It complements MCP: **MCP gives an agent hands (tools)**; **ACP gives an agent memory (workspace state)**.

---

## The Problem ACP Solves

When an AI agent opens a project, it has no memory of prior work. The user either re-explains the entire context, or the agent wanders until it stumbles into something coherent. This is the **Agent Cold Start Problem** — the dominant tax on long-running, multi-agent, multi-session work.

ACP eliminates it by standardizing a small set of files that an agent reads before doing anything:

- `.acp/WORKSPACE.md` — goal, stack, current focus.
- `.acp/memory.yaml` — long-term conventions and constraints.
- `.acp/tasks/*.yaml` — units of work, with explicit status.
- `.acp/sessions/*.md` — per-conversation summaries.
- `.acp/snapshots/*.md` — durable cross-session conclusions.

---

## Quick Start

### 1. Install the Skill (one-time)

Copy [`SKILL.md`](SKILL.md) into the agent's system prompt, custom instructions, or skills directory of Claude / GPT / Hermes / your in-house agent. That's it.

### 2. Initialize a New Project

The ACP-aware agent should scaffold `.acp/` directly: read each starter from `templates/.acp/` and write its contents into your project root. If you prefer to do it manually:

```bash
cp -r templates/.acp/ ./my-project/.acp/
```

Either way, edit `.acp/WORKSPACE.md` to describe the project's goal, and `.acp/memory.yaml` to record language, frameworks, and conventions.

### 3. Switch Agents Mid-Project

Open the project in any ACP-compliant agent. The agent reads `.acp/`, identifies the active task, and reports the current state. **No verbal preamble required.**

(For the full report structure including the `## Verification` block, see [`spec/READ.md`](spec/READ.md).)

```text
**Project:** my-project
**Active task:** feature-002: Add OAuth login
**Where we left off:**
  - JWT middleware finished
  - Next: integration tests in src/auth/auth.test.ts
**Suggested next action:** Write the integration tests.
**Open questions:** none
```

---

## Repository Layout

```
.
├── README.md                this file
├── SKILL.md                 the installable skill (≤2000 chars)
├── .gitignore               excludes plan.md and local-only artifacts
├── spec/                    ACP protocol definition
│   ├── overview.md          problem, principles, roadmap
│   ├── folder-structure.md  what lives under .acp/
│   ├── READ.md              entry flow
│   ├── task.md              task.yaml format
│   ├── snapshot.md          snapshot.md format + latest alias
│   ├── session.md           session.md format
│   ├── triggers.md          when to activate / stay dormant
│   ├── verification.md      self-check after READ
│   ├── gotchas.md           failure modes
│   ├── trigger-tests.md     positive/negative/edge scenarios
│   ├── rationale.md         why each design choice
│   ├── memory.md            (v0.2) long-form reasoning layer
│   ├── context-assembly.md  (v0.2) task-type context rules
│   └── prompts.md           (v0.3) versioned prompt fragments
├── templates/
│   └── .acp/                starter files to copy into a new project
│       ├── WORKSPACE.md
│       ├── memory.yaml
│       ├── memory.md        (v0.2)
│       ├── context-assembly.yaml  (v0.2, optional)
│       ├── prompts/         (v0.3) — .gitkeep at init
│       └── tasks, sessions, snapshots/   each holds .gitkeep at init
├── references/              optional supporting material
│   ├── examples.md          ACP walkthrough: Agent A / Agent B scenarios
│   └── migration.md          upgrading from v0.1 to v0.2
└── examples/
    └── example-task.yaml    reference task file; not copied into scaffolds
```

---

## Relationship to MCP

| Aspect       | MCP                                          | ACP                                              |
|--------------|----------------------------------------------|--------------------------------------------------|
| Full name    | Model Context Protocol                       | Agent Context Protocol                           |
| Layer        | Tool calls / external data                   | Workspace state / memory                         |
| Primary read | Programmatic (an MCP client)                 | A large language model (any agent)               |
| Format       | JSON-RPC over stdio / HTTP / SSE             | Markdown and YAML under `.acp/`                  |
| Install      | Run an MCP server                            | Paste `SKILL.md` into the agent                  |

The two compose. A workspace with both MCP servers and `.acp/` gives an agent hands **and** memory.

---

## Roadmap

- **Phase 1 — v0.1 (released).** Folder layout, file formats, READ flow, `SKILL.md`. Progressive disclosure: short SKILL.md links to richer `spec/` documents.
- **Phase 2 — v0.2 (released).** `memory.md` (long-form reasoning), `context-assembly.yaml` (task-type context rules), cross-workspace `related:` field.
- **Phase 3 — v0.3 (current).** Optional `prompts/` library, conformance suite, migration helpers from common stores.

See `spec/overview.md` and `spec/rationale.md` for the full reasoning.

---

## Contributing

Issues and PRs are welcome. Before proposing changes:

1. Read [`spec/rationale.md`](spec/rationale.md). Many rejected proposals were re-inventing something rationale already explains.
2. For ambiguous scope (does X change the file format or just internal convention?), start a GitHub Issue first.
3. New failure modes belong in [`spec/gotchas.md`](spec/gotchas.md). New activation rules belong in [`spec/triggers.md`](spec/triggers.md) plus a paired scenario in [`spec/trigger-tests.md`](spec/trigger-tests.md).

`plan.md` is intentionally **not** committed; it is an internal design document for the maintainer. See [`.gitignore`](.gitignore).

---

## License

[MIT](LICENSE).
