# bug-fix

Resolves a specific, reported bug through a strict reproduce → root-cause → fix-with-regression-test pipeline. Mirrors the local-only branch/todos workflow used by `/solve-todo`, specialized for bug resolution.

## Operating Methodology — Deploy-Driven

This skill operates under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — the reported symptom is rarely the root cause; chase the failure until you find what actually broke.
- First-principles engineering — fix the cause, not the symptom. Reject surface-level patches that mask the real defect.
- Minimal Viable Intelligence — reproduce, diagnose, fix. No speculative refactors, no "while we're here" cleanup.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — the bug report, issue, commit, and merge message all state the user-visible impact, not just the stack trace.
- Documentation as code — the regression test *is* the documentation; it captures exactly what was broken and proves it stays fixed.
- Radical Transparency — each phase summary ends with 🟢/🟡/🔴 status: what reproduced, what was diagnosed, what is verified, what is next.

**Deploy-Driven (Accelerated Execution)**
- Continuous Value Delivery — target ≤24h from reported bug to merged fix on `{DEV_BRANCH}`; if the diagnosis blows past that, report back and re-scope.
- Automated Governance — the failing regression test is the guardrail; never `--no-verify`, never `--force` push.
- 80/20 Deployment — fix the reproducible defect now; if you find adjacent issues, open follow-up todos instead of bundling them.

**Value Matrix:**

| Dimension | Traditional | Optimized | Value |
|---|---|---|---|
| Problem solving | Deep research then plan | Prototype in public | Faster feedback |
| Project mgmt | Timelines & gantts | Remove friction/blockers | Team velocity |
| Execution | Feature completeness | Deployment frequency | Live user data |
| Strategy | Long-term roadmap | Dynamic pivot capability | Resilience |

Pipeline tripwire: if a phase can't produce a reproducible failure, a named root cause, or a passing regression test when it's supposed to, stop and report — do not paper over the gap.

## Usage

```
/bug-fix <description>
/bug-fix
```

- Pass a description of the observed bug (e.g., `/bug-fix PDF export crashes on orders with >100 line items`).
- With no argument: the skill asks for a description.

## Explicit Bug-Fix Rules (non-negotiable)

1. **No fix without reproduction.** Phase 1 must produce reproduction steps that fail deterministically on the current code. If you cannot reproduce, stop and ask the user for more detail (logs, payloads, environment) — do not guess.
2. **No fix without root cause.** Phase 2 must name the specific commit/function/line/input that introduces the incorrect behavior. "Added a null check" is not a root cause; "`parseItems` assumes `line_items` is always an array but the new export path sends `null` when the order is empty" is.
3. **Every fix ships with a regression test.** Phase 1 writes a failing test; Phase 4 makes it pass. The test must fail on the pre-fix code and pass on the post-fix code. No regression test → no merge.
4. **Smallest possible diff.** Change only what's required to fix the bug. No drive-by refactors, no renames, no formatting sweeps in unrelated files. If you spot adjacent issues, file them as new todos via `/new-feature` or `/review-and-plan`, don't fold them in.
5. **Todos and feature branches are local-only.** Same rules as `/solve-todo`: `todos/` is gitignored, the `fix/{NUMBER}-...` branch is never pushed, only `{DEV_BRANCH}` is ever pushed — never with `--force`.
6. **User testing is a hard gate.** The local `fix/...` branch is not merged into `{DEV_BRANCH}` until the user has manually validated the fix on that branch.

## Permission Rules (apply to ALL phases)

- **ALLOW without asking:** All file reads, grep/search, glob, git status/log/diff/branch, running tests (`go test`, `npm test`, `vitest`, `jest`, etc.), build commands, lint commands
- **PAUSE and ask user approval for:** Any file creation, file edit, file deletion, git commits, git checkout (branch switches), git merges, git push

## Rejection & Retry Policy

When the user rejects a phase's output (says "no", requests changes, or wants a different approach):

1. Ask the user what they want changed.
2. Re-run the **current phase** incorporating their feedback.
3. Maximum **3 retries per phase**. After 3 rejections, ask: "Do you want to skip this phase, go back to a previous phase, or abort the pipeline?"

## Workflow Pipeline

Execute each phase **in strict order**. After each skill phase, pause and summarize the output before moving to the next.

### Phase 0 — Setup

**Pre-flight checks (run before anything else):**

