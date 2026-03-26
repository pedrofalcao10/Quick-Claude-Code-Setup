# review-and-plan

Reviews the codebase, creates the `todos/` architecture, generates todo files and a priority document, then hands off to `/solve-todo next`.

## Usage

```
/review-and-plan
/review-and-plan <path-to-focus>
```

- With no argument: reviews the entire codebase.
- With a path (e.g., `backend/`, `frontend/src/`): focuses the review on that area only.

## Permission Rules (apply to ALL phases)

- **ALLOW without asking:** All file reads, grep/search, glob, git status/log/diff/branch, running tests, build commands, lint commands
- **PAUSE and ask user approval for:** Any file/directory creation, file edit, file deletion, git commits, git push, git checkout (branch switches), GitHub mutations (`gh issue create`, `gh pr create`)

## Workflow

### Phase 0 — Pre-flight

1. **Check working tree:** Run `git status --porcelain`. If there are uncommitted changes, warn the user and ask whether to stash, commit, or abort before continuing.

2. **Determine development branch (`{DEV_BRANCH}`):**
   - Auto-detect: Run `git branch -a` and check for branches matching common integration branch names: `dev`, `develop`, `development`, `staging`, `next`.
   - If exactly one match is found: announce "Detected development branch: `{name}`." Use it as `{DEV_BRANCH}`.
   - If multiple matches are found: ask the user which one to use.
   - If no matches are found: ask the user: "What is your development/integration branch name?"
   - Validate the branch exists locally or on remote. If it doesn't exist, ask: "Branch `{name}` doesn't exist yet. Create it from the current branch, or enter a different name?"
   - Use `{DEV_BRANCH}` for all subsequent references to the integration branch in this pipeline.

3. **Check if `todos/` architecture exists.** Look for all four directories: `todos/backlog/`, `todos/doing/`, `todos/done/`, `todos/priority/`. Note which exist and which are missing.

4. **Scan for existing state.** Check all three directories (`todos/backlog/`, `todos/doing/`, `todos/done/`) for `.md` todo files, and `todos/priority/` for priority documents.
   - **Determine the next priority document number:** Count existing files in `todos/priority/` (e.g., if `000-priority-todos.md` exists, the next is `001`). The new priority document will be named `{NEXT_PRIORITY_NUMBER}-priority-todos.md`.
   - **Determine the next todo file number:** Find the highest `{NUMBER}` across all `.md` files in `backlog/`, `doing/`, and `done/`. New findings start from `highest + 1`. If no files exist anywhere, start from `001`.
   - **If existing todo files are found in `backlog/`** (items not yet solved):
     - Report how many existing todo files were found and in which directories.
     - Ask the user: "Existing backlog findings found. **Keep** them and add new findings alongside, **overwrite** them with a fresh review, or **abort**?"
     - **Keep:** New findings are numbered from `highest + 1` (no duplicates since numbering continues). Existing backlog files are left untouched. A new priority document is generated that includes both old and new findings.
     - **Overwrite:** Only deletes files in `todos/backlog/`. Files in `todos/doing/` and `todos/done/` are **never touched** (they represent in-progress or completed work). New findings still start from `highest + 1` across all directories to avoid number collisions.
   - **If no backlog files exist** (all previous items solved or fresh codebase): proceed normally. New findings start from `highest + 1` across `doing/` and `done/`, or from `001` if no files exist anywhere.

5. **If any directories are missing:** proceed to Phase 1. Otherwise skip to Phase 2.

### Phase 1 — Create Architecture

Create the `todos/` directory structure. Only create directories that do not already exist:

```
todos/
  backlog/      # Items not yet started (status: BACKLOG)
  doing/        # Items in progress (status: DOING)
  done/         # Completed items (status: DONE)
  priority/     # Priority/execution order document
```

### Phase 2 — Code Review

Run: `/ce:review`

Perform a comprehensive code review of the codebase (or focused path if provided). The review should identify issues across these categories:

- **Security** — vulnerabilities, auth gaps, injection risks, data exposure
- **Performance** — slow queries, memory leaks, missing indexes, unbounded loads
- **Architecture** — missing layers, tight coupling, global state, untestable code
- **TypeScript** — `any` types, missing interfaces, broken references
- **Go** — error handling, goroutine safety, interface compliance
- **Simplicity** — dead code, fake data, no-op functions, unnecessary complexity

For each finding, capture:
- A short title (3-6 words)
- Priority: `P1 CRITICAL`, `P2 IMPORTANT`, or `P3 NICE-TO-HAVE`
- Source category (Security, Performance, Architecture, TypeScript, Go, Simplicity)
- Affected file(s) with line numbers
- Problem description (2-3 sentences)
- Suggested fix (1-2 sentences)

