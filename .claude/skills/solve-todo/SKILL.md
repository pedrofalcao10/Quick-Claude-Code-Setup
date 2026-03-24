# solve-todo

Executes the full workflow pipeline for resolving a backlog item from `todos/backlog/`.

## Usage

```
/solve-todo <todo-number>
/solve-todo next
```

- Pass a number (e.g., `001`) to work on a specific todo.
- Pass `next` to pick the next item from the **Quick Reference** table at the bottom of the **latest priority file** in `todos/priority/` (the one with the highest number, e.g., `001-priority-todos.md` takes precedence over `000-priority-todos.md`). Scan rows top-to-bottom (by the `Order` column, which defines execution priority). The first row where the `Status` column is `BACKLOG` is the next item. Use the `#` column (todo number) to locate the file, **not** the `Order` column.

## Permission Rules (apply to ALL phases)

- **ALLOW without asking:** All file reads, grep/search, glob, git status/log/diff/branch, running tests (`go test`, `npm test`, `vitest`, `jest`, etc.), build commands, lint commands
- **PAUSE and ask user approval for:** Any file creation, file edit, file deletion, git commits, git push, git checkout (branch switches), GitHub mutations (`gh issue create`, `gh pr create`)

## Rejection & Retry Policy

When the user rejects a phase's output (says "no", requests changes, or wants a different approach):

1. Ask the user what they want changed.
2. Re-run the **current phase** incorporating their feedback.
3. Maximum **3 retries per phase**. After 3 rejections, ask: "Do you want to skip this phase, go back to a previous phase, or abort the pipeline?"

## Workflow Pipeline

Execute each phase **in strict order**. After each skill phase, pause and summarize the output before moving to the next.

### Phase 0 — Setup

**Pre-flight checks (run before anything else):**

1. **Validate input:** If no argument was provided, ask the user for a todo number or `next`. If a number was given, verify the file `todos/backlog/{number}-*.md` exists. If the file is not found, check `todos/doing/` and `todos/done/` — if found there, report the item's current status and stop. If not found anywhere, report "todo not found" and stop.
2. **Check working tree:** Run `git status --porcelain`. If there are uncommitted changes, warn the user and ask whether to stash, commit, or abort before continuing.
3. **Check for existing branch:** Run `git branch --list 'fix/{NUMBER}-*' 'refactor/{NUMBER}-*' 'test/{NUMBER}-*' 'chore/{NUMBER}-*' 'feat/{NUMBER}-*'`. If a branch already exists for this todo (any prefix), ask the user: "A branch for this todo already exists. Switch to it and resume, or delete it and start fresh?"
4. **Check for existing issue:** Run `gh issue list --search "{PRIORITY}/{NUMBER}" --state open`. If a matching issue already exists, reuse its number instead of creating a new one.
5. **Check dependencies:** Read the **Quick Reference** table's Dependencies column for this todo. If any dependency is still `BACKLOG` or `DOING` (check their Status in the same table), warn: "Warning: #{DEP_NUMBER} ({dep description}) is not done yet. This todo depends on it. Continue anyway?" Let the user decide.
6. **Determine development branch (`{DEV_BRANCH}`):**
   - Auto-detect: Run `git branch -a` and check for branches matching common integration branch names: `dev`, `develop`, `development`, `staging`, `next`.
   - If exactly one match is found: announce "Detected development branch: `{name}`. Using this as the integration branch. Say 'no' to override." Use it as `{DEV_BRANCH}`.
   - If multiple matches are found: ask the user which one to use.
   - If no matches are found: ask the user: "What is your development/integration branch name?"
   - Validate the branch exists locally or on remote. If it doesn't exist, ask: "Branch `{name}` doesn't exist yet. Create it from the current branch, or enter a different name?"
   - Use `{DEV_BRANCH}` for all subsequent references to the integration branch in this pipeline.

**Main setup steps:**

1. Read the todo file from `todos/backlog/{number}-*.md` to get all context.
2. Read the priority file `the latest priority file in todos/priority/` to get the priority, order number, and short description for this todo.
3. Extract from the todo file:
   - `PRIORITY`: e.g., P1, P2, P3
   - `NUMBER`: e.g., 001, 002
   - `SHORT_DESC`: a very short (3-6 word) description from the todo title
4. Determine branch prefix based on the todo's source/nature:
   - Security or bug fix → `fix/`
   - Architecture or refactoring → `refactor/`
   - Test coverage → `test/`
   - Cleanup or dead code removal → `chore/`
   - New feature (Source: Feature) → `feat/`
   - Default → `fix/`
   - Branch name: `{prefix}/{NUMBER}-{kebab-case-short-desc}` (e.g., `fix/002-wildcard-cors`, `refactor/016-service-layer`, `feat/029-whatsapp-notifications`)
5. Create a GitHub issue (skip if one already exists per pre-flight check 4):
   - **Title:** `{PRIORITY}/{NUMBER} - {SHORT_DESC}` (e.g., `P1/002 - wildcard CORS config`)
   - **Body:** The full content of the todo `.md` file (problem + fix sections)
   - **Labels:** Add `priority:{PRIORITY}` label if it exists, otherwise create it
   - Command: `gh issue create --title "..." --body "..." --label "..."`
   - Capture the issue number from the output.