1. **Validate input:** If no argument was provided, ask: "What's the bug? Describe the observed behavior, the expected behavior, and any reproduction steps or error messages you have."
2. **Check working tree:** Run `git status --porcelain`. If there are uncommitted changes, warn and ask whether to stash, commit, or abort.
3. **Determine development branch (`{DEV_BRANCH}`):**
   - Auto-detect: Run `git branch -a` and check for branches matching common integration branch names: `dev`, `develop`, `development`, `staging`, `next`.
   - If exactly one match is found: announce "Detected development branch: `{name}`. Using this as the integration branch. Say 'no' to override." Use it as `{DEV_BRANCH}`.
   - If multiple matches are found: ask the user which one to use.
   - If no matches are found: ask the user: "What is your development/integration branch name?"
   - Validate the branch exists locally or on remote. If it doesn't exist, ask: "Branch `{name}` doesn't exist yet. Create it from the current branch, or enter a different name?"
   - Use `{DEV_BRANCH}` for all subsequent references to the integration branch in this pipeline.
4. **Validate prerequisites.** Check that `todos/backlog/`, `todos/doing/`, `todos/done/`, `todos/priority/` all exist AND that at least one priority file exists in `todos/priority/`. If missing, tell the user: "The todos/ architecture is missing. Run `/review-and-plan` first to set up the backlog structure, or confirm if you'd like me to create the directories now." Let the user decide.
5. **Ensure `todos/` is local-only (gitignored).** The entire `todos/` tree is workflow state that must never leave the local machine.
   - Check `.gitignore` for a line matching `todos/` (or `/todos/`). If missing, ask: "Add `todos/` to `.gitignore` so backlog/priority files stay local only?" If yes, append `todos/` to `.gitignore`.
   - Check tracked files: `git ls-files todos/`. If any are listed, ask: "These `todos/` files are currently tracked in git. Untrack them so they become local-only?" If yes: `git rm -r --cached todos/` and commit with `chore: untrack todos/ (local-only workflow state)`.
   - From this point forward, **never** run `git add todos/...`, `git mv todos/...`, or commit anything under `todos/`.
6. **Feature branches are local-only.** The `fix/{NUMBER}-...` branch created below must **never** be pushed. Phase 7 merges it into `{DEV_BRANCH}` locally and then pushes `{DEV_BRANCH}`. No `git push` of the feature branch at any point. No `gh pr create`.
7. **Check for partial prior run / duplicates.** Scan `todos/backlog/`, `todos/doing/`, and `todos/done/` for any existing todo whose title contains the same significant nouns/verbs as the user's description. If overlaps are found:
   - Report them: "These existing items look related: #{NNN} - {title}, ..."
   - Ask: "Continue with a new bug-fix, resume work on one of these existing items (`/solve-todo {NUMBER}`), or abort?"

**Main setup steps:**

1. **Derive bug metadata:**
   - `NUMBER`: highest `{NNN}` across `todos/backlog/`, `todos/doing/`, `todos/done/` + 1, zero-padded to 3 digits. If none exist, start at `001`.
   - `PRIORITY`: propose `P1 CRITICAL` for data loss / security / prod outage, `P2 IMPORTANT` for broken user flows with workaround, `P3 NICE-TO-HAVE` for minor/cosmetic. Ask the user to confirm or override.
   - `SHORT_DESC`: a 3-6 word kebab-case description of the bug (e.g., `pdf-export-empty-order-crash`).
   - `AFFECTED FILES`: best guess from grep/read. Use `TBD` if unknown at this point.
2. **Create the local-only todo file** in `todos/backlog/{NUMBER}-{pN}-{kebab-desc}.md` (no `git add` — gitignored):

   ```markdown
   # #{NUMBER} - {Title}

   - **Priority:** {P1 CRITICAL | P2 IMPORTANT | P3 NICE-TO-HAVE}
   - **Source:** Bug
   - **Files:** {`path/to/file.ext` or `TBD`}

   ## Observed Behavior

   {What the user sees going wrong — the symptom.}

   ## Expected Behavior

   {What should happen instead.}

   ## Reproduction (draft — validated in Phase 1)

   {Steps provided by the user, or "TBD — validate in Phase 1" if not yet known.}

   ## Root Cause

   TBD — determined in Phase 2.

   ## Fix

   TBD — drafted in Phase 3.
   ```

3. **Append to the latest priority file** in `todos/priority/` (no `git add` — gitignored). Add a row to the `### Bugs` section in both **Recommended Execution Order** and **Quick Reference**. If no `### Bugs` section exists, create one after the last existing section. Use `Order = max(existing Order across all tables) + 1`.
4. **Ensure `{DEV_BRANCH}` exists and is current:**
   - If it doesn't exist locally or remotely: `git checkout -b {DEV_BRANCH} && git push -u origin {DEV_BRANCH}`
   - If it exists only on remote: `git fetch origin {DEV_BRANCH} && git checkout -b {DEV_BRANCH} origin/{DEV_BRANCH}`
   - If it exists locally: `git checkout {DEV_BRANCH} && git pull origin {DEV_BRANCH}`
