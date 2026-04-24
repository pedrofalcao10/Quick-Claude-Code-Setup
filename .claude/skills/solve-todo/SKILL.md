# solve-todo

Executes the full workflow pipeline for resolving a backlog item from `todos/backlog/`.

## Operating Methodology — Deploy-Driven

This skill operates under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — identify the systemic bottleneck in the todo before coding; if the fix is symptom-level, flag it.
- First-principles engineering — strip the problem to its core; reject over-engineered plans in Phase 2.
- Minimal Viable Intelligence — Phase 1 analysis ends the moment the approach is obvious. Small-effort items get a paragraph, not a dissertation.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — phase summaries, commit messages, and PR descriptions must state the user/stakeholder outcome, not just the diff.
- Documentation as code — every commit + PR body must let a teammate resume without a sync meeting.
- Radical Transparency — each phase pause ends with 🟢/🟡/🔴 status: what shipped, blockers, next action. No fluff.

**Deploy-Driven (Accelerated Execution)**
- Continuous Value Delivery — target ≤48-hour turnaround from `doing/` to merged PR; if the plan exceeds that, split the todo.
- Automated Governance — let tests, lint, and CI enforce correctness; never `--no-verify`, never `--force` push.
- 80/20 Deployment — ship the 20% of the fix that resolves 80% of the risk/impact first; open a follow-up todo for the rest rather than blocking this PR.

**Value Matrix — choose the optimized column:**

| Dimension | Traditional | Optimized | Value |
|---|---|---|---|
| Problem solving | Deep research then plan | Prototype in public | Faster feedback |
| Project mgmt | Timelines & gantts | Remove friction/blockers | Team velocity |
| Execution | Feature completeness | Deployment frequency | Live user data |
| Strategy | Long-term roadmap | Dynamic pivot capability | Resilience |

Pipeline tripwire: if any phase output can't be explained to a stakeholder in 30 seconds, it isn't ready — rework before moving on.

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
4. **Check for existing issue:** First, check if the todo file contains an `**Issue:** #{NUMBER}` field — if so, reuse that issue number directly. Otherwise, run `gh issue list --search "{PRIORITY}/{NUMBER}" --state open`. If a matching issue already exists, reuse its number instead of creating a new one.
5. **Check dependencies:** Read the **Quick Reference** table's Dependencies column for this todo. If any dependency is still `BACKLOG` or `DOING` (check their Status in the same table), warn: "Warning: #{DEP_NUMBER} ({dep description}) is not done yet. This todo depends on it. Continue anyway?" Let the user decide.
6. **Determine development branch (`{DEV_BRANCH}`):**
   - Auto-detect: Run `git branch -a` and check for branches matching common integration branch names: `dev`, `develop`, `development`, `staging`, `next`.
   - If exactly one match is found: announce "Detected development branch: `{name}`. Using this as the integration branch. Say 'no' to override." Use it as `{DEV_BRANCH}`.
   - If multiple matches are found: ask the user which one to use.
   - If no matches are found: ask the user: "What is your development/integration branch name?"
   - Validate the branch exists locally or on remote. If it doesn't exist, ask: "Branch `{name}` doesn't exist yet. Create it from the current branch, or enter a different name?"
   - Use `{DEV_BRANCH}` for all subsequent references to the integration branch in this pipeline.

7. **Ensure `todos/` is local-only (gitignored).** The entire `todos/` tree is workflow state that must never leave the local machine.
   - Check `.gitignore` for a line matching `todos/` (or `/todos/`). If missing, ask: "Add `todos/` to `.gitignore` so backlog/priority files stay local only?" If yes, append `todos/` to `.gitignore`.
   - Check tracked files: `git ls-files todos/`. If any are listed, ask: "These `todos/` files are currently tracked in git. Untrack them so they become local-only?" If yes: `git rm -r --cached todos/` and commit with `chore: untrack todos/ (local-only workflow state)`.
   - From this point forward, **never** run `git add todos/...`, `git mv todos/...`, or commit anything under `todos/`.

8. **Feature branches are local-only.** The feature branch created below (`{prefix}/{NUMBER}-...`) must **never** be pushed. The pipeline merges it into `{DEV_BRANCH}` locally in Phase 6 and then pushes `{DEV_BRANCH}`. No `git push` of the feature branch at any point. No `gh pr create`.

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
7. Create a **local-only** feature branch from `{DEV_BRANCH}` (skip if branch already exists per pre-flight check 3):
   - Command: `git checkout -b {prefix}/{NUMBER}-{kebab-case-short-desc}`
   - **Do NOT push this branch.** It stays local for the entire lifecycle.
8. Move the todo file to `doing/` locally (no `git mv` — todos/ is gitignored):
   - Unix/bash: `mv todos/backlog/{file} todos/doing/{file}`
   - PowerShell: `Move-Item todos/backlog/{file} todos/doing/{file}`
9. Update the status in the latest priority file in `todos/priority/` from `BACKLOG` to `DOING` for this item. **Do not commit** — `todos/` is gitignored.
10. No commit here. The first commit on the feature branch happens in Phase 3 when real code changes exist.

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

