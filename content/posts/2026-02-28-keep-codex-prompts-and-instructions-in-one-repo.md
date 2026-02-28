---
title:  "How I use /reflect and /retrospective with Codex"
slug: keep-codex-prompts-and-instructions-in-one-repo
date:   2026-02-27 20:20:00
draft: false
categories: ["ai", "codex", "workflow"]
---
I keep two custom prompts in [`github.com/pmenglund/codex`](https://github.com/pmenglund/codex): `/reflect` and `/retrospective`.

The point is simple: stop tuning Codex instructions from memory and tune them from evidence.

Right before I close a Codex session, I run `/reflect`.

It saves structured JSON under `~/.codex/reflection/YYYY/MM/DD/` and captures what worked, what failed, root causes, and proposed fixes.

I like that format because it is machine-readable and consistent, so later analysis is fast and does not depend on how tired I was when I wrote notes.

Once I have enough sessions, I run `/retrospective` (usually weekly).

That prompt reads the saved reflections for a time window and highlights recurring instruction debt instead of one-off noise.

In other words, it helps me separate "bad day" problems from "bad default" problems.

Then I apply changes in three places:

- `AGENTS.md` when the fix is a durable guardrail or workflow rule
- saved prompts under `~/.codex/prompts` when the fix is reusable behavior
- hand-written prompts in the current session when the fix is specific to the task

A useful rule of thumb is to promote fixes upward only when they repeat.

If an issue happens once in one repo, I keep it local. If it repeats across repos, I move it into global guidance.

This split matters. If I put everything in `AGENTS.md`, it gets bloated. If I keep everything as ad hoc prompts, I repeat myself.

I also keep these prompts repo neutral on purpose.

`/reflect` and `/retrospective` should work the same whether I am in a Go service, a static site, or an infra repo.

That means no hardcoded project commands, no repo-specific paths, and no assumptions about one codebase.

Repo-specific behavior belongs in each repo's `AGENTS.md` or in the hand-written prompt for that session.

Keeping the saved prompts neutral makes them portable, easier to trust, and easier to reuse everywhere.

I treat `~/.codex/AGENTS.md` and repo `AGENTS.md` as different layers.

The one in `~/.codex/AGENTS.md` is for global defaults I want in every repo: behavior rules, safety rails, and generic PR conventions.

The repo `AGENTS.md` is for local reality: architecture constraints, required generators, validation commands, and team-specific workflow details.

So global instructions keep me consistent, and repo instructions keep me correct in the current codebase.

This also lines up with the template approach I described in my earlier post about [`github.com/pmenglund/agents`](https://github.com/pmenglund/agents).

I use that template to bootstrap per-repo files like `APP.md`, `LANGUAGE.md`, `WORKFLOW.md`, and a repo `AGENTS.md`, then let `~/.codex/AGENTS.md` provide cross-repo defaults above that.

In practice: global rules in `~/.codex/AGENTS.md`, repo contract in template-generated files, and prompt-level tuning from `/reflect` + `/retrospective`.

That layering also keeps each file focused: global files stay generic, repo files stay operational, and prompts stay analytical.

The two prompts give me a feedback loop:

1. run a session
2. run `/reflect`
3. run `/retrospective` over several reflections
4. update `AGENTS.md`, saved prompts, or manual prompts based on the pattern

Then I run normal work again and repeat the loop.

If you want to copy this, the minimum structure is:

```text
codex/
  AGENTS.md
  prompts/
    reflect.md
    retrospective.md
```

For me this has made Codex sessions less ad hoc, easier to debug, and easier to improve week over week.
