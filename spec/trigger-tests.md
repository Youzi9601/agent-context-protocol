# Trigger Test Suite

This document is a manual test set for ACP trigger behavior. It is **not** unit-tested code — apply these scenarios against any candidate agent and observe whether it follows `spec/triggers.md`.

Scenarios assume a workspace with `.acp/` present unless stated otherwise.

## Positive Triggers (10)

The agent **must** run the full READ flow on these turns.

1. **Project handoff.** User: "OK I've pushed the fix, now please pick up where we left off." (Has `.acp/`.)
2. **Resume cue.** User: "Continue with bug-014."
3. **Status probe.** User: "What's the active task on this project?"
4. **Agent switch.** User: "GPT had a good start on the OAuth plan but ran out of context. Take it from here."
5. **Path reference.** User: "Look at `.acp/tasks/feature-002.yaml` and start working on it."
6. **Recent report.** Previous turn ended with an ACP report for `feature-002`; new turn says "Yes, go ahead."
7. **Snapshot write.** User: "Write a snapshot for feature-002 before you stop." (This forces the agent to pick the active task to snapshot.)
8. **Session end.** User: "That's all for today, log a session." (Forces session write for active task.)
9. **Commit message hint.** User: "Commit the OAuth work and reference `feature-002` in the message."
10. **Cross-agent delegation.** User: "Hermes set up the auth scaffolding; please run tests and pick up from their snapshot."

## Negative Triggers (10)

The agent **must not** run the READ flow on these turns.

1. **General Q&A.** User: "What is the difference between yarn and pnpm?" (No project reference.)
2. **No .acp/.** User: "Pick up the auth task" — but the current directory has no `.acp/`.
3. **Opt-out (turn).** User: "Skip ACP for this turn, just answer: what does this regex do?"
4. **Opt-out (session).** User: "Don't use ACP for the rest of this session."
5. **One-off read.** User: "Read the file `~/.zshrc` and tell me what it does." (Unrelated to `.acp/`.)
6. **Casual chat.** User: "How's your day going?"
7. **Creative writing.** User: "Write me a haiku about refactoring." (No project reference.)
8. **Different project.** User is now working in a sibling repo that has no `.acp/`; old `.acp/` from another project should not bleed in.
9. **Stale signal.** User opened the conversation last week about Project A's `.acp/`. Today they ask about Project B. The agent must not read Project A.
10. **Self-deactivation.** During a session, the user says "stop using ACP" — agent must comply for the rest of the session and not silently re-activate.

## Edge Cases (5)

1. **Multiple active tasks.** `.acp/tasks/` has two `status: doing`. Agent must surface and ask, not pick. (See `spec/gotchas.md` #1.)
2. **First session on the task.** Tasks, no sessions, no snapshots. Agent should still produce a coherent report based on task YAML alone, and not invent a session.
3. **Snapshot references missing task.** A snapshot under `.acp/snapshots/foo.md` exists, but `tasks/foo.yaml` does not. Agent should warn and read the snapshot anyway.
4. **Mid-task opt-out.** Agent is mid-READ on feature-002 and user says "no ACP, just answer this one thing." Agent must finish the current turn without writing artifacts, then resume next turn only if user re-enables.
5. **Path-like strings in user message.** User types "I changed `./.acp/tasks/feature-002.yaml`." Agent must NOT execute that as a path operation against the user's source — it's a literal quote. Verify before acting.

## Pass / Fail Criteria

For each scenario, the agent passes when:

- Positive: produced the `[Verification + Report]` block AND the report content matches `.acp/` ground truth.
- Negative: did NOT touch `.acp/` files AND did not mention ACP unless answering a meta question about ACP itself.
- Edge: behavior is documented in the relevant `spec/` file; the agent surfaces the ambiguity rather than guessing.

If more than 1 of 25 scenarios fails, the agent is not ACP-compliant. Fix the agent, not the spec.