**Pause and present all findings to the user for approval before continuing.** The user may add, remove, or reprioritize findings. If the user adds a finding with vague details, investigate the codebase to fill in affected files and line numbers before generating the todo file.

**Zero findings:** If the review produces no findings (or the user removes all findings during approval), skip Phase 3 but still proceed to Phase 4 to create a minimal priority document with empty tables. This ensures the `todos/priority/` directory has at least one file, which is a prerequisite for `/new-feature`.

### Phase 3 — Generate Todo Files

For each approved finding, create a `.md` file in `todos/backlog/`.

**File naming convention:**
```
{NUMBER}-{priority-lowercase}-{kebab-case-description}.md
```
Examples:
- `001-p1-no-auth-dashboard.md`
- `008-p2-hardcoded-credentials.md`
- `019-p3-no-security-headers.md`

Numbers are zero-padded to 3 digits, assigned sequentially starting from `001`. If merging with existing files, scan **all three directories** (`backlog/`, `doing/`, `done/`) for the highest existing number and continue from highest + 1.

**File template:**
```markdown
# #{NUMBER} - {Title}

- **Priority:** {P1 CRITICAL | P2 IMPORTANT | P3 NICE-TO-HAVE}
- **Source:** {Security | Performance | Architecture | TypeScript | Go | Simplicity}
- **Files:** {`path/to/file.ext:line-range`, ...}

## Problem

{2-3 sentence description of the issue and its impact.}

## Fix

{1-2 sentence description of the suggested fix approach.}
```

Always use `**Files:**` (plural), even for single-file findings.

### Phase 4 — Generate Priority Document

Create `todos/priority/{NEXT_PRIORITY_NUMBER}-priority-todos.md` (using the number determined in Phase 0 step 3). Each review cycle gets its own priority document. Examples:
- First review: `000-priority-todos.md`
- Second review (after all items solved): `001-priority-todos.md`
- Third review: `002-priority-todos.md`

Structure:

```markdown
# {Project Name} - Code Review Findings

> Generated: {YYYY-MM-DD} | Branch: {current branch} | Commit: {short hash}

---
```

Then, for each priority level (P1, P2, P3), generate a section with all findings:

```markdown
## P1 - CRITICAL (Blocks Production Deployment)

### #{NUMBER} - {Title}
- **Status:** BACKLOG
- **Source:** {category}
- **Files:** {file paths}
- **Description:** {problem description}
- **Fix:** {suggested fix}

---
```

Then generate the **Recommended Execution Order** as a single flat table. Order by: security impact first, then dependencies, then effort-to-value ratio.

```markdown
## Recommended Execution Order

| Order | # | Todo | Priority | Effort | Dependencies | Rationale |
|-------|---|------|----------|--------|--------------|-----------|
| {seq} | {todo#} | {short title} | {P1/P2/P3} | {Small/Medium/Large (~time)} | {None or #dep} | {1-line rationale} |
```

Effort tiers (use exactly one — no hybrid values):
- **Small:** ~5 min to ~1 hr
- **Medium:** ~2-4 hrs
- **Large:** ~6-8+ hrs

Finally, generate the **Quick Reference** table (used by `/solve-todo next`):

```markdown
## Quick Reference

| Order | # | Todo | Priority | Effort | Dependencies | Status |
|-------|---|------|----------|--------|--------------|--------|
| {order} | {todo#} | {short title} | {P1/P2/P3} | {effort} | {None or #dep} | BACKLOG |
```

The Quick Reference table must list ALL findings sorted by execution order. This is the table that `/solve-todo next` parses. It includes a Dependencies column so `/solve-todo` does not need to cross-reference other tables.

**Pause and present the priority document to the user for final approval.**

### Phase 5 — Commit & Hand Off

1. Ensure `{DEV_BRANCH}` is up to date: `git checkout {DEV_BRANCH} && git pull origin {DEV_BRANCH}`. If the work was done on a different branch, cherry-pick or merge the changes into `{DEV_BRANCH}`.
2. Stage all new files in `todos/`.
3. Commit on `{DEV_BRANCH}`: `git commit -m "chore: add code review findings for {scope}"` where `{scope}` is the path argument or "full codebase".
4. Report a summary:
   - Total findings by priority (P1/P2/P3)
   - Total estimated effort
5. Ask the user: **"Ready to start solving? Run `/solve-todo next` to begin with the first item, or `/solve-todo {number}` to pick a specific one."**

If the user says yes or gives a number, invoke `/solve-todo` with the appropriate argument.

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations.
