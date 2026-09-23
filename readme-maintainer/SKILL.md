---
name: readme-maintainer
description: Use whenever you finish a chunk of work that changes what a repository does, how it's used, or how it's structured — a new/removed public API, a changed CLI or install flow, a new major module, a dependency swap, or a milestone worth calling "status." Check the repo's README.md (or note that one should be created if none exists) and update it before considering the work done, especially right before a commit. Don't trigger for internal refactors, bugfixes, formatting, or comment-only changes that don't change what a user-facing README would say. Make sure to consult this skill even if the user never mentions "README" or "documentation" explicitly — keeping docs in sync with code is the point of this skill, not something that waits to be asked for.
---

# README Maintainer

A README goes stale the moment nobody's job is to update it. This skill makes that job automatic: after a change that actually affects what a user or new contributor would need to know, check whether `README.md` still tells the truth, and if not, fix only the parts that are now wrong.

## When to act

Fire this at the natural checkpoint after a unit of work is functionally done — the same moment you'd otherwise stage a commit — not after every individual edit. A change is **commit-worthy for the README** if it touches any of:

- Public interface: CLI flags/commands, exported functions/classes, API routes, config options
- How someone installs, runs, or gets started with the project
- Project structure: a new top-level module/directory/service, or one that was removed/merged
- Dependencies that affect setup (a new required tool, a bumped major version with breaking changes)
- Status-worthy milestones: first working release, a feature going from experimental to stable, a deprecation, a test suite gaining real coverage worth citing, a roadmap item landing or a new one becoming real

It is **not** commit-worthy if the change is an internal refactor, a bugfix with no observable behavior change, a test, a comment, or formatting — these don't change what the README should say, so touching it anyway just adds noise and risks introducing drift between the README's claims and a diff nobody asked it to reflect.

When in doubt, ask: "if someone only read the README, would they now be wrong about something?" If yes, update. If no, leave it alone.

## Before writing: read, don't assume

1. Read the current `README.md` in full if it exists. You're doing surgery, not writing from scratch — you need to know what's already there, including its existing tone and structure, before touching anything.
2. Identify exactly which sections are now inaccurate or incomplete because of the change you just made. Most changes affect one or two sections, not the whole file.
3. If `README.md` doesn't exist yet, that's the one case where you're writing broadly rather than surgically — bootstrap one using the structure below, scoped to what you actually know about the project (don't invent features, badges, or a roadmap that don't exist).

## Surgical editing rule

**Only edit the sections whose underlying facts changed.** This is the core constraint of this skill:

- Never regenerate the whole README from a template on a routine update — that discards hand-written voice, examples, and framing the maintainer chose deliberately, even if a fresh template might read more cleanly in isolation.
- Use targeted edits (the same way you'd edit any file) rather than overwriting the file wholesale.
- Preserve section ordering, heading style, and tone that's already established, even if it's not how you'd have written it from scratch — consistency with the rest of the file matters more than your personal preference for structure.
- If an existing section is simply missing (e.g. the project now has an architecture worth diagramming but there's no Architecture section), add it in a sensible place rather than restructuring what's around it.

## What a healthy README covers

"Professional" means something more specific than a few tidy paragraphs: it means reaching for the format that actually communicates the fact at hand — a table instead of prose when there are several things to compare at a glance, a diagram instead of a paragraph when a flow is hard to describe linearly, a real number instead of a vague claim.

Not every project needs every section below — apply judgment based on project size and audience, and never add a section just to check a box. But when relevant, this is the bar, and what to keep in sync as facts change:

1. **Title + badges + one-line value proposition.** The first couple lines answer "what is this" and "why would I care" without requiring a scroll. Badges belong here when they convey real, checkable facts — language/runtime version, key frameworks, test tooling, CI status, license, published package version — sourced from files that actually exist in the repo (`package.json`, a CI workflow, a `LICENSE` file). Never a decorative or aspirational badge. Update the hook if the project's core purpose has shifted.
2. **A callout pointer to deeper docs**, when a separate architecture/design doc exists (e.g. `docs/ARCHITECTURE.md`) — a short blockquote near the top, not the full design inlined. Keeps the README navigable while the deep-dive has room to breathe elsewhere.
3. **Table of contents**, once the README has enough sections that scanning it takes real scrolling (roughly five or more) — anchor links to each heading. Skip it for a short README; keep it in sync (add/remove/reorder entries) whenever a section is added, removed, or renamed.
4. **Status.** For a project with several components or features worth individually reporting on, use a table rather than a paragraph — it's scannable in a way prose isn't:

   ```markdown
   | | Status |
   |---|---|
   | Core feature A | ✅ Done, tested |
   | Feature B | 🚧 In progress |
   | Feature C | 🚧 Not started |
   ```

   For a single-component project, one honest sentence is enough; don't build a table with one row. Either way, source status only from what you can verify — version numbers, changelogs, what's actually implemented and tested in the repo — never invent a maturity claim or mark something ✅ that isn't.
5. **Tech stack**, as a table once there are enough distinct, deliberate technology choices that the *reasoning* behind them is worth capturing:

   ```markdown
   | Layer | Choice | Why |
   |---|---|---|
   | Language | TypeScript, strict mode | One type system across the whole codebase |
   | Testing | Vitest | Fast, native ESM/TS support |
   ```

   The "Why" column is the entire value, since the "Choice" column alone is just a restatement of the manifest file. Skip this for a project with one obvious runtime and no real choices to explain.
6. **Project structure**, as an annotated directory tree once there's more than a couple of top-level directories worth orienting a newcomer to:

   ```markdown
   project/
   ├── src/
   │   ├── core/       # main business logic
   │   └── api/        # HTTP layer, thin — delegates to core/
   └── tests/
   ```

   A flat single-file or single-module project doesn't need this.
7. **Getting started / Quick start.** Install + minimal working example, high enough up that a new user doesn't have to dig. For anything with multiple moving pieces to run (a server plus clients, several processes, multiple entry points), give an explicit numbered walkthrough of the real sequence, and a flags/options table for anything with more than a couple of CLI arguments. This is usually the single most important section to keep accurate — a broken quick start actively drives people away, and almost any CLI/config/install change means this section needs an update.
8. **Architecture and diagrams** (only when the project's structure or a key process is actually worth a picture — multiple interacting components, a non-obvious data/control flow, or a runtime sequence that's hard to describe linearly). Use inline **Mermaid** — GitHub and most modern git hosts render it natively in `.md`, no image generation or external tool needed:

   ````markdown
   ```mermaid
   graph LR
       Client --> API --> Queue
       Worker -->|drain| Queue
       Worker --> DB
   ```
   ````

   Reach for a component/relationship diagram (`graph`) *or* a process/flow diagram (`flowchart`, `sequenceDiagram`) depending on which fact you're conveying — a static component picture doesn't help someone trying to picture what happens over time, and vice versa. Base every diagram on the real repo (real module names, real sequence of events), never a generic placeholder shape. Skip diagrams entirely for small or single-file projects — an unnecessary one is worse than none.
9. **A design-philosophy section** ("How it thinks" / "Design" / similar), separate from the diagram and the usage instructions, when the codebase has a real, non-obvious abstraction or design decision that shapes how someone should reason about it (e.g. a key interface boundary, a deliberate architectural constraint). Don't manufacture one for a project with no such decision to explain.
10. **Testing**, with real specifics — what the suite actually covers, and genuine numbers (test count, coverage of edge cases, a simulation/fuzz suite's scale) pulled from the repo — rather than just "run the test command and see."
11. **Roadmap**, as a concrete ordered list, only when there's something real to draw it from (issues, TODOs, the maintainer's stated intent, commit history). Never fabricate one, and never leave a generic "future improvements" placeholder.
12. **License**, a short section or link, when a `LICENSE` file (or equivalent) exists.
13. **Contributor detail stays out.** Development setup, testing-the-tests, and PR process belong in `CONTRIBUTING.md`, not the main README — link to it if it exists rather than inlining it. The README is optimized for someone deciding whether/how to *use* the project, not for contributors.

Treat every one of these the same way as prose under the surgical-editing rule above: add a section the first time a project's complexity actually earns it, update just the row/cell/paragraph whose underlying fact changed, and leave the rest alone. Updating a status table means editing the one row that changed, not regenerating the table; updating a diagram means changing the one edge or node that's now wrong, not redrawing it from scratch.

## Boundaries

- Don't touch the README for changes that don't change any fact it states — see "When to act" above.
- Don't regenerate the whole file from a template when doing a routine update; that's reserved for the bootstrap case where no README exists yet.
- Don't fabricate status, metrics, badges, or roadmap items. If you don't have a real source for a claim, leave it out rather than guessing.
- Don't move contributor-only workflow detail into the README, and don't duplicate content that already lives in `CONTRIBUTING.md`.
- Don't let this skill block or delay the actual commit — it's a fast check-and-fix step, not a separate review gate.
