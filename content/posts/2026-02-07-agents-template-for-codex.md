---
title:  "A practical template repo for Codex instructions"
slug: agents-template-for-codex
date:   2026-02-07 11:50:00
draft: false
categories: ["ai", "codex", "workflow"]
---
When you start using AI coding agents across multiple repositories, a giant one-off `AGENTS.md` turns into a maintenance problem.

I wanted a repeatable way to bootstrap projects with the same planning and execution standards, while still keeping app-specific details local to each repo.

That is why I put together [`github.com/pmenglund/agents`](https://github.com/pmenglund/agents).

The repo is a template for a modular instruction system:

- `_AGENTS.md` as a thin entrypoint (renamed to `AGENTS.md` in consuming repos)
- `PLANS.md` as the ExecPlan contract for non-trivial work
- `APP.md` for architecture and repo-specific constraints
- `languages/` for language-specific rules (for example Go or Python)
- `workflows/` for tracker/process variants (Linear, GitHub, Markdown, Beads)

I also like the `_AGENTS.md` detail: in the template repo itself, the leading underscore prevents automatic instruction loading, so you can edit the template safely. In a target repo, you rename it to `AGENTS.md` to enable it.

The default precedence model is explicit:

1. active ExecPlan
2. `APP.md`
3. `LANGUAGE.md`
4. `WORKFLOW.md`
5. `AGENTS.md`

That ordering keeps project-specific decisions on top while still preserving shared defaults.

The other key idea is treating ExecPlans as first-class artifacts, not optional notes. `PLANS.md` defines what must be included (progress tracking, decisions, discoveries, outcomes), and the workflow templates map plan steps to tracker items.

For me, this has made agent runs more predictable: fewer “creative” detours, clearer review context, and a better audit trail for why decisions were made.

If you want a starter layout, the recommended structure is:

```text
AGENTS.md
APP.md
PLANS.md
LANGUAGE.md
WORKFLOW.md
plans/
```

If you do not want to set this up manually, there is also a skill in [`github.com/pmenglund/skills`](https://github.com/pmenglund/skills) called `agents-md` that bootstraps these files for you.

A practical example in Codex is:

```text
$skill-installer install from repo pmenglund/skills the skill agents-md
$agents-md set up /path/to/repo using language go and workflow linear
```

Under the hood, that flow runs the `agents-md` setup script to infer placeholder values first, lets you confirm them, and then writes `AGENTS.md`, `APP.md`, `PLANS.md`, `LANGUAGE.md`, `WORKFLOW.md`, and `plans/`.

From there, customize `APP.md`, pick a language file, pick a workflow file, and keep `AGENTS.md` + `PLANS.md` mostly stable across repos.
