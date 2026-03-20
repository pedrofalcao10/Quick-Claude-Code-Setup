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
