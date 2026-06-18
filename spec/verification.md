# Self-Verification

After the READ flow, an agent **must** verify that the expected state was actually loaded before reporting to the user. This catches silent load failures (encoding, permission, parse errors) that produce confidently-wrong reports.

## The Five Verification Checks

The agent ticks each box. **All five must pass** before reporting.

| # | Check                                              | How to verify                                  |
|---|----------------------------------------------------|-----------------------------------------------|
| 1 | `WORKSPACE.md` loaded                              | File opened, non-empty, has `# ` heading.       |
| 2 | `memory.yaml` loaded                               | YAML parsed with no error, not empty.           |
| 3 | Active task identified                             | At most one `status: doing` task, or user-confirmed choice among multiple. |
| 4 | Latest session identified                          | Most recent session for the active task (by filename date + seq) is selected and read. |
| 5 | Referenced snapshots loaded                        | Every path in active task's `context_refs` exists and was read. |

## Verifying Each Check (in detail)

### Check 1: `WORKSPACE.md` loaded

- File exists at `.acp/WORKSPACE.md`.
- Contains at least one `# ` heading.
- If missing or empty: report "WORKSPACE.md missing or empty" and skip the project-name line in the report.

### Check 2: `memory.yaml` loaded

- File exists at `.acp/memory.yaml`.
- YAML parser returns no error.
- Root keys include `project` (or document explicitly that there is no `project` block when missing — that's still a successful load).

### Check 3: Active task identified

- Read every `.acp/tasks/*.yaml`.
- Filter to `status: doing`.
- Zero matches → "Active task: none" in the report; the agent must not invent one.
- One match → that is the active task.
- Multiple matches → the agent must surface the ambiguity:

  > "Multiple tasks have `status: doing`. Which one should I work on?"
  > - feature-002: Add OAuth login
  > - bug-014: Fix race in payment worker

  Wait for user clarification before proceeding.

### Check 4: Latest session identified

- List `.acp/sessions/*.md`.
- Filter to filenames that start with the active task ID (`{task-id}-`).
- Sort matching filename stems `{task-id}-{YYYY-MM-DD}-{seq}.md` lexicographically; the largest is the latest.
- Tiebreaker: if today's date appears multiple times, prefer the highest `seq`. Else prefer the most recent date for that task.
- Open it. If the file is corrupt or frontmatter is malformed, fall back to the next most recent session for that task.
- If no matching session exists, report "First session on this task" and continue from the task file plus referenced snapshots.

### Check 5: Referenced snapshots loaded

- Iterate `active_task.context_refs` (a list of strings).
- For each entry, resolve it relative to `.acp/`.
- Open and read every referenced file.
- If any path is missing, log "Snapshot not found: {path}" in the **Open questions** section, but do not block the report.

## Reporting a Failed Check

When any check fails, the report must include a `## Verification` block:

```
## Verification
- [x] WORKSPACE.md loaded
- [x] memory.yaml loaded
- [x] Active task identified (feature-002: Add OAuth login)
- [x] Latest session identified (feature-002-2026-06-15-02.md)
- [ ] Snapshot not found: snapshots/auth-design.md
```

A failed check **never silently disappears**. The user must see exactly what did and did not load.

## Why This Step Exists

Two recurring failure modes motivate this:

1. **YAML silently parses a partial file.** Some parsers return partial data on error. Treating that as success produces wrong reports.
2. **Path resolution bugs.** `context_refs` paths are relative to `.acp/`; off-by-one mistakes (relative to root, relative to `snapshots/`) are common. Verification makes them visible immediately.

Verification is cheap — five `open()` calls and a list comprehension in your head — and prevents the agent from confidently lying about the project state.