### Phase 6 — User Testing, Local Merge & Finalize

The feature branch is **local-only** and has never been pushed. Validation happens on the local feature branch before it is merged into `{DEV_BRANCH}` locally. Only `{DEV_BRANCH}` ever gets pushed.

1. **Confirm the feature branch state:**
   - Verify the current branch is `{prefix}/{NUMBER}-{kebab-case-short-desc}` (`git rev-parse --abbrev-ref HEAD`).
   - Ensure all **code** changes are committed. (Anything under `todos/` is gitignored and must not be staged — `git status --porcelain | grep '^.. todos/'` must be empty before committing.)
   - Verify the branch has no remote tracking / was never pushed: `git rev-parse --abbrev-ref --symbolic-full-name '@{u}' 2>&1` should fail for this branch (if it succeeds, something pushed it earlier — stop and ask the user how to proceed).

2. **User Testing Gate (MANDATORY PAUSE — do NOT merge or push until the user validates):**
   - Present a summary of what was changed and how to test it on this local feature branch (test commands from Phase 2's plan + any manual steps).
   - Ask: "You are on the local feature branch `{prefix}/{NUMBER}-...` — it has NOT been pushed and NOT merged. Please test everything here. When done, confirm:
     (a) Everything works — merge this branch into `{DEV_BRANCH}` locally and push `{DEV_BRANCH}`
     (b) Found issues — describe them and I'll go back to Phase 3 to fix
     (c) Abort — leave the local feature branch as-is for later (nothing is pushed, nothing is merged)"
   - If (b): return to Phase 3 with the user's feedback. The same 3-iteration limit from the Phase 3/4 cycle applies.
   - If (c): report current state (local branch name, todo still in `doing/`, nothing pushed), stop.

3. **Move the todo locally** (no `git mv` — `todos/` is gitignored):
   - Unix/bash: `mv todos/doing/{file} todos/done/{file}`
   - PowerShell: `Move-Item todos/doing/{file} todos/done/{file}`

4. **Update the priority file status** from `DOING` to `DONE` for this item. **Do not commit** — `todos/` is gitignored.

5. **Switch to `{DEV_BRANCH}` and update it:**
   - `git checkout {DEV_BRANCH}`
   - `git pull origin {DEV_BRANCH}`
   - If the checkout or pull fails, stop and ask the user — do NOT proceed with the merge.

6. **Merge the feature branch into `{DEV_BRANCH}` locally (no push yet):**
   - `git merge --no-ff {prefix}/{NUMBER}-{kebab-case-short-desc} -m "merge {prefix}: {PRIORITY}/{NUMBER} - {SHORT_DESC} (closes #{ISSUE_NUMBER})"`
   - `--no-ff` preserves the feature branch as a logical unit in history.
   - If the merge conflicts, stop and walk the user through resolution before continuing.

7. **Push `{DEV_BRANCH}` (never the feature branch):**
   - `git push origin {DEV_BRANCH}`
   - **Never** `git push` the feature branch. Never use `--force`.
   - If the push is rejected because the remote moved ahead, run `git pull --rebase origin {DEV_BRANCH}` and retry. If the rebase conflicts, stop and ask the user.

8. **Delete the feature branch locally (it was never pushed, so there is no remote branch to delete):**
   - `git branch -d {prefix}/{NUMBER}-{kebab-case-short-desc}`
   - If git refuses with "not fully merged" (shouldn't happen after step 6), stop and ask the user — do **not** use `-D` without explicit approval.

9. **Close the GitHub issue:**
   - Capture the merge commit SHA: `MERGE_SHA=$(git rev-parse HEAD)`
   - `gh issue close {ISSUE_NUMBER} --comment "Resolved in {DEV_BRANCH} at commit ${MERGE_SHA}"`

10. Report to the user:
    - `{DEV_BRANCH}` HEAD commit SHA and URL
    - Issue URL (now closed)
    - Confirmation that the local feature branch has been deleted
    - Reminder: "`todos/` stayed local; only the code merge is on the remote."

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations.

## Notes

- Each todo file in `todos/backlog/` contains the full problem description and suggested fix.
- The `{DEV_BRANCH}` is the integration branch (determined during Phase 0, pre-flight check 6). The feature branch merges into `{DEV_BRANCH}` **locally**, then `{DEV_BRANCH}` is pushed — **never** `main` directly.
- **`todos/` is local-only.** It is gitignored in Phase 0 step 7 and must never be staged, committed, or pushed. Status transitions (BACKLOG → DOING → DONE) happen via plain `mv` + in-place edits to the priority file, never `git mv` and never a commit.
- **Feature branches are local-only.** The `{prefix}/{NUMBER}-...` branch is never pushed to the remote. It exists only to isolate changes until the user validates them, at which point Phase 6 merges it into `{DEV_BRANCH}` and deletes it with `git branch -d`.
- **No pull requests.** The pipeline does not run `gh pr create`. Validation is a local test gate in Phase 6, and integration is a local `--no-ff` merge into `{DEV_BRANCH}`.
- `git push` is **only** allowed for `{DEV_BRANCH}`, and **never** with `--force`.
