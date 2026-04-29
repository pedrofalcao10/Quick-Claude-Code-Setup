# new-feature

Creates a new backlog item from a human-originated idea, problem, or necessity. Runs an interactive brainstorm, generates a todo file + priority entry, then hands off to `/solve-todo`.

## Operating Methodology — Deploy-Driven

This skill operates under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — flag systemic bottlenecks and overlaps early.
- First-principles engineering — strip the idea to its core components; reject over-engineered traps.
- Minimal Viable Intelligence — brainstorm only until the decision is obvious; no analysis paralysis.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — every backlog entry must state the user/stakeholder outcome, not just the implementation.
- Documentation as code — the todo file + brainstorm doc must let any teammate resume without a meeting.
- Radical Transparency — each phase summary ends with 🟢/🟡/🔴 status + blockers + next action. No fluff.

**Deploy-Driven (Accelerated Execution)**
- Continuous Value Delivery — scope toward ≤48-hour shippable increments; split Large items on sight.
- Automated Governance — lean on labels, issue templates, and hooks over manual checklists.
- 80/20 Deployment — prefer the 20% of the feature that delivers 80% of the ROI; defer the rest to a follow-up todo.

**Value Matrix — choose the optimized column:**

| Dimension | Traditional | Optimized | Value |
|---|---|---|---|
| Problem solving | Deep research then plan | Prototype in public | Faster feedback |
| Project mgmt | Timelines & gantts | Remove friction/blockers | Team velocity |
| Execution | Feature completeness | Deployment frequency | Live user data |
| Strategy | Long-term roadmap | Dynamic pivot capability | Resilience |

If any phase output violates the mantra (too big, too vague, no clear user value), shrink it before handing off.

## Usage

```
/new-feature <description>
/new-feature
```

- Pass a description of the idea, problem, or necessity (e.g., `/new-feature add WhatsApp notifications for low NPS scores`).
- With no argument: the skill will ask you to describe what you want to build.

## Permission Rules (apply to ALL phases)

- **ALLOW without asking:** All file reads, grep/search, glob, git status/log/diff/branch, running tests, build commands, lint commands
- **PAUSE and ask user approval for:** Any file/directory creation, file edit, file deletion, git commits, git push, git checkout (branch switches)

## Workflow

### Phase 0 — Pre-flight

1. **Check working tree:** Run `git status --porcelain`. If there are uncommitted changes, warn the user and ask whether to stash, commit, or abort before continuing.

2. **Validate input:** If no description was provided, ask the user: "What would you like to build? Describe the problem, idea, or necessity."

3. **Determine development branch (`{DEV_BRANCH}`):**
   - Auto-detect: Run `git branch -a` and check for branches matching common integration branch names: `dev`, `develop`, `development`, `staging`, `next`.
   - If exactly one match is found: announce "Detected development branch: `{name}`." Use it as `{DEV_BRANCH}`.
   - If multiple matches are found: ask the user which one to use.
   - If no matches are found: ask the user: "What is your development/integration branch name?"
   - Validate the branch exists locally or on remote. If it doesn't exist, ask: "Branch `{name}` doesn't exist yet. Create it from the current branch, or enter a different name?"
   - Use `{DEV_BRANCH}` for all subsequent references to the integration branch in this pipeline.

4. **Validate prerequisites.** Check that `todos/backlog/`, `todos/doing/`, `todos/done/`, `todos/priority/` all exist AND that at least one priority file exists in `todos/priority/`. If missing, tell the user: "The todos/ architecture or priority document is missing. Run `/review-and-plan` first to set up the backlog structure, or confirm if you'd like me to create the directories now." Let the user decide.

5. **Ensure `todos/` is local-only (gitignored).** The entire `todos/` tree is workflow state that must never leave the local machine.
   - Check `.gitignore` for a line matching `todos/` (or `/todos/`). If missing, ask: "Add `todos/` to `.gitignore` so backlog/priority files stay local only?" If yes, append `todos/` to `.gitignore`.
   - Check tracked files: `git ls-files todos/`. If any are listed, ask: "These `todos/` files are currently tracked in git. Untrack them so they become local-only?" If yes: `git rm -r --cached todos/` and commit with `chore: untrack todos/ (local-only workflow state)`.
   - From this point forward, **never** run `git add todos/...`, `git mv todos/...`, or commit anything under `todos/`.

6. **Check for partial prior run.** Scan `todos/backlog/` for leftover files and `docs/brainstorms/` for recent (today) brainstorm docs matching the user's description topic. If found, ask: "A previous `/new-feature` run may have partially completed. Resume from existing artifacts, or start fresh (deletes partial artifacts)?"

7. **Scan for overlaps.** Read the `# Title` line from each todo file in `todos/backlog/`, `todos/doing/`, and `todos/done/`. Compare against the user's description: extract significant nouns and verbs (ignoring stop words like "the", "a", "add", "fix"), and report any todo where 2 or more significant words match. If potential overlaps are found:
   - Report them: "These existing items look related: #{NNN} - {title}, #{NNN} - {title}"
   - Ask: "Continue with a new feature, or work on one of these existing items instead?"
   - If the user picks an existing item, invoke `/solve-todo {NUMBER}` and stop.

### Phase 1 — Brainstorm

Run: `/ce:brainstorm`

