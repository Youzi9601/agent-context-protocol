# ACP Examples

This file walks through concrete ACP scenarios. It is **not** a specification — it is illustrative material to help maintainers and contributors understand how ACP behaves in practice.

---

## Example 1: Agent A Starts a Project, Agent B Continues

### Project: `task-service` (a REST API)

**Agent A: Initial scaffold**

Agent A is asked to start a new project for a task management API. It scaffolds `.acp/`:

```
task-service/.acp/
├── WORKSPACE.md
├── memory.yaml
├── memory.md
├── tasks/
├── sessions/
└── snapshots/
```

Agent A fills in `memory.yaml` and starts a task.

**Agent A: First task, first session**

After the first conversation, Agent A writes a session summary:

`task-service/.acp/sessions/feature-auth-2026-06-17-001.md`

```markdown
---
task: feature-auth
date: 2026-06-17T09:00:00+08:00
agent: claude-3-5-sonnet
---

Decided to use JWT RS256 for auth tokens. No server-side session store needed.
MongoDB chosen for task storage. Mongoose ODM.
Next: implement /auth/login endpoint returning a signed JWT.
```

**Agent A: After completing auth**

Agent A writes a snapshot:

`task-service/.acp/snapshots/feature-auth-2026-06-17.md`

```markdown
---
task: feature-auth
date: 2026-06-17
agent: claude-3-5-sonnet
---

## Conclusions

- JWT RS256 auth implemented at /auth/login and /auth/verify
- Token expiry: 24h. Refresh not implemented (see Decision Index).
- MongoDB 6.x required for this project.
- All routes under /api/* require valid JWT in Authorization header.

## Next Entry Point

Continue with: feature-task-crud (session: feature-task-crud-2026-06-18-001)
```

Agent A also fills in `memory.md`:

`task-service/.acp/memory.md`

```markdown
# Memory: task-service

## Architecture
Monolithic REST API. Node.js + Fastify + MongoDB/Mongoose.
Auth layer is middleware-based; all /api/* routes are protected.

## Conventions Beyond Code Style
- Feature branches named: feature-*, bugfix-*, docs-*
- Commit format: `type(scope): description`
- All dates in session/snapshot filenames use Asia/Taipei (IANA: Asia/Taipei)

## Recurring Pitfalls
- **Symptom:** Mongoose connection refused on cold start
  **Cause:** MongoDB container starts slower than the app.
  **Fix:** Added `await mongoose.connect()` retry loop with 3 attempts.

## Tribal Knowledge
- The `userId` field on tasks is the MongoDB ObjectId, not the JWT sub.
- Auth tokens are stored in memory only; no refresh token persistence.

## Decision Index
- **JWT RS256, not sessions** — stateless API, multiple mobile clients. Link: snapshots/feature-auth-2026-06-17.md
- **MongoDB, not Postgres** — team familiarity; schema-less适合 task data.
```

---

### Agent B enters the project (day 2)

Agent B is asked to continue the task management API work.

**Agent B runs READ:**

1. Reads `WORKSPACE.md` → goal: task management REST API, focus: feature-task-crud
2. Reads `memory.yaml` → language: Node.js, framework: Fastify + Mongoose
3. Finds active task `feature-task-crud` with `status: doing`
4. Finds latest session: `feature-task-crud-2026-06-18-001.md`
5. Loads `context_refs` snapshots from task.yaml: `feature-auth-2026-06-17.md`

Agent B also checks `context-assembly.yaml` (since this is a `feature` task type, it includes `memory.md`). Agent B reads `memory.md`.

**Agent B produces workspace report:**

```text
**Project:** task-service
**Active task:** feature-task-crud (status: doing)
**Current focus:** implementing /tasks CRUD endpoints

**Reasoning trace:**
  - Auth uses JWT RS256 (stateless, see snapshots/feature-auth-2026-06-17.md)
  - MongoDB 6.x required; Mongoose connection retry fix applied
  - userId on tasks is MongoDB ObjectId, not JWT sub (from memory.md)

**Next action:**
  Implement GET /tasks and POST /tasks endpoints using the auth middleware.
  Mongoose schema already defined in src/models/task.ts.

**What I still need to know:**
  - The Mongoose task schema field names (not yet in any .acp/ file)
```

Agent B reconstructed: what is happening, why decisions were made, what comes next — without asking the user.

---

## Example 2: Bug-Fix with memory.md Pitfall

Agent B is fixing a bug where `userId` occasionally points to the wrong user.

**Agent B runs READ** → sees `memory.md` includes `## Recurring Pitfalls → userId is MongoDB ObjectId, not JWT sub`.

Agent B checks the current code — it is passing `token.sub` (JWT subject string) instead of looking up the MongoDB ObjectId from the token.

Agent B fixes the bug, writes a snapshot, and adds an entry to `## Recurring Pitfalls`:

```markdown
- **Symptom:** userId points to wrong user under concurrent load
  **Cause:** Using `token.sub` (JWT string) as MongoDB ObjectId; sub !== _id
  **Fix:** Lookup user by `token.sub` first, use resulting `_id` on tasks.
  **Link:** snapshots/bugfix-userid-2026-06-18.md
```

Next agent entering the project will read this pitfall entry and avoid the same mistake.

---

## Example 3: Monorepo with `related:` Field

**Project structure:**

```
monorepo/
├── services/
│   ├── task-service/.acp/
│   └── auth-service/.acp/
└── packages/
    └── shared/.acp/
```

**In `task-service/.acp/memory.yaml`:**

```yaml
related:
  - path: ../../auth-service/.acp
    relationship: dependency
  - path: ../../../packages/shared/.acp
    relationship: dependency
```

**Agent B in task-service** checks `related:` → discovers `auth-service` is a dependency. When Agent B is asked to integrate with auth, it reads `auth-service/.acp/WORKSPACE.md` and `auth-service/.acp/memory.yaml` to understand the auth service's state before making integration decisions.

---

## Example 4: `context-assembly.yaml` in Action

**Workspace:** `task-service/.acp/context-assembly.yaml`

```yaml
defaults:
  - WORKSPACE.md
  - memory.yaml

tasks:
  bug-fix:
    - WORKSPACE.md
    - memory.yaml
    - tasks/${task-id}.yaml
    - snapshots/${task-id}-*.md
    - sessions/${task-id}-${today}-*.md

  feature:
    - WORKSPACE.md
    - memory.yaml
    - memory.md
    - tasks/${task-id}.yaml
    - snapshots/${task-id}-*.md
    - sessions/${task-id}-${today}-*.md
```

**Agent B enters with a `bug-fix` task:**

- Loads defaults: WORKSPACE.md, memory.yaml
- Checks task type: `bug-fix` → loads task yaml + all snapshots matching task-id + today's sessions
- Does NOT load `memory.md` (not listed under `bug-fix` — bug fixes don't typically need architecture docs)
- Proceeds with reconstruction

**Agent B enters with a `feature` task:**

- Loads defaults: WORKSPACE.md, memory.yaml
- Checks task type: `feature` → also loads `memory.md` for architecture context
- Loads task yaml + snapshots + sessions
- Proceeds with reconstruction

Both task types load the same core reconstruction files, but `feature` gets additional context that matters for design decisions.