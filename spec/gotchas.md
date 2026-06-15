# Gotchas and Failure Modes

Real-world agents repeatedly make the same handful of mistakes on ACP workspaces. This file enumerates them so a future agent can recognize and avoid them.

Each gotcha lists: **symptom**, **root cause**, **prevention**.

## 1. Multiple Active Tasks

**Symptom:** Two or more `tasks/*.yaml` files have `status: doing`. The agent picks one arbitrarily or merges them.

**Root cause:** A previous agent marked a task `doing`, never marked it `done`, then started a second one. Or the user worked on two tracks in parallel and forgot to close one.

**Prevention:**

- On entry, **report** the multiple matches and ask which to work on. Do not pick.
- If the user says any is fine, work the **highest-priority** task (lowest ID, or first declared in `WORKSPACE.md`).
- Once chosen, mark the others back to `todo` with a comment in `notes` referencing why they were deferred.

## 2. Empty `tasks/` Directory

**Symptom:** `.acp/tasks/` exists but contains no `*.yaml`.

**Root cause:** Freshly initialized project, or all tasks were archived and `tasks/` was emptied.

**Prevention:**

- Report "No active task" and offer to create one.
- Do **not** invent a default task from the project name.

## 3. Missing Session History

**Symptom:** The active task references no sessions, or all sessions are corrupted.

**Root cause:** This is the first session on the task, or sessions were never persisted.

**Prevention:**

- Report "First session on this task" and base the report on the snapshot alone.
- Treat the absence of session history as **normal**, not an error. Do not fabricate a session.

## 4. Missing Snapshots

**Symptom:** `context_refs` lists a path that does not exist.

**Root cause:** Snapshot was deleted, moved, or never written. Or the path is wrong relative to `.acp/`.

**Prevention:**

- See `spec/verification.md` Check 5. List missing snapshots explicitly in `Open questions`.
- Continue working from the task YAML and any sessions that did load.
- Ask the user whether to (a) recreate the snapshot, (b) fix the path, or (c) remove the broken `context_refs` entry.

## 5. Contradicting Information Between Memory and Tasks

**Symptom:** `memory.yaml` declares a constraint (e.g. "TypeScript only") but a task implicitly violates it (e.g. `files: src/*.py`).

**Root cause:** Stale memory, or task was imported from a different language branch.

**Prevention:**

- Surface the contradiction in `Open questions`.
- Default to **honoring the constraint** (memory.yaml is the durable rule).
- Suggest updating either memory or the task — do not silently overwrite either.

## 6. Stale Active Status

**Symptom:** A task has been `status: doing` for many days/weeks with no corresponding recent session.

**Root cause:** A previous agent or user flipped status but never wrote a closing session.

**Prevention:**

- Treat as low-trust. Report "Last activity on this task was N days ago."
- Ask the user before treating the stale status as authoritative. Possible paths: (a) resume, (b) close it, (c) split into smaller tasks.

## 7. Title vs. ID Drift

**Symptom:** A task's `id` is `auth-feature` but the file is `feature-auth.yaml`. Or `id` is renamed but downstream `context_refs` still use the old one.

**Root cause:** Manual renaming without refactoring `context_refs`.

**Prevention:**

- Treat mismatch as a soft error. Use the file's actual `id` (its filename stem) as source of truth.
- Update every `context_refs` in the affected task to point at the renaming-conformant filename.
- Make a note in the new snapshot's body describing the rename.

## 8. Timezone Confusion in Session Filenames

**Symptom:** A session file at midnight UTC has a `date` frontmatter from a different day than the filename `YYYY-MM-DD` implies.

**Root cause:** The agent used local time for the frontmatter but UTC for the filename, or vice versa.

**Prevention:**

- Decide **one** canonical clock per workspace. The default is the project's home timezone (declared in `memory.yaml` `project.timezone`).
- Filename and frontmatter must use that same clock. If they disagree, the file is malformed and must be repaired.

## 9. Snapshot Stored in Wrong Folder

**Symptom:** A snapshot lives under `sessions/` or `tasks/`, not under `snapshots/`.

**Root cause:** Human (or agent) error during init.

**Prevention:**

- On entry, locate snapshots in `.acp/snapshots/`. Files matching the snapshot naming pattern in other folders are warnings, not data.
- Recommend the user move the file and amend the relevant `context_refs`.

## 10. Agent Reports Before Verifying

**Symptom:** The agent reports "active task is X" when in fact the YAML for X was malformed and ignored.

**Root cause:** Skipped verification step.

**Prevention:**

- This entire file exists because of this gotcha. Always run verification. See `spec/verification.md`.

## General Principle

When you notice a failure mode that isn't listed here, add it. This file is not a design document — it is a post-mortem log the future you will thank the past you for writing.
