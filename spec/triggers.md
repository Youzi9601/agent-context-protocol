# Trigger Conditions

This document defines when ACP activates and when it must stay dormant. `SKILL.md` gives the headline rules; this file is the authoritative reference.

## The Activation Question

For every user turn, the agent must answer this question **before** doing anything else:

> "Did the user implicitly or explicitly hand me the project, such that prior context under `.acp/` is relevant?"

If **yes**, run READ. If **no**, ignore `.acp/` entirely and respond normally.

## Positive Triggers (activate ACP)

Activate when **any** of these is true AND the project contains `.acp/`:

| # | Trigger                                                                            |
|---|------------------------------------------------------------------------------------|
| 1 | User opens a conversation about "this project" / "the repo" / "the codebase".      |
| 2 | User says "continue", "resume", "where were we", "what's next" while in a project. |
| 3 | User refers to a task, ticket, or feature that has a `tasks/*.yaml` entry.         |
| 4 | User switches agents mid-task ("let me try Hermes", "I'll ask GPT next").          |
| 5 | User asks the agent to "follow the protocol" / "use ACP".                          |
| 6 | The agent is reading or editing any file under `.acp/`.                            |
| 7 | The user pastes an `.acp/` file path.                                              |
| 8 | The previous turn ended with an ACP report and a new session has begun.            |
| 9 | The agent is asked to write a `snapshot` or `session` file.                        |
| 10 | A commit message, PR description, or issue body mentions a task ID from `.acp/tasks/`. |

If only some are true (e.g. user mentions a project but `.acp/` is missing), fall through to [Negative Triggers](#negative-triggers-do-not-activate) and follow the [No .acp/ Path](#no-acp-path).

## Negative Triggers (do NOT activate)

| # | Negative                                                                    |
|---|------------------------------------------------------------------------------|
| 1 | User asks a general knowledge / Q&A question unrelated to the project.       |
| 2 | Project has no `.acp/` directory.                                            |
| 3 | User says "skip ACP", "no ACP", "without .acp", "ignore the protocol".        |
| 4 | The turn is a one-shot file operation (read X, write Y, search Z).           |
| 5 | User is in a non-code conversation (creative writing, casual chat).          |
| 6 | An older turn referenced the project but the user has moved on.              |
| 7 | The model default behavior would already ignore `.acp/` (e.g. spinning up a fresh subprocess per turn). |

## No `.acp/` Path

When the project lacks `.acp/` but the user appears to want continuity:

1. State plainly: "This project has no `.acp/` directory. ACP requires one for cross-agent continuity."
2. Offer two paths:
   - **Install**: suggest `cp -r templates/.acp/ ./` then customizing `WORKSPACE.md` and `memory.yaml`.
   - **Skip**: continue without ACP. The user accepts that no handoff will be possible later.
3. **Do not invent `.acp/` files.** Invented files break later agents' trust.

## Opt-Out Semantics

The user can opt out for:

- **This turn only**: "skip ACP for this question" — agent proceeds normally this turn; READ resumes next turn.
- **The rest of the session**: "no ACP for this session" — agent ignores `.acp/` until the user re-enables.
- **All future sessions on this project**: edit `memory.yaml` and add `acp: disabled` under a project-level block (Phase 2 feature; see `spec/memory.md`).

Agent must respect the chosen scope exactly. Re-enabling without explicit user input is a bug.

## Ambiguity Rule

When the activation question has no clear answer (mixed signals), the agent must:

1. State the ambiguity in one sentence.
2. Pick the safer side (usually: do not activate, then ask).
3. Never silently activate just because `.acp/` exists — the observable context dominates.

This is the single most important trigger rule. False positives waste tokens and erode user trust; false negatives just trigger a follow-up question.
