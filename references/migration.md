# ACP Migration Guide: v0.1 → v0.2

v0.2 does not introduce breaking changes. All v0.1 workspaces remain valid. The changes in v0.2 are purely additive optimizations that improve Agent B's ability to reconstruct project state.

This guide covers: what changed, what is optional, and how to add v0.2 artifacts to an existing workspace.

---

## What Changed in v0.2

### New: `memory.md`

A long-form Markdown supplement to `memory.yaml`. Where `memory.yaml` holds structured key/value fields, `memory.md` holds the prose behind those fields: architectural decisions, recurring pitfalls, tribal knowledge, and a decision index.

**Purpose:** Enables Agent B to answer "why was this approach chosen?" — not just "what was chosen."

**v0.1 behavior:** `memory.yaml` exists. `memory.md` does not exist.
**v0.2 behavior:** `memory.yaml` exists. `memory.md` may be added. Both are valid.

### New: `context-assembly.yaml`

An optional manifest that maps task types to the `.acp/` files an agent should load.

**Purpose:** Reduces ambiguity in which files to load for a given task. Agents loading `feature` tasks get `memory.md`; agents loading `bug-fix` tasks do not (unless configured otherwise).

**v0.1 behavior:** Agent loads files via explicit READ.
**v0.2 behavior:** Agent may consult `context-assembly.yaml` for task-type-specific context. Explicit READ is still valid if the manifest is absent.

### New: `related:` field in `memory.yaml`

An optional list of sibling `.acp/` workspaces for monorepo awareness.

**Purpose:** Agent B entering workspace A can discover related workspaces B and C without asking the user.

**v0.1 behavior:** No cross-workspace awareness.
**v0.2 behavior:** `memory.yaml` may carry a `related:` list. One-hop only.

### Snapshot alias convention

The `{task-id}-latest.md` convention is clarified as an explicit alias for the most recent snapshot by date.

**v0.1 behavior:** Snapshot naming convention exists; `{task-id}-latest` alias not explicitly documented.
**v0.2 behavior:** `{task-id}-latest.md` is a documented alias convention. No new file format.

---

## What Stayed the Same

- `.acp/` folder layout (unchanged)
- `task.yaml`, `snapshot.md`, `session.md` formats (unchanged)
- Explicit READ flow (still the canonical entry path)
- `SKILL.md` ≤ 2000 char constraint (preserved)
- All v0.1 workspaces function without modification

---

## Backward Compatibility

v0.2 is **purely additive**. There are no required migrations.

An agent entering a v0.1 workspace behaves identically to v0.1. The new artifacts (`memory.md`, `context-assembly.yaml`, `related:`) are **optional optimizations** — if absent, the agent uses explicit READ and reconstructs what it can.

---

## Adding `memory.md` to a v0.1 Workspace

Copy the starter from `templates/.acp/memory.md` into your project root:

```bash
cp templates/.acp/memory.md my-project/.acp/memory.md
```

Fill in the five sections as the project accumulates decisions:

| Section | What to write |
|---------|--------------|
| `## Architecture` | System structure, key design choices in prose |
| `## Conventions Beyond Code Style` | Non-linter conventions, deployment rituals, code review norms |
| `## Recurring Pitfalls` | Known footguns: **Symptom** / **Cause** / **Fix** |
| `## Tribal Knowledge` | Business-domain assumptions new contributors need |
| `## Decision Index` | One entry per significant decision with reasoning |

Sections may be empty initially. Write `None yet` or `- none` to indicate the section exists but has no content yet.

---

## Adding `context-assembly.yaml` to a v0.1 Workspace

Copy the starter from `templates/.acp/context-assembly.yaml`:

```bash
cp templates/.acp/context-assembly.yaml my-project/.acp/context-assembly.yaml
```

Customize `tasks:` to match your project's actual task types. The `defaults:` section is loaded for all task types; task-type-specific entries override or extend it.

If your project only uses one task type, you can use `defaults:` only and omit the `tasks:` section.

---

## Adding `related:` to `memory.yaml`

Add a `related:` key to the top level of your existing `memory.yaml`:

```yaml
project:
  name: task-service
  # ... existing fields

related:
  - path: ../auth-service/.acp
    relationship: sibling
  - path: ../packages/shared/.acp
    relationship: dependency
```

Only add `related:` if your project has sibling `.acp/` workspaces. If not, omit it.

---

## What Agent B Can Reconstruct After Adding v0.2 Artifacts

| Artifact | Question Agent B can now answer |
|----------|-------------------------------|
| `memory.md` | Why was this approach chosen? What are the known pitfalls? |
| `context-assembly.yaml` | Which files are relevant to this task type? |
| `related:` | Which other workspaces are adjacent? |
| `{task-id}-latest.md` | What was the most recent conclusion on this task? |

Without these artifacts, Agent B still reconstructs current state (task, session, snapshots) via explicit READ. With them, Agent B also reconstructs **reasoning trace** and **task-specific context** — the primary improvement of v0.2.

---

## Verifying a v0.2 Workspace

After adding `memory.md` and `context-assembly.yaml`:

1. Run READ flow manually (as Agent B would)
2. Confirm you can answer:
   - What is happening now? ✓
   - Why is it happening? ✓ (from `memory.md`)
   - What should happen next? ✓ (from `context-assembly.yaml` context loading)
3. Confirm `context-assembly.yaml` parses with `yaml.safe_load`
4. Confirm `memory.md` has all five required sections