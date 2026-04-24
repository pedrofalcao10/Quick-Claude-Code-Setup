# Claude Code Skills

Four skills that form a complete engineering workflow: from identifying problems to shipping fixes.

## Operating Methodology — Deploy-Driven

All three skills operate under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — flag systemic bottlenecks early, not symptoms.
- First-principles engineering — strip problems to core components; reject over-engineered traps.
- Minimal Viable Intelligence — research/brainstorm only until the decision is obvious.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — every finding, todo, commit, and PR states the user/stakeholder outcome.
- Documentation as code — artifacts must let any teammate resume without a meeting.
- Radical Transparency — phase summaries use 🟢/🟡/🔴 status with blockers + next action. No fluff.

**Deploy-Driven (Accelerated Execution)**
- Continuous Value Delivery — target ≤48-hour shippable increments; split Large items on sight.
- Automated Governance — lean on lint/tests/CI/labels over manual checklists; never `--no-verify`, never `--force` push.
- 80/20 Deployment — ship the 20% that removes 80% of the risk first; defer the rest to follow-up todos.

**Value Matrix:**

| Dimension | Traditional | Optimized | Value |
|---|---|---|---|
| Problem solving | Deep research then plan | Prototype in public | Faster feedback |
| Project mgmt | Timelines & gantts | Remove friction/blockers | Team velocity |
| Execution | Feature completeness | Deployment frequency | Live user data |
| Strategy | Long-term roadmap | Dynamic pivot capability | Resilience |

If a phase output can't be explained to a stakeholder in 30 seconds, it isn't ready — rework before moving on.

## Overview

```
/review-and-plan  -->  finds code issues  -->  creates MANY todos  -->  /solve-todo next
/new-feature      -->  brainstorms idea   -->  creates ONE todo    -->  /solve-todo {NNN}
/bug-fix          -->  reproduce -> root cause -> fix + regression test -> local merge
/solve-todo       -->  resolves any todo through analysis -> plan -> implement -> review -> merge
```

## Skills

### /review-and-plan

**Purpose:** Reviews the codebase for issues, creates the `todos/` backlog architecture, generates todo files and a priority document, then hands off to `/solve-todo`.

**When to use:** Starting a new project, beginning a review cycle, or auditing code for the first time.

**Usage:**
```
/review-and-plan                    # review entire codebase
/review-and-plan backend/           # review a specific path
```

**What it does:**
1. Auto-detects the development branch (`dev`, `develop`, `staging`, etc.) or asks the user
2. Creates `todos/` directory structure (backlog, doing, done, priority)
3. Runs a comprehensive code review (`/ce:review`)
4. Generates individual todo files in `todos/backlog/`
5. Generates a priority document with execution order and Quick Reference table
6. Commits on the development branch and offers to start `/solve-todo next`

**Handles re-runs:** Detects existing findings and offers to keep, overwrite, or abort. Numbering never collides across review cycles.

---

### /new-feature

**Purpose:** Takes a human-originated idea, problem, or necessity, runs an interactive brainstorm, creates a backlog item, and hands off to `/solve-todo`.

**When to use:** You have a new feature idea, a user-reported problem, or a business requirement that doesn't come from code review.

**Usage:**
```
/new-feature add WhatsApp notifications for low NPS scores
/new-feature                        # will ask for description
```

**What it does:**
1. Validates prerequisites (requires `todos/` architecture and a priority document to exist)
2. Scans for overlapping existing backlog items
3. Runs `/ce:brainstorm` interactively to shape the idea into concrete requirements
4. Creates a todo file with `Source: Feature`, a link to the brainstorm doc, and an `Issue` field
5. Appends the new item to the latest existing priority document (Execution Order + Quick Reference)
6. Creates a GitHub issue with `feature` label and writes the issue number back into the todo file
7. Commits on the development branch (auto-detected or user-specified) and offers to start `/solve-todo {NNN}`

**Exit path:** If the brainstorm concludes the idea shouldn't be built, stops cleanly with no artifacts created.

**Prerequisite:** Run `/review-and-plan` at least once first to set up the `todos/` architecture.

---

### /bug-fix

**Purpose:** Resolves a specific, reported bug through a strict **reproduce → root-cause → fix-with-regression-test** pipeline. Uses the `fix/` branch prefix and mirrors the local-only workflow used by `/solve-todo`.

**When to use:** You have a concrete bug report (user complaint, error log, broken behavior) and need to fix it — not exploring an idea, not doing a broad review.

**Usage:**
```
/bug-fix PDF export crashes on orders with >100 line items
/bug-fix #42                        # start from an existing GitHub issue
/bug-fix                            # will ask for a description
```

