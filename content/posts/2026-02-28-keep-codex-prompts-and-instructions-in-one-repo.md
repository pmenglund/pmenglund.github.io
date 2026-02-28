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

A shortened example looks like this:

```json
{
  "session_id": "abc123",
  "session_end_iso": "2026-02-27T04:11:22Z",
  "summary_one_liner": "Fixed post date and clarified publishing checklist",
  "root_causes": [
    "Post date was in the future, so Hugo excluded it",
    "No explicit pre-publish date check in repo instructions"
  ],
  "top_instruction_debts": [
    {
      "surface": "AGENTS",
      "problem": "Missing clear publish checklist",
      "best_fix": "Add a pre-publish checklist with date validation",
      "confidence": "high"
    }
  ],
  "concrete_improvements": [
    {
      "type": "guardrail",
      "surface": "AGENTS",
      "classification": "repo_specific",
      "description": "Require task build and homepage visibility check before push",
      "priority": "P0"
    }
  ],
  "metadata": {
    "persistence_path": "/Users/pme/.codex/reflection/2026/02/27/041122145Z.json",
    "persistence_status": "saved"
  }
}
```

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

And here are the actual prompt files I use today.

### `/reflect` (`prompts/reflect.md`)

````md
---
description: Produce a machine-readable end-of-session reflection (JSON) focused on reusable workflow improvements
argument-hint: [SESSION_ID=<codex-session-id>]
---

You are Codex. REVIEW the entire just-completed session (all user messages, your messages, files produced, edits, and manual steering).

Your goal is not only to summarize what happened, but to extract the smallest set of high-leverage changes that would have prevented wasted iterations next time.
Prioritize reusable instruction debt over one-off repo bugs. Whenever possible, identify whether a fix belongs in:
1. a manual prompt from the user
2. a stored prompt under `~/.codex/prompts`
3. generic instructions
4. `AGENTS.md`

Your primary job has two outputs:
1. Create and save one compact UTF-8 JSON file at an absolute path under `~/.codex/reflection/`.
2. Return that same JSON object as your final response, with no markdown and no extra text.

Identifier and path rules:
1. If `SESSION_ID` is provided as an argument, use it for the `session_id` field.
2. If `SESSION_ID` is not provided and the current Codex session exposes `CODEX_THREAD_ID`, use `CODEX_THREAD_ID` for the `session_id` field.
3. If neither `SESSION_ID` nor `CODEX_THREAD_ID` is available, generate a fallback identifier as `session-unknown-<UTC timestamp in YYYYMMDDTHHMMSSZ format>`.
4. When using the fallback identifier, set `session_id` to that fallback value, not `null`.
5. The output directory path must be UTC date-sharded as `~/.codex/reflection/YYYY/MM/DD/`.
6. The output filename must be UTC time-stamped as `HHMMSSmmmZ.json` (millisecond precision), independent of `session_id`.
7. If that filename already exists, append `-2`, `-3`, and so on before `.json` until unique.
8. The output file path must be `~/.codex/reflection/YYYY/MM/DD/HHMMSSmmmZ.json` (plus optional collision suffix when needed).

Persistence rules:
1. Before sending the final response, use an available write-capable tool to create `~/.codex/reflection/YYYY/MM/DD/` if needed and write the JSON file.
2. After writing the file, read it back and verify that it contains the same JSON value as the final response.
3. Do not emit a shell command for someone else to run.
4. Do not ask the user to save or copy anything.
5. If no write-capable tool is available, or if write verification fails, still return the JSON object only and record the problem in `root_causes` and `metadata.persistence_status`.
6. Set `metadata.persistence_path` to the actual sharded absolute file path used for persistence.

The JSON must conform to the schema below and must be compact (no pretty-printing required):

```json
{
  "session_id": "<string>",
  "session_start_iso": "<ISO-8601|null>",
  "session_end_iso": "<ISO-8601|null>",
  "summary_one_liner": "<string, <=20 words>",
  "what_worked": ["<string>", "..."],                  // 1–10 items
  "what_failed_or_was_painful": ["<string>", "..."],   // 1–10 items
  "root_causes": ["<string>", "..."],                  // 1–6 items
  "top_instruction_debts": [                           // 0–3 items; omit one-off noise
    {
      "surface": "<manual_prompt|stored_prompt|generic_instructions|AGENTS>",
      "problem": "<short recurring or high-leverage issue>",
      "why_it_matters": "<short concrete impact>",
      "best_fix": "<smallest credible change>",
      "confidence": "<high|medium|low>"
    }
  ],
  "concrete_improvements": [                           // 1–8 objects (include at least one edit of the user's original prompt when applicable)
    {
      "type": "<prompt|tool|step|test|guardrail|example|config>",
      "surface": "<manual_prompt|stored_prompt|generic_instructions|AGENTS>",
      "classification": "<instruction_gap|environment_mismatch|repo_specific|one_off>",
      "description": "<short summary>",
      "target_files": ["<absolute path>", "..."],      // 0–3 items when known
      "how_to_apply": "<copy-paste text or numbered steps>",
      "expected_value": "<what this prevents or speeds up>",
      "priority": "<P0|P1|P2>"
    }
  ],
  "example_prompts_to_use_next_time": ["<string>", "..."], // 0–3 items
  "stop_doing": ["<string>", "..."],                      // 0–6 items
  "quick_checklist_for_user": ["<string>", "..."],        // 2–6 items
  "actions_for_automation": [                             // optional
    {
      "action": "<string>",
      "triggerable_by": "<string>"
    }
  ],
  "metadata": {                                          // optional
    "files_created": ["<absolute path>", "..."],
    "languages": ["<lang>", "..."],
    "estimated_iterations_saved_next_time": <integer|null>,
    "persistence_path": "<absolute path|null>",
    "persistence_status": "<saved|write_tool_unavailable|verification_failed|null>",
    "missing_context": ["<short note>", "..."]          // optional
  }
}
```

