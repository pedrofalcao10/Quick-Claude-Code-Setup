# Guide to Compound Engineering Plugin (Optional)

> This is an optional enhancement for your Claude Code setup.
> The [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by Every Inc. adds a structured workflow to Claude Code that makes each unit of engineering work compound into the next — through better planning, review, and knowledge retention.

---

## Install

In any Claude Code session, run:

```
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

That's it. The skills become available immediately as `/ce:*` commands.

---

## Operating Methodology — Deploy-Driven

These commands are most effective when run under the **Value-First Mantra**: *"Does this action put a working solution in front of a user today? If not, how can I simplify it until it does?"*

**Clever (Strategic Sharpness)**
- Rapid pattern recognition — use `/ce:ideate` and `/ce:review` to surface systemic bottlenecks, not symptoms.
- First-principles engineering — in `/ce:brainstorm` and `/ce:plan`, strip problems to core components; reject over-engineered traps.
- Minimal Viable Intelligence — stop ideating/brainstorming the moment the decision is obvious.

**Clear (High-Fidelity Communication)**
- Technical-to-value translation — every brainstorm, plan, and compound doc states the user/stakeholder outcome.
- Documentation as code — `/ce:compound` artifacts must let a teammate resume without a meeting.
- Radical Transparency — report each command's output with 🟢/🟡/🔴 status + blockers + next action.

**Deploy-Driven (Accelerated Execution)**
- Continuous Value Delivery — prefer many ≤48-hour `/ce:work` increments over one long-running effort.
- Automated Governance — let `/ce:review` + CI do the enforcement; never `--no-verify`, never `--force` push.
- 80/20 Deployment — ship the 20% of the plan that removes 80% of the risk first; defer the rest.

**Value Matrix:**

| Dimension | Traditional | Optimized | Value |
|---|---|---|---|
| Problem solving | Deep research then plan | Prototype in public | Faster feedback |
| Project mgmt | Timelines & gantts | Remove friction/blockers | Team velocity |
| Execution | Feature completeness | Deployment frequency | Live user data |
| Strategy | Long-term roadmap | Dynamic pivot capability | Resilience |

---

## The Workflow

The plugin structures work around a repeating cycle:

```
Ideate → Brainstorm → Plan → Work → Review → Compound → repeat
```

You don't have to use every step. Start with the ones that fit your task.

---

## Commands

### `/ce:ideate`
Generates and filters high-impact improvement ideas for your current project.
Use it when you're not sure what to work on next or want the AI to surface strong directions before you commit to one.

### `/ce:brainstorm`
Opens a collaborative dialogue to explore requirements and approaches before writing any plan.
Use it for vague or ambitious features, or when you want to think through options before deciding what to build.

### `/ce:plan`
Converts a feature description or brainstorm output into a structured implementation plan.
Use it before writing code on anything non-trivial.

### `/ce:work`
Executes a plan using worktrees and task tracking.
Use it to start implementing once you have a plan and want Claude to work through it step by step.

### `/ce:review`
Runs a multi-agent code review before merging.
Use it after implementation to catch issues across architecture, security, performance, and style.

### `/ce:compound`
Documents what was learned from a solved problem so future work can build on it.
Use it after closing a task or bug — especially one that took real effort to figure out.

---

## Recommended starting point

If you're new to this plugin, start with just two commands:

1. `/ce:brainstorm` — before starting any feature
2. `/ce:compound` — after finishing it

These two alone significantly improve the quality and reusability of your work over time.

---

## Custom Skills (built on top of this plugin)

This repo includes three custom skills in `.claude/skills/` that chain the `/ce:*` commands above into end-to-end workflows:

| Skill | What it does |
|-------|-------------|
| `/review-and-plan` | Reviews codebase → creates `todos/` backlog → generates priority doc → hands off to `/solve-todo` |
| `/new-feature` | Interactive brainstorm → creates one backlog item + GitHub issue → hands off to `/solve-todo` |
| `/solve-todo` | Picks a todo → analysis → plan → implement → review → PR (full pipeline) |

See [`custom-skills.md`](custom-skills.md) for full documentation, or browse the skill definitions directly in [`.claude/skills/`](.claude/skills/).
