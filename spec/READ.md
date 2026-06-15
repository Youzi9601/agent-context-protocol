# READ: Entering a Workspace

READ is the canonical entry protocol. Every ACP-compliant agent follows it on its first action in a workspace, before doing anything else.

## The Five-Step Entry Flow

When an ACP agent detects a `.acp/` directory in the current project:

1. **Read `.acp/WORKSPACE.md`**
   Establish the goal, tech stack, and what the user cares about right now.

2. **Read `.acp/memory.yaml`**
   Load long-term conventions: language, frameworks, coding style, constraints, key decisions.

3. **Read tasks whose `status` is `doing`**
   From `.acp/tasks/`, parse every `*.yaml` and select those with `status: doing`. If none, look at `status: todo` only if the user has asked to start something new.

4. **Find the latest session for the active task**
   In `.acp/sessions/`, pick the file with the highest sequence number for today's date under that task's ID. If none today, pick the most recent across all dates.

5. **Load `context_refs` snapshots**
   For each path listed in the active task's `context_refs`, read the referenced snapshot under `.acp/`. These carry durable conclusions and the next-entry point.

After all five steps, the agent has a coherent picture and **must** run the [Self-Verification](verification.md) checks before reporting back.

## Required Report Format

After READING, the agent replies with a short structured report:

```
## Workspace Report

**Project:** <name from WORKSPACE.md or directory name>
**Active task:** <ID + title, or "none">
**Where we left off:**
- <bullet from the latest session or snapshot>
- <bullet from the latest snapshot's "Next Entry Point">

**Suggested next action:**
<one concrete sentence>

**Open questions / blockers:**
- <item>
- "none" if empty
```

If no `doing` task exists, the report says "Active task: none" and asks the user which task to pick up.

## Exception Handling

### `.acp/` does not exist
The agent must **not** silently invent one. It should:

1. Inform the user: "This project has no `.acp/` directory. ACP requires one for cross-agent continuity."
2. Offer two paths:
   - **Install**: copy `templates/.acp/` from the ACP repo into the project root and customize.
   - **Skip**: continue without ACP — the user accepts that there will be no cold-start handoff.

### `.acp/` exists but is malformed
- Invalid YAML → read what you can, report the file, refuse to act until fixed.
- Task IDs that don't match their filenames → report the mismatch, refuse to act on that task.
- Missing frontmatter in a session/snapshot → treat as legacy; warn, but include in reading.

### `.acp/` exists but is empty
Report "Workspace initialized but empty. Add tasks to `.acp/tasks/` to begin." Do not begin work.

## When to Update

An agent should **proactively** write `sessions/` and `snapshots/` artifacts when the conditions in [`session.md`](session.md) and [`snapshot.md`](snapshot.md) fire. In addition:

- Before declaring a task `done`, the agent must write a snapshot.
- After the user ends a conversation, the agent should at minimum write a session file.
- Major design decisions made mid-task warrant an out-of-band snapshot, even if no other trigger fires.

## Trigger and Edge-Case References

Activation logic (when to even start this flow): see [`triggers.md`](triggers.md).
Failure modes encountered during the flow: see [`gotchas.md`](gotchas.md).
Pass/fail scenarios for an agent's ACP behavior: see [`trigger-tests.md`](trigger-tests.md).
Why the flow is shaped this way: see [`rationale.md`](rationale.md).

## Why This Order Matters

Steps 1–2 establish the **macro** view (project identity, conventions). Steps 3–4 establish the **micro** view (what's actually running). Step 5 loads the **durable conclusions** the previous agent left behind. Jumping straight to step 5 wastes tokens on stale context; skipping 3 risks starting work on a completed task.