Pass the user's description as input. The brainstorm is interactive — the user participates in shaping requirements, challenging assumptions, and defining scope.

The brainstorm produces a requirements document at `docs/brainstorms/YYYY-MM-DD-{topic}-requirements.md`.

**Exit path:** If the brainstorm concludes the idea should not be built (already exists, wrong approach, not worth the effort, conflicts with architecture), stop cleanly:
- Report the brainstorm's conclusion and reasoning.
- Do not create any todo file or priority entry.
- Say: "Brainstorm concluded this should not be built. No artifacts created."
- Stop.

**Pause and ask the user to approve the brainstorm output before continuing.**

### Phase 2 — Structure

Convert the brainstorm output into project artifacts.

**Step 1: Determine metadata**

- **Number:** Scan `todos/backlog/`, `todos/doing/`, and `todos/done/` for the highest existing `{NNN}` across all `.md` files. Use `highest + 1`. If no files exist, start from `001`.
- **Priority:** Propose P1/P2/P3 based on the brainstorm's urgency and impact analysis. Ask the user to confirm or override.
- **Effort:** Propose Small/Medium/Large based on the brainstorm's scope. Ask the user to confirm or override. Use exactly one tier — no hybrid values.
- **Short description:** Derive a 3-6 word kebab-case description from the brainstorm title.
- **Affected files:** List files/modules that will likely need modification based on the brainstorm analysis. Use `TBD` if the feature is entirely new with no existing touch points.

**Step 2: Append to the latest priority file**

Find the latest (highest-numbered) priority file in `todos/priority/`. Append the new feature entry to it — do NOT create a separate priority file.

- **Recommended Execution Order:** If a `### New Features` section exists at the end of the file, add a row to its table. Otherwise, create a new `### New Features` section after the last existing section, with the standard table header and one row. Use `Order = max(existing Order across all tables) + 1`.
- **Quick Reference:** Add a new row at the bottom with `Order = max(existing Order) + 1` and columns matching the existing schema. Set Dependencies to `None` unless the brainstorm identified prerequisites.

The appended entry in **Recommended Execution Order** should include:

| Order | # | Todo | Priority | Effort | Dependencies | Rationale |
|-------|---|------|----------|--------|--------------|-----------|
| {next order} | {NNN} | {short title} | {P1/P2/P3} | {Small/Medium/Large} | {None or #dep} | {1-line rationale from brainstorm} |

The appended entry in **Quick Reference** should include:

| Order | # | Todo | Priority | Effort | Dependencies | Status |
|-------|---|------|----------|--------|--------------|--------|
| {next order} | {NNN} | {short title} | {P1/P2/P3} | {effort} | {None or #dep} | BACKLOG |

Effort tiers (use exactly one — no hybrid values):
- **Small:** ~5 min to ~1 hr
- **Medium:** ~2-4 hrs
- **Large:** ~6-8+ hrs

**Step 3: Create backlog entry**

Create a lightweight todo file in `todos/backlog/` that references the priority file. This is required for pipeline compatibility with `/solve-todo`.

File: `todos/backlog/{NNN}-{pN}-{kebab-desc}.md`

```markdown
# #{NNN} - {Title}

- **Priority:** {P1 CRITICAL | P2 IMPORTANT | P3 NICE-TO-HAVE}
- **Source:** Feature
- **Files:** {`path/to/file.ext` or `TBD`}
- **Brainstorm:** {`docs/brainstorms/YYYY-MM-DD-{topic}-requirements.md`}
- **Priority File:** {`todos/priority/{LATEST_PRIORITY_NUMBER}-priority-todos.md`}

## Problem

{The problem, idea, or necessity — derived from the brainstorm's Problem Frame section.}

## Fix

{The proposed approach — derived from the brainstorm's requirements and chosen approach.}
```

**Pause and show the user the backlog entry and the priority document updates for approval.**

### Phase 3 — Commit (brainstorm doc only)

Everything under `todos/` is **local-only** (gitignored per Phase 0 step 5). The backlog entry and priority document updates stay on disk and are **never** staged or committed. Only the brainstorm requirements doc (`docs/brainstorms/...`) is committable.

1. Ensure `{DEV_BRANCH}` branch is up to date: `git checkout {DEV_BRANCH} && git pull origin {DEV_BRANCH}`
2. Stage **only** the brainstorm requirements document: `git add docs/brainstorms/{file}`. Do **not** run `git add -A`, `git add .`, or `git add todos/...`.
3. Verify nothing under `todos/` was staged: `git diff --cached --name-only | grep -v '^todos/'` (if any staged path starts with `todos/`, unstage it before continuing).
4. Commit on `{DEV_BRANCH}`: `git commit -m "docs: add brainstorm for #{NNN} - {short desc}"`. Do not push yet — pushing happens in `/solve-todo` Phase 6 after the feature branch is merged back in.
5. Stay on `{DEV_BRANCH}` for the handoff to `/solve-todo`.

### Phase 4 — Hand Off

1. Report summary:
   - Todo: `#{NNN} - {title}` (Priority: {P}, Effort: {effort})
   - Brainstorm: `docs/brainstorms/{file}`
2. Ask the user: **"Ready to start implementing? This will run `/solve-todo {NNN}`."**
   - If yes: invoke `/solve-todo {NNN}`
   - If no: report "Run `/solve-todo {NNN}` when ready." and stop.

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations.