**Explicit rules (non-negotiable):**
1. No fix without reproduction (Phase 1 must produce a deterministically failing test).
2. No fix without root cause (Phase 2 must name the specific file/function/input that breaks).
3. Every fix ships with a regression test that fails before and passes after.
4. Smallest possible diff — no drive-by refactors; follow-up issues get their own todos.

**What it does:**
1. Pre-flight (working tree, dev branch, gitignore `todos/`, duplicate scan, create GitHub issue with `bug` label, create local-only `fix/{NNN}-...` branch, move todo to `doing/`)
2. **Reproduce** — write a failing regression test (halts the pipeline if reproduction fails)
3. **Root Cause** — `/ce:brainstorm` narrowly focused on why the test fails
4. **Plan** — `/ce:plan` with the smallest possible diff; explicit "will NOT change" list
5. **Implement** — `/ce:work` to make the failing test pass; add edge-case tests
6. **Review** — `/ce:review` focused on regression surface, edge cases, and scope creep
7. **Documentation** — `/ce:compound` for non-trivial root causes (skipped for trivial)
8. **User Testing, Local Merge & Finalize** — mandatory test gate on the local branch, then `--no-ff` merge into `{DEV_BRANCH}`, push dev, delete local `fix/` branch, close issue

---

### /solve-todo

**Purpose:** Takes a single backlog item through the full implementation pipeline: analysis, planning, implementation, review, and merge into `{DEV_BRANCH}`.

**When to use:** Resolving any item from the backlog, whether it came from `/review-and-plan` or `/new-feature`.

**Usage:**
```
/solve-todo 002                     # work on a specific todo
/solve-todo next                    # pick the next item from priority order
```

**What it does:**
1. Pre-flight checks (input validation, clean working tree, existing branch/issue, dependencies)
2. Creates a GitHub issue and feature branch from the development branch
3. Moves todo to `todos/doing/` (status: DOING)
4. **Analysis** — reads brainstorm doc if linked (from `/new-feature`), otherwise runs `/ce:ideate` + `/ce:brainstorm`
5. **Plan** — runs `/ce:plan` for implementation strategy
6. **Implementation** — runs `/ce:work` to write code
7. **Review** — runs `/ce:review` (max 3 fix iterations)
8. **Documentation** — runs `/ce:compound` (skipped for small items)
9. Moves todo to `todos/done/`, creates PR targeting the development branch, closes the GitHub issue

**Branch prefixes:** Automatically chosen based on todo source:
- `fix/` — security or bug fixes
- `refactor/` — architecture changes
- `test/` — test coverage
- `chore/` — cleanup
- `feat/` — new features (Source: Feature)

---

## Typical Workflows

### Starting fresh on a new codebase

```
/review-and-plan              # generates 28 todos + priority doc
/solve-todo next              # starts fixing the first item
/solve-todo next              # continues to the next item
...
```

### Adding a new feature

```
/new-feature add email digest for weekly feedback summaries
# interactive brainstorm -> todo created -> /solve-todo runs automatically
```

### Working through the backlog

```
/solve-todo next              # picks the highest-priority BACKLOG item
/solve-todo 015               # or pick a specific one by number
```

### New review cycle (all previous items done)

```
/review-and-plan              # creates 001-priority-todos.md (next cycle)
                              # todo numbering continues from where the last cycle ended
```

---

## Shared Conventions

| Convention | Format | Example |
|-----------|--------|---------|
| Todo file | `{NNN}-{pN}-{kebab-desc}.md` | `029-p2-whatsapp-notifications.md` |
| Priority doc | `{NNN}-priority-todos.md` | `000-priority-todos.md` |
| GitHub issue | `{PRIORITY}/{NNN} - {desc}` | `P2/029 - WhatsApp notifications` |
| Branch | `{prefix}/{NNN}-{kebab-desc}` | `feat/029-whatsapp-notifications` |
| PR title | `{prefix}: {PRIORITY}/{NNN} - {desc}` | `feat: P2/029 - WhatsApp notifications` |

## Permissions (all skills)

- **Freely allowed:** File reads, code search, tests, builds, lints, git read operations
- **Requires approval:** File writes, git commits/push/checkout, GitHub issue/PR creation

## Directory Structure

```
todos/
  backlog/    # Items not yet started (BACKLOG)
  doing/      # Items in progress (DOING)
  done/       # Completed items (DONE)
  priority/   # Priority documents, one per review cycle
docs/
  brainstorms/  # Requirements docs from /ce:brainstorm (used by /new-feature)
  plans/        # Implementation plans from /ce:plan
```
