# Custom Claude Code Skills

Three slash-command skills that form a complete engineering workflow: from identifying problems to shipping fixes.

> **Prerequisite:** These skills depend on the [compound-engineering plugin](https://github.com/anthropics/claude-code-plugins). They call `/ce:review`, `/ce:brainstorm`, `/ce:ideate`, `/ce:plan`, `/ce:work`, and `/ce:compound` internally. Without the plugin installed, these skills will not function.

---

## Operating Methodology — Deploy-Driven

All three skills operate under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — flag systemic bottlenecks early, not symptoms.
- First-principles engineering — strip problems to core components; reject over-engineered traps.
- Minimal Viable Intelligence — research/brainstorm only until the decision is obvious.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — every finding, todo, commit, and PR states the user/stakeholder outcome, not just the diff.
- Documentation as code — artifacts must let any teammate resume without a meeting.
- Radical Transparency — phase summaries end with 🟢/🟡/🔴 status + blockers + next action. No fluff.

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

Pipeline tripwire: if a phase output can't be explained to a stakeholder in 30 seconds, it isn't ready — rework before moving on.

---

## Quick Overview

```
/review-and-plan  →  finds code issues  →  creates MANY todos  →  /solve-todo next
/new-feature      →  brainstorms idea   →  creates ONE todo    →  /solve-todo {NNN}
/solve-todo       →  resolves any todo through analysis → plan → implement → review → PR
```

---

## /review-and-plan

**What it does:** Reviews your codebase for issues (security, performance, architecture, code quality), creates a `todos/` backlog system, generates individual todo files for each finding, and produces a priority document with execution order.

**When to use:** Starting work on a new project, beginning a review cycle, or auditing code for the first time.

**Usage:**
```
/review-and-plan                    # review entire codebase
/review-and-plan backend/           # review a specific path
```

**Workflow:**
1. **Pre-flight** — checks for uncommitted changes, detects existing todos, handles re-runs (keep/overwrite/abort)
2. **Architecture** — creates `todos/backlog/`, `todos/doing/`, `todos/done/`, `todos/priority/` if missing
3. **Code Review** — runs `/ce:review` (multi-agent analysis), presents findings for your approval
4. **Todo Generation** — creates one `.md` file per finding in `todos/backlog/`
5. **Priority Document** — generates execution order + quick reference table in `todos/priority/`
6. **Commit & Hand Off** — commits everything, offers to start `/solve-todo next`

**Output:** A fully populated `todos/` directory with prioritized, actionable backlog items ready for `/solve-todo`.

---

## /new-feature

**What it does:** Takes a human-originated idea, problem, or business need, runs an interactive brainstorm session to shape it into concrete requirements, then creates a single backlog item with a linked requirements document.

**When to use:** You have a new feature idea, a user-reported problem, or a business requirement that doesn't come from code review.

**Usage:**
```
/new-feature add WhatsApp notifications for low NPS scores
/new-feature                        # will ask for description
```

**Workflow:**
1. **Pre-flight** — checks for uncommitted changes, validates `todos/` architecture exists (run `/review-and-plan` first), scans for overlapping existing items
2. **Brainstorm** — runs `/ce:brainstorm` interactively with you to shape the idea into requirements; produces a doc in `docs/brainstorms/`
3. **Structure** — creates a todo file in `todos/backlog/`, updates the priority document, creates a GitHub issue with `feature` label
4. **Commit** — stages and commits all artifacts on the development branch (auto-detected or user-specified during pre-flight)
5. **Hand Off** — offers to start `/solve-todo {NNN}` immediately

**Exit path:** If the brainstorm concludes the idea shouldn't be built, it stops cleanly with no artifacts created.

**Prerequisite:** Requires `todos/` architecture to exist. Run `/review-and-plan` at least once first.

---

## /solve-todo

**What it does:** Takes a single backlog item through the full implementation pipeline — from analysis to a merged PR. This is where code actually gets written.

**When to use:** Resolving any item from the backlog, whether it came from `/review-and-plan` or `/new-feature`.

**Usage:**
```
/solve-todo 002                     # work on a specific todo
/solve-todo next                    # pick the next item from priority order
```

**Workflow:**
1. **Setup** — validates the todo exists, checks for existing branches/issues, detects the development branch, creates a GitHub issue, branches from the development branch, moves todo to `todos/doing/`
2. **Analysis** — reads linked brainstorm doc (if from `/new-feature`) or runs `/ce:ideate` + `/ce:brainstorm` to understand the problem
3. **Plan** — runs `/ce:plan` to create a concrete implementation plan with specific files and test strategy
4. **Implementation** — runs `/ce:work` to write the code changes
5. **Review** — runs `/ce:review` to check for security, performance, quality issues (up to 3 fix iterations)
6. **Documentation** — runs `/ce:compound` to document learnings (skipped for small items)
7. **Finalize** — moves todo to `todos/done/`, pushes branch, creates PR targeting the development branch, closes the GitHub issue

**Branch naming:** Automatically chosen based on the todo's nature:
- `fix/` — security or bug fixes
- `refactor/` — architecture changes
- `feat/` — new features
- `test/` — test coverage
- `chore/` — cleanup

**Each phase pauses for your approval** before proceeding to the next.

---

## Typical Workflows

### Starting fresh on a new codebase
```
/review-and-plan              # generates todos + priority doc
/solve-todo next              # starts fixing the first item
/solve-todo next              # continues to the next
```

### Adding a new feature
```
/new-feature add email digest for weekly feedback summaries
# interactive brainstorm → todo created → /solve-todo runs automatically
```

### Working through the backlog
```
/solve-todo next              # picks the highest-priority BACKLOG item
/solve-todo 015               # or pick a specific one by number
```

---

## Conventions

| Convention | Format | Example |
|-----------|--------|---------|
| Todo file | `{NNN}-{pN}-{kebab-desc}.md` | `029-p2-whatsapp-notifications.md` |
| Priority doc | `{NNN}-priority-todos.md` | `000-priority-todos.md` |
| GitHub issue | `{PRIORITY}/{NNN} - {desc}` | `P2/029 - WhatsApp notifications` |
| Branch | `{prefix}/{NNN}-{kebab-desc}` | `feat/029-whatsapp-notifications` |
| PR title | `{prefix}: {PRIORITY}/{NNN} - {desc}` | `feat: P2/029 - WhatsApp notifications` |

## Directory Structure

```
todos/
  backlog/    # Items not yet started
  doing/      # Items in progress
  done/       # Completed items
  priority/   # Priority documents (one per review cycle)
docs/
  brainstorms/  # Requirements docs from /ce:brainstorm
  plans/        # Implementation plans from /ce:plan
  solutions/    # Knowledge docs from /ce:compound
```

## Permissions

- **Freely allowed:** File reads, code search, tests, builds, lints, git read operations
- **Requires approval:** File writes, git commits/push/checkout, GitHub issue/PR creation