5. **Create a local-only feature branch from `{DEV_BRANCH}`:**
   - `git checkout -b fix/{NUMBER}-{kebab-case-short-desc}`
   - **Do NOT push this branch.** It stays local for the entire lifecycle.
6. **Move the todo to `doing/` locally** (no `git mv` — `todos/` is gitignored):
   - Unix/bash: `mv todos/backlog/{file} todos/doing/{file}`
   - PowerShell: `Move-Item todos/backlog/{file} todos/doing/{file}`
7. **Update the status in the latest priority file** from `BACKLOG` to `DOING` for this item. **Do not commit** — `todos/` is gitignored.
8. No commit here. The first commit on the feature branch happens in Phase 1 when the failing regression test lands.

### Phase 1 — Reproduce (MANDATORY GATE)

The bug cannot be fixed until it can be reproduced deterministically on the current code.

1. **Establish reliable reproduction.** Start from the user's description + any logs/payloads. Attempt to reproduce via:
   - An existing test that can be extended
   - A new unit/integration test
   - A scripted CLI invocation or HTTP request
2. **Write a failing test that captures the bug.** The test must:
   - Fail deterministically on the current (pre-fix) code
   - Describe the correct behavior, not just the current broken behavior
   - Live in the right test file for the affected module
3. **Record the reproduction in the todo file.** Replace the "Reproduction (draft)" section with the validated steps and the path to the failing test.
4. **Commit the failing test on the feature branch:** `git commit -m "test: add failing regression test for #{NUMBER} - {SHORT_DESC}"`.
5. **If the bug cannot be reproduced** after reasonable effort:
   - Do NOT proceed to Phase 2.
   - Report the reproduction attempts made and what information is still missing.
   - Ask the user for: exact payload/input, exact environment, exact sequence of actions, logs/traces.
   - If the user cannot provide enough info, ask: "Keep the todo in `doing/` with 'cannot reproduce' notes, or move it back to `backlog/` with a `needs-repro` note?"

**Pause and ask the user to approve the reproduction before continuing.**

### Phase 2 — Root Cause Analysis

Run: `/ce:brainstorm` focused narrowly on "why is the failing test failing?"

1. Trace the defect to the specific code path: file, function, line, and the exact input/state condition that triggers it.
2. Identify the commit/PR that introduced the behavior if relevant (`git log -S`, `git blame`). Note it — but do NOT revert without explicit user approval; reverts often bring back their own bugs.
3. Confirm the scope of the bug: is it isolated to the reported path, or does the same root cause affect other code paths? If broader, list the other affected paths — they may warrant follow-up todos (do not fold them into this fix).
4. Update the todo file's **Root Cause** section with the validated explanation.

For trivial bugs (typo, off-by-one in an obviously isolated function): this phase can be a single paragraph. For non-trivial bugs: a short diagnosis document.

**Pause and ask the user to approve the root-cause analysis before continuing.**

### Phase 3 — Plan the Fix

Run: `/ce:plan`

1. Describe the **smallest possible diff** that addresses the root cause.
2. Explicitly list files that will change and files that will **not** change (to guard against scope creep).
3. Include the test strategy — the failing test from Phase 1 must pass, and any additional edge-case tests should be called out.
4. Update the todo file's **Fix** section with the chosen approach.

If the plan requires changes beyond the reported bug (e.g., a broader refactor), stop and ask the user: "This fix requires {scope}. Proceed, or file a follow-up todo for the refactor and ship the narrow fix first?"

**Pause and ask the user to approve before continuing.**

### Phase 4 — Implementation

Run: `/ce:work`

1. Execute the approved plan. Write the code changes.
2. Run the failing test from Phase 1 — it must now pass.
3. Add any additional regression tests called out in Phase 3.
4. Run the full test suite for the affected module(s). Do not proceed until green.
5. Commit the fix on the feature branch with a clear, value-framed message: `git commit -m "fix: {user-visible outcome} ({PRIORITY}/{NUMBER})"` (e.g., `fix: PDF export no longer crashes on empty orders (P1/042)`).

**Pause and ask the user to approve before continuing.**

### Phase 5 — Post-fix Review

Run: `/ce:review`

Comprehensive review of the fix. Check for:
- **Regression surface:** does the change affect any other code paths that share the modified function/state?
- **Edge cases:** empty inputs, null/undefined, concurrency, boundary conditions — does the fix still hold?
- **Test coverage:** are the regression tests complete, not brittle, and named meaningfully?
- **Scope creep:** no unrelated changes snuck in.
- **Security/performance impact:** did the fix introduce a new vulnerability or hot-loop?

Report findings. If critical issues are found, go back to Phase 4 to fix them. **Maximum 3 round-trips between Phase 4 and Phase 5.** After 3 iterations, present remaining issues and ask the user how to proceed (accept as-is, keep iterating, or abort).

