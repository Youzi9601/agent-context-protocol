# Contributing to ACP

Thanks for your interest in improving Agent Context Protocol. ACP is a **convention spec**, not software — the contribution surface is mostly Markdown and YAML, which keeps the bar low but the consequences high (every agent reads these files).

## Before You Start

1. Read [`spec/rationale.md`](spec/rationale.md). Many proposals duplicate choices already explained there.
2. Read [`spec/overview.md`](spec/overview.md) to grasp the design principles (**LLM Native**, **Provider Agnostic**, **File First**, **Human Readable**).
3. Skim [`spec/gotchas.md`](spec/gotchas.md) and [`spec/triggers.md`](spec/triggers.md) to learn the failure modes we're trying to prevent.

## What Belongs in a Pull Request

| Change type                  | Where to propose                                 |
|------------------------------|--------------------------------------------------|
| New activation rule          | `spec/triggers.md` + paired scenario in `spec/trigger-tests.md` |
| New failure mode seen in use | `spec/gotchas.md`                                |
| Rationale for an existing rule | `spec/rationale.md`                             |
| File format change           | GitHub Issue first (breaking change to spec)     |
| New spec file                | GitHub Issue first                               |
| A claim that SKILL.md violates Anthropic skill conventions | GitHub Issue with citation |

For ambiguous scope — "does X change the file format or just internal convention?" — open a GitHub Issue before sending code. File format changes break every existing `.acp/` workspace.

## Issue Template — Bug Report (Agent Behavior)

Use this when an ACP-compliant agent **misbehaves** (over-triggers, under-triggers, produces wrong report, ignores `.acp/`, etc.).

```markdown
## Environment

- Agent: <Claude Code / claude.ai / GPT / Hermes / other>
- Agent version: <if applicable>
- ACP-SKILL version: <from SKILL.md header, e.g. v0.1>

## Observed

<paste the agent's actual behavior — full turn transcript preferred over summary>

## Expected

<what the agent should have done, citing spec/*.md>

## Which spec rule was violated

- [ ] spec/triggers.md — wrong activation decision
- [ ] spec/READ.md — skipped a step
- [ ] spec/verification.md — reported without verifying
- [ ] spec/gotchas.md — hit a known gotcha
- [ ] SKILL.md — directly violated
- [ ] Other: <link>

## Repro

<minimal `.acp/` directory state that reproduces it>
```

## Issue Template — Proposal

Use this for **additions** to the spec (not behavior bugs).

```markdown
## Problem

<what is the spec missing or wrong that makes a real workflow impossible or painful>

## Why the existing spec cannot handle this

<cite sections in spec/*.md that you read. If you have not, you will be asked to.>

## Proposed change

<minimal Markdown/YAML edit, with diff if possible>

## Alternative considered

<at least one alternative rejected, with reasoning. PRs without this section
slow down review.>

## Phase tag

- [ ] Phase 1 (v0.1) — backwards compatible, additive only
- [ ] Phase 2 (v0.2) — extends memory/context-assembly
- [ ] Phase 3 (v0.3) — prompt library, conformance tooling
- [ ] Breaking — bump major version
```

## Local Editing Tips

- All spec files use **British or US English consistently within a file**, prefer US English for new content.
- Do not introduce hyperlinks to time-sensitive content (vendor docs rot).
- Do not add emojis unless the file already uses them.
- Character budget on `SKILL.md` is **≤ 2000 characters**. Run `wc -c SKILL.md` before opening a PR that touches it.
- YAML examples must round-trip a standard parser. Do not include placeholders like `<your-value>` that would break parsing.

## Review and Merge

- Maintainer reviews for design rationale (does the change contradict anything in `spec/rationale.md`?).
- Lint locally before sending a PR: `yamllint templates/ spec/` (and `markdownlint` if installed). At the time of writing there is no GitHub Actions workflow in this repo, so a clean local run is the gate. When a workflow is added later, the same jobs must be green on CI.
- Conventional Commits preferred (`feat:`, `fix:`, `docs:`, `refactor:`).
- Maintainer squash-merges to keep `main` linear.

## Code of Conduct

Be precise, not polite. Quote the spec section you disagree with. Save goodwill for the design discussion.