6. Ensure the `{DEV_BRANCH}` branch exists:
   - If it doesn't exist locally or remotely: `git checkout -b {DEV_BRANCH} && git push -u origin {DEV_BRANCH}`
   - If it exists only on remote: `git fetch origin {DEV_BRANCH} && git checkout -b {DEV_BRANCH} origin/{DEV_BRANCH}`
   - If it exists locally: `git checkout {DEV_BRANCH} && git pull origin {DEV_BRANCH}`
7. Create a feature branch from `{DEV_BRANCH}` (skip if branch already exists per pre-flight check 3):
   - Command: `git checkout -b {prefix}/{NUMBER}-{kebab-case-short-desc}`
8. Move the todo file to `doing/`: `git mv todos/backlog/{file} todos/doing/{file}`
9. Update the status in `the latest priority file in todos/priority/` from `BACKLOG` to `DOING` for this item.
10. Commit: `git commit -m "chore: start work on #{NUMBER}"`

### Phase 1 — Analysis

**If the todo file contains a `**Brainstorm:**` field** (feature-originated from `/new-feature`):
- Read the linked brainstorm document instead of running `/ce:ideate` and `/ce:brainstorm` from scratch.
- Summarize the key decisions and requirements from the brainstorm.
- Only run `/ce:ideate` if the user explicitly wants to explore additional approaches beyond what the brainstorm covered.

**Otherwise:** Run `/ce:ideate` then `/ce:brainstorm`. Combine ideation and requirements analysis into a single phase. Focus on the specific problem described in the todo file. Explore approaches, then ground the discussion in the codebase and produce a focused requirements summary.

For **Small effort** items (as marked in the priority file): keep this phase brief — a short analysis paragraph is sufficient. Do not over-analyze one-line fixes.

**Pause and ask the user to approve before continuing.**

### Phase 2 — Plan

Run: `/ce:plan`

Create a concrete implementation plan based on the analysis output. Include specific files to change, functions to modify, and test strategy.

**Pause and ask the user to approve before continuing.**

### Phase 3 — Implementation

Run: `/ce:work`

Execute the approved plan. Write the code changes.

**Pause and ask the user to approve before continuing.**

### Phase 4 — Post-implementation Review

Run: `/ce:review`

Comprehensive review of all code changes. Check for:
- Security implications
- Performance impact
- Code quality and conventions
- Test coverage
- Edge cases

Report findings. If critical issues are found, go back to Phase 3 to fix them. **Maximum 3 round-trips between Phase 3 and Phase 4.** After 3 iterations, present remaining issues and ask the user how to proceed (accept as-is, keep iterating, or abort).

**Pause and ask the user to approve the final state before continuing.**

### Phase 5 — Documentation & Knowledge

Run: `/ce:compound`

Document what was learned and any patterns that emerged. This compounds team knowledge for future work.

For **Small effort** items: skip this phase unless something genuinely surprising was learned. A one-line CORS fix does not need a knowledge document.

### Phase 6 — Finalize

1. Ensure all changes are committed on the feature branch.
2. **User Testing Gate (MANDATORY PAUSE):**
   - Present a summary of what was changed and how to test it (include test commands from Phase 2's plan and any manual testing steps).
   - Ask: "All code changes are committed. Please test locally. When done, confirm:
     (a) Everything works — proceed to PR
     (b) Found issues — describe them and I'll go back to fix
     (c) Abort — leave branch as-is for later"
   - If (b): return to Phase 3 with the user's feedback. The same 3-iteration limit from the Phase 3/4 cycle applies.
   - If (c): report current state (branch name, todo still in `doing/`), stop.
3. Push the branch: `git push -u origin {prefix}/{NUMBER}-{kebab-case-short-desc}`
   - If push fails: report the error clearly and do NOT proceed. The todo stays in `doing/`.
4. Move the todo file: `git mv todos/doing/{file} todos/done/{file}`
5. Update the status in `the latest priority file in todos/priority/` from `DOING` to `DONE` for this item.
6. Commit the todo status change: `git commit -m "chore: mark #{NUMBER} as done"`
7. Push the status commit: `git push`
8. Create a Pull Request:
   ```
   gh pr create \
     --base {DEV_BRANCH} \
     --title "{prefix}: {PRIORITY}/{NUMBER} - {SHORT_DESC}" \
     --body "$(cat <<'EOF'
   ## Summary

   Closes #{ISSUE_NUMBER}

   {Brief description of what was done and why}

   ## Changes
   {Bullet list of key changes}

   ## Test Plan
   {How to verify the fix works}

   ---
   Workflow: `/solve-todo {NUMBER}`
   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```
   - If PR creation fails: report the error and provide the exact `gh pr create` command for manual retry.
9. Close the GitHub issue (since PRs target `{DEV_BRANCH}`, not the default branch, GitHub's auto-close via `Closes #` won't trigger):
   ```
   gh issue close {ISSUE_NUMBER} --comment "Resolved via PR #{PR_NUMBER}"
   ```
10. Report the PR URL and issue URL to the user.
11. Switch back to the `{DEV_BRANCH}` branch: `git checkout {DEV_BRANCH}`
    - If checkout fails: warn but do not treat as a pipeline failure (URLs already reported in step 10).

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations.

## Notes

- Each todo file in `todos/backlog/` contains the full problem description and suggested fix.
- The `{DEV_BRANCH}` is the integration branch (determined during Phase 0, pre-flight check 6) — all PRs target `{DEV_BRANCH}`, **never** `main` directly.
- While pushing the branch, it must NEVER be done with the `--force` flag.
