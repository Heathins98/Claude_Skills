---
name: visual-plan
description: Use automatically during Claude Code Plan Mode whenever a final implementation plan is about to be presented via ExitPlanMode, or an existing plan is being revised. Before calling ExitPlanMode, render the plan's content as an enhanced HTML page — illustrative code-diff blocks, before/after architecture diagrams, and a visual summary of exact changes — and publish it as an Artifact so the user reviews a visual plan alongside the plain markdown. On revision, redeploy to the same Artifact instead of publishing a new one. If the environment does not have access to Artifacts, create the HTML locally and automatically open it in the browser once completed.
---

# Visual Plan

Plan Mode's built-in flow writes a plain-markdown plan file and calls `ExitPlanMode` to get it approved. That mechanism stays exactly as-is — don't change where the plan file lives or how `ExitPlanMode` works. This skill adds one step in between: turn that same plan content into a richer HTML page and publish it as an Artifact, so the user has a visual companion to review before approving.

## When to act

Right after you finish writing (or rewriting) the plan markdown file at the harness-provided path — in Plan Mode's own Phase 4 ("Final Plan") — and **before** calling `ExitPlanMode`. This includes revisions: if the user rejects a plan with feedback, or asks for changes to an already-published plan within the same conversation, rebuild and republish rather than skipping this step.

Do not use this skill outside Plan Mode, and don't invent an HTML plan for tasks too trivial to reach Phase 4 (the same "skip for trivial tasks" judgment Plan Mode itself already applies).

## Prerequisite skills

Before writing the HTML, load:
- `artifact-design` — mandatory for any Artifact; calibrates how much visual investment this plan warrants.
- `artifact-diagramming` — load it when the plan changes structure, data flow, or component relationships enough to be worth drawing (see Architecture diagrams below).

Do not load `artifact-capabilities` — this page is a static, per-session rendering. It doesn't need a database, user identity, or asset storage.

If Artifacts are not supported in the Claude environment instead make an html of the result in the plan directory and automatically open it in browser for the user. To get the same experience as using Artifacts.

## Content requirements

The HTML must be a **superset** of the plan markdown, not a lighter summary — every decision, file, and step in the `.md` must appear here too, just presented more visually. Structure:

1. **Header** — the plan's actual subject as the title (not "Plan" or "Implementation Plan"), plus the one-paragraph Context (why this change, what prompted it) from the plan file.
2. **One section per plan step/phase**, each with:
   - A plain-language description of the change (same substance as the corresponding markdown bullet/paragraph).
   - An **illustrative diff block** for any concrete code change — unified-diff style: monospace, `+`/`-` gutters, theme-aware red/green tints (define both light and dark tokens per the artifact theme rules — never hardcode a single-mode color). Label it clearly as a *proposed* change ("here's what this will look like"), since it is not a real `git diff` output yet.
   - A **before/after architecture diagram** via inline Mermaid (`<pre class="mermaid">...</pre>`) whenever the step changes structure, data flow, or how components relate — Artifacts render Mermaid natively, don't load a diagram library for this.
   - An affected-files list that visually distinguishes new / modified / deleted paths (e.g. distinct icon or tag per state) — don't just prose-describe file paths when the plan touches several.
3. **Verification section** mirroring the plan's own verification/testing steps.

If a step has no code-level change (e.g. a research or investigation step), skip the diff block for it rather than fabricating one.

## Mechanics

1. Write the page to a file in the session scratchpad directory, e.g. `plan.html`.
2. Publish it with the `Artifact` tool: `action: "publish"`, `file_path` pointing at that scratchpad file, a descriptive `title` (the plan's subject), a one-sentence `description`, and a `favicon` (only needed on the first publish for this plan).
3. **On revision within the same conversation**: rewrite the same scratchpad file and call `Artifact` again with the **same `file_path`** (omit `url` — same-conversation redeploys don't need it). This redeploys to the same URL. Never publish a second, separate artifact for a revision of the same plan.
4. Share the artifact link with the user, then continue the normal Plan Mode flow: the plan `.md` file is already written, so proceed straight to calling `ExitPlanMode`.

## Boundaries

- Never change the plan `.md` file's location or skip writing it — `ExitPlanMode`'s approval UI depends on it and is unaffected by this skill.
- Never call `ExitPlanMode` before the Artifact publish step for that revision has completed.
- Don't add Artifact capabilities (db/user/assets/etc.) to this page — it's a static rendering per plan revision, not an app.