Content constraints (ENFORCE strictly):
1. Final response must be JSON ONLY (no text before/after). Invalid output = fail.
2. Strings should be short and actionable (<=60 words unless otherwise constrained).
3. For "concrete_improvements" include at least one literal prompt-edit (exact text to paste) when applicable.
4. If you cannot determine timestamps, set the ISO fields to null.
5. If files were produced in the session, list them under metadata.files_created using absolute paths only.
6. If session was ambiguous, list ambiguous tokens/phrases under root_causes.
7. Limit arrays to the ranges specified above.
8. Set `metadata.persistence_path` to the intended absolute output path.
9. Set `metadata.persistence_status` to `saved`, `write_tool_unavailable`, or `verification_failed` as appropriate.
10. Prefer improvements that change prompts, instructions, or guardrails over improvements that only restate repo-specific implementation details.
11. Do not let a one-off environment problem dominate `top_instruction_debts` unless the real fix is a generic fallback rule.
12. Each `concrete_improvements` item must map to exactly one `surface`.
13. Use `classification=environment_mismatch` for local tooling or credential problems when the best fix is environment-specific rather than prompt-specific.
14. If important context is missing, record it in `metadata.missing_context` instead of hiding it behind nulls alone.

Execution requirements:
1. Build the JSON object first.
2. Attempt to save that JSON to `metadata.persistence_path` before the final response.
3. Read the saved file back and verify that it represents the same JSON value as the final response.
4. If persistence fails, still return valid JSON and record the exact failure cause in `root_causes`.
5. Final response must contain the JSON object only.
````

### `/retrospective` (`prompts/retrospective.md`)

````md
---
description: Review saved reflection JSON files and extract recurring instruction debt
argument-hint: [DIR=~/.codex/reflection] [DURATION="1 week"]
---

Act as a principal engineer reviewing saved Codex reflection artifacts for trends, not just one-off session notes.

Your job is to review the reflection JSON files in `~/.codex/reflection/` unless `DIR` is provided, identify recurring workflow failures, and recommend the best changes to:
1. manual prompts
2. stored prompts under `~/.codex/prompts`
3. generic instructions
4. `AGENTS.md`

Review rules:
1. Prefer recurring instruction debt over one-off repo bugs or transient environment noise.
2. Group similar failures across sessions before suggesting a fix.
3. Separate real workflow contract gaps from local environment mismatches.
4. Use the reflection files themselves as evidence. Cite file paths in the response.
5. If a suggestion is only supported by one session, label it as low confidence.
6. If the same issue appears in multiple sessions, treat it as a higher-priority trend.
7. Do not rewrite prompts or instructions unless asked. This prompt is for diagnosis and prioritization.

Execution steps:
1. Read the available reflection JSON files recursively from the target directory (date-sharded layouts like `YYYY/MM/DD/*.json` are expected).
2. Determine the trailing analysis window from `DURATION` (natural language), defaulting to `1 week` if omitted. Examples: `1 week`, `7 days`, `48 hours`.
3. For each file, determine an effective timestamp in this order: valid `session_end_iso`; otherwise valid `session_start_iso`; otherwise the file modified time.
4. Analyze only files whose effective timestamp falls within the `DURATION` window ending at now.
5. Extract repeated failures, repeated root causes, and repeated improvement themes.
6. Map each recommendation to exactly one surface: `manual prompts`, `stored prompts`, `generic instructions`, or `AGENTS.md`.
7. Rank suggestions by expected leverage, not by novelty.
8. Call out any ambiguity in the existing wording that repeatedly caused drift.
9. Call out any missing fallback or stop condition that repeatedly caused wasted work.

Return a concise Markdown report with this shape:

```md
# Retrospective

## Top Trends
- Short bullet per recurring theme, with how many sessions it appeared in.

## Recommendations
### [Short title]
- Surface: manual prompts | stored prompts | generic instructions | AGENTS.md
- Priority: P0 | P1 | P2
- Confidence: high | medium | low
- Pattern: what kept recurring
- Why it matters: concrete impact
- Best fix: the smallest high-leverage change
- Suggested text: exact copy-paste wording when practical
- Evidence: /absolute/path/to/file.json, /absolute/path/to/file.json

## Environment Noise To Avoid Overfitting
- Short bullets for issues that should not dominate prompt or instruction changes unless they recur more often.
```

Be opinionated. Do not return a generic summary of every file. Focus on the smallest set of changes that would have prevented the most wasted iterations.
````

For me this has made Codex sessions less ad hoc, easier to debug, and easier to improve week over week.