**Pause and ask the user to approve the final state before continuing.**

### Phase 6 — Documentation & Knowledge (optional)

Run: `/ce:compound`

Document what was learned — especially the root cause if it was non-obvious. This compounds team knowledge for future debugging.

Skip this phase for trivial bugs (typo, obvious off-by-one) unless something genuinely surprising was learned about the system. A one-line null-check fix does not need a knowledge document.

### Phase 7 — User Testing, Local Merge & Finalize

The feature branch is **local-only** and has never been pushed. Validation happens on the local feature branch before it is merged into `{DEV_BRANCH}` locally. Only `{DEV_BRANCH}` ever gets pushed.

1. **Confirm the feature branch state:**
   - Verify the current branch is `fix/{NUMBER}-{kebab-case-short-desc}` (`git rev-parse --abbrev-ref HEAD`).
   - Ensure all **code** changes are committed. (Anything under `todos/` is gitignored and must not be staged — `git status --porcelain | grep '^.. todos/'` must be empty before committing.)
   - Verify the branch has no remote tracking / was never pushed: `git rev-parse --abbrev-ref --symbolic-full-name '@{u}' 2>&1` should fail for this branch (if it succeeds, something pushed it earlier — stop and ask the user how to proceed).

2. **User Testing Gate (MANDATORY PAUSE — do NOT merge or push until the user validates):**
   - Present a summary: the bug, the reproduction, the root cause, the fix, the regression test(s), and how to manually verify on this local feature branch.
   - Ask: "You are on the local feature branch `fix/{NUMBER}-...` — it has NOT been pushed and NOT merged. Please reproduce the original bug scenario and confirm it's fixed. When done, confirm:
     (a) Bug is fixed — merge this branch into `{DEV_BRANCH}` locally and push `{DEV_BRANCH}`
     (b) Bug still reproduces (or a new issue surfaced) — describe it and I'll go back to Phase 4 to fix
     (c) Abort — leave the local feature branch as-is for later (nothing is pushed, nothing is merged)"
   - If (b): return to Phase 4 with the user's feedback. The same 3-iteration limit from the Phase 4/5 cycle applies.
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
   - `git merge --no-ff fix/{NUMBER}-{kebab-case-short-desc} -m "merge fix: {PRIORITY}/{NUMBER} - {SHORT_DESC}"`
   - `--no-ff` preserves the bug-fix branch as a logical unit in history.
   - If the merge conflicts, stop and walk the user through resolution before continuing.

7. **Push `{DEV_BRANCH}` (never the feature branch):**
   - `git push origin {DEV_BRANCH}`
   - **Never** `git push` the feature branch. Never use `--force`.
   - If the push is rejected because the remote moved ahead, run `git pull --rebase origin {DEV_BRANCH}` and retry. If the rebase conflicts, stop and ask the user.

8. **Delete the feature branch locally** (it was never pushed, so there is no remote branch to delete):
   - `git branch -d fix/{NUMBER}-{kebab-case-short-desc}`
   - If git refuses with "not fully merged" (shouldn't happen after step 6), stop and ask the user — do **not** use `-D` without explicit approval.

9. Report to the user:
    - `{DEV_BRANCH}` HEAD commit SHA and URL
    - Confirmation that the local feature branch has been deleted
    - One-line summary of the root cause and the fix
    - Reminder: "`todos/` stayed local; only the code merge is on the remote."

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations. In particular:
- If Phase 1 cannot reproduce the bug, **do not proceed** — the pipeline halts there until reproduction exists or the user explicitly aborts.
- If Phase 4 cannot make the failing test pass, **do not weaken the test** to make it pass; re-diagnose in Phase 2.

## Notes

- Bug fixes use the `fix/` branch prefix; `feat/`, `refactor/`, `chore/`, `test/` belong to `/solve-todo`.
- **`todos/` is local-only.** Gitignored in Phase 0 step 5; never staged, committed, or pushed. Status transitions (BACKLOG → DOING → DONE) happen via plain `mv` + in-place edits to the priority file.
- **Feature branches are local-only.** The `fix/{NUMBER}-...` branch is never pushed. Phase 7 merges it into `{DEV_BRANCH}` locally and deletes it with `git branch -d`.
- **No pull requests.** The pipeline does not run `gh pr create`. Validation is a local test gate in Phase 7, and integration is a local `--no-ff` merge into `{DEV_BRANCH}`.
- `git push` is **only** allowed for `{DEV_BRANCH}`, and **never** with `--force`.
- The regression test is the contract: it proves the bug existed, proves the fix works, and prevents the bug from returning. Every bug fix ships with one.
