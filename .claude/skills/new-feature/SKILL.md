# new-feature

Creates a new backlog item from a human-originated idea, problem, or necessity. Runs an interactive brainstorm, generates a todo file + priority entry + GitHub issue, then hands off to `/solve-todo`.

## Usage

```
/new-feature <description>
/new-feature
```

- Pass a description of the idea, problem, or necessity (e.g., `/new-feature add WhatsApp notifications for low NPS scores`).
- With no argument: the skill will ask you to describe what you want to build.

## Permission Rules (apply to ALL phases)

- **ALLOW without asking:** All file reads, grep/search, glob, git status/log/diff/branch, running tests, build commands, lint commands
- **PAUSE and ask user approval for:** Any file/directory creation, file edit, file deletion, git commits, git push, git checkout (branch switches), GitHub mutations (`gh issue create`, `gh pr create`)

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

5. **Check for partial prior run.** Scan `todos/backlog/` for uncommitted files and `docs/brainstorms/` for recent (today) brainstorm docs matching the user's description topic. If found, ask: "A previous `/new-feature` run may have partially completed. Resume from existing artifacts, or start fresh (deletes partial artifacts)?"

6. **Scan for overlaps.** Read the `# Title` line from each todo file in `todos/backlog/`, `todos/doing/`, and `todos/done/`. Compare against the user's description: extract significant nouns and verbs (ignoring stop words like "the", "a", "add", "fix"), and report any todo where 2 or more significant words match. If potential overlaps are found:
   - Report them: "These existing items look related: #{NNN} - {title}, #{NNN} - {title}"
   - Ask: "Continue with a new feature, or work on one of these existing items instead?"
   - If the user picks an existing item, invoke `/solve-todo {NUMBER}` and stop.

### Phase 1 — Brainstorm

Run: `/ce:brainstorm`

Pass the user's description as input. The brainstorm is interactive — the user participates in shaping requirements, challenging assumptions, and defining scope.

The brainstorm produces a requirements document at `docs/brainstorms/YYYY-MM-DD-{topic}-requirements.md`.

**Exit path:** If the brainstorm concludes the idea should not be built (already exists, wrong approach, not worth the effort, conflicts with architecture), stop cleanly:
- Report the brainstorm's conclusion and reasoning.
- Do not create any todo file, priority entry, or GitHub issue.
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
- **Issue:** {`#{ISSUE_NUMBER}` — filled after Step 4}

## Problem

{The problem, idea, or necessity — derived from the brainstorm's Problem Frame section.}

## Fix

{The proposed approach — derived from the brainstorm's requirements and chosen approach.}
```

**Step 4: Create GitHub issue**

- Ensure labels exist: `gh label create "priority:{pN}" --force 2>/dev/null; gh label create "feature" --force 2>/dev/null`
- Title: `{PRIORITY}/{NNN} - {SHORT_DESC}`
- Body: The full content of the todo `.md` file
- Labels: `priority:{pN}`, `feature`
- Command: `gh issue create --title "..." --body "..." --label "..."`
- Capture the issue number.
- **Update the backlog file:** Replace `**Issue:** {placeholder}` with `**Issue:** #{ISSUE_NUMBER}` in the backlog `.md` file created in Step 3. This allows `/solve-todo` to find the existing issue directly instead of searching.

**Pause and show the user the backlog entry and the priority document updates for approval.**

### Phase 3 — Commit

1. Stage the specific files created/modified: the backlog entry, the updated priority document, and the brainstorm requirements document.
2. Ensure `{DEV_BRANCH}` branch is up to date: `git checkout {DEV_BRANCH} && git pull origin {DEV_BRANCH}`
3. If the work was done on a different branch, cherry-pick or merge the changes into `{DEV_BRANCH}`.
4. Commit on `{DEV_BRANCH}`: `git commit -m "feat: add backlog item #{NNN} - {short desc}"`
5. Switch back to `{DEV_BRANCH}` (stay on `{DEV_BRANCH}` for the handoff to `/solve-todo`).

### Phase 4 — Hand Off

1. Report summary:
   - Todo: `#{NNN} - {title}` (Priority: {P}, Effort: {effort})
   - Issue: `#{ISSUE_NUMBER}`
   - Brainstorm: `docs/brainstorms/{file}`
2. Ask the user: **"Ready to start implementing? This will run `/solve-todo {NNN}`."**
   - If yes: invoke `/solve-todo {NNN}`
   - If no: report "Run `/solve-todo {NNN}` when ready." and stop.

## Error Handling

If any phase fails, report the error clearly and ask the user how to proceed. Never skip phases or auto-retry destructive operations.
