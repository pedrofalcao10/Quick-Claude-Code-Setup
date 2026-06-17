# Branches vs Worktrees — Decision Guide

How to decide *what* to create when starting work, and how to *merge it back* cleanly.
Written for a **local-only workflow** (no remote feature branches, no PRs).

> **This is the canonical reference for the workflow skills.** `/solve-todo` (Phase 0 + Phase 6), `/bug-fix` (Phase 0 + Phase 7), `/review-and-plan` (Phase 4 ordering), and `/new-feature` (overlap → dependency capture) all point here for the branch-vs-worktree call, conflict handling, and parallel merge order. The **repo-specific hotspots below are written for one project (Influencers Hub)** — when reusing these skills in another repo, replace that section with that repo's own shared files and migration paths.

---

## Core concepts (the mental model)

- **Branch** = a movable pointer to a commit. It's *what* you're working on (a line of history). Cheap — just metadata. **Only one branch can be checked out per working directory at a time.**
- **Worktree** = a physical folder on disk with files checked out, tied to a branch. It's *where* you work. Multiple worktrees share **one** `.git` history.

> Branch = bookmark in the book. Worktree = a second physical copy of the book open to a different page.

A branch can be checked out in **at most one worktree at a time** (git enforces this).
You **always need a branch**; a worktree is the optional add-on that lets a second branch be *physically present and runnable at the same time*.

---

## Decision 1: Branch only, or branch + worktree?

You **always create a branch** for non-trivial work. The only question is whether you *also* spin up a worktree.

| Situation | Use |
|-----------|-----|
| Working on **one thing at a time**; fine to finish before starting the next | **Branch only** |
| **Two+ things genuinely in parallel** (bouncing between them, or running both dev servers) | **Branch + worktree** each |
| Quick experiment you might throw away | **Branch only** (cheap to delete) |
| Need feature B's app *running* while you test feature A | **Worktree** (can't run two branches from one folder) |
| Long-running feature that keeps getting interrupted by hotfixes | **Worktree** for the hotfix; feature state stays untouched |

**Honest heuristic:** worktrees only earn their keep when you need two things *physically alive at the same time*. If the plan is "focus A → finish A → focus B," a plain branch is simpler — skip the disk cost and the shared-resource caveats below.

Don't reach for worktrees because they sound powerful. Reach for them when sequential genuinely doesn't work.

---

## Decision 2: Will these two features conflict?

Before going parallel, ask: **do both features edit the same files?**

- **Different pages/modules** → parallel worktrees are safe; usually merge with zero conflicts.
- **Same shared files** → parallel work *will* conflict on merge. Prefer **sequential branches**, or accept you'll resolve conflicts later.

### Repo-specific conflict hotspots (Influencers Hub)
- `server/server.js` (~1400 lines of routes) — high collision risk.
- `src/services/api.ts` — shared API client.
- **Migration files** (`server/db/migrations/`) — two features both adding `mig 0XX` collide on the **number**; one must be renumbered. Picking wrong corrupts the auto-migration runner at boot. **Always pause on migration-number collisions.**

### Repo-specific shared-resource caveat
Parallel worktrees share **one Postgres** (`localhost:54321`) and **one `.env`**.
- Pure frontend / backend-logic features → fully parallel (Vite auto-picks a free port).
- Either feature touches **migrations/schema** → run only one dev server against the DB at a time, or the boot-time migration runner races.

---

## Setup commands (local-only)

```bash
# from the main repo dir
git worktree add ../influencers-hub-featureA -b feat/featureA
git worktree add ../influencers-hub-featureB -b feat/featureB
```

Open a **separate editor/agent session per worktree** so each stays focused on one feature.

```
influencers-hub/              ← worktree 1, branch feat/featureA
influencers-hub-featureB/     ← worktree 2, branch feat/featureB
        ↘                ↙
         shared .git (one history)
```

Branch-only (no worktree):

```bash
git checkout -b feat/featureA   # work, commit, merge, then move on
```

---

## Merging back into main

You merge the **branch**, never "the folder." Same steps whether or not a worktree was used.

```bash
git checkout main
git merge feat/featureA
```

If a worktree was used, delete it once merged:

```bash
git worktree remove ../influencers-hub-featureA
git branch -d feat/featureA
```

### Merging TWO parallel features — order matters
Merge A first, then pull main *into* B (resolving inside B's worktree where you have full context) before merging B back:

```bash
# 1. A is done → merge into main
git checkout main
git merge feat/featureA

# 2. update B with A's changes, resolve conflicts HERE
cd ../influencers-hub-featureB
git merge main
#   ...fix conflicts, commit...

# 3. B is now clean → merge back
cd ../influencers-hub
git merge feat/featureB
```

This surfaces conflicts inside the feature's own context instead of dumping them on `main`.

---

## Resolving conflicts

A conflict happens only when **both branches changed the same lines of the same file**. Git pauses and marks them:

```
<<<<<<< HEAD
code from the branch you're merging INTO (e.g. main)
=======
code from the branch you're merging IN (e.g. featureB)
>>>>>>> feat/featureB
```

Workflow:

```bash
git status          # files listed "both modified" = your conflict list
```

1. Open each conflicted file. For every `<<<<<<<` block, **decide the final code** (keep one side, the other, or combine), then delete all three marker lines.
2. In VS Code, inline "Accept Current / Incoming / Both" buttons appear — **don't accept blindly**; read both sides and pick what's actually correct.
3. Mark resolved and finish:

```bash
git add path/to/resolved-file.ts
git commit          # completes the merge (message pre-filled)
```

**Escape hatch:** `git merge --abort` returns to the pre-merge state, no harm done.

---

## Quick decision flowchart

```
Starting work?
│
├─ One thing at a time? ───────────────► branch only
│
├─ Two+ things in parallel?
│   │
│   ├─ Do they edit the same files? ───► prefer sequential branches
│   │     (server.js, api.ts, migrations)   (or accept merge conflicts)
│   │
│   └─ Different modules? ─────────────► branch + worktree each
│             │
│             └─ Either touches migrations/schema?
│                   └─ yes → run ONE dev server vs the DB at a time
│
└─ Throwaway experiment? ──────────────► branch only (delete after)
```

---

## TL;DR

- **Branch always.** Worktree only when two things must be *alive at once*.
- **Same files = sequential.** Different modules = parallel worktrees.
- **Merge branches, not folders.** For two features: A→main, then main→B, then B→main.
- **Conflicts = both sides edited the same lines.** Read both, pick the right code, delete the markers, `add` + `commit`.
- **Always pause on migration-number collisions** — they break boot.
