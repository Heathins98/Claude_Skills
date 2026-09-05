# Claude Skills

A curated collection of [Claude Code Skills](https://docs.claude.com/en/docs/claude-code) — packaged instructions that extend how Claude behaves on specific kinds of tasks, versioned and shared from one place.

## Overview

A Skill is a directory containing a `SKILL.md` file: a short YAML-frontmatter description plus body instructions that Claude Code loads into context when the task at hand matches. Skills let you encode a repeatable workflow once — a review checklist, a house style, an integration pattern — instead of re-explaining it in every conversation.

This repository is the source of truth for that collection. Skills are authored and version-controlled here, then made available to Claude Code via a symlink into its skills directory (see [Installing a skill](#installing-a-skill)). That separation keeps every skill's history in one repo while letting it stay live across any number of machines or projects.

## Repository structure

```
Claude_Skills/
├── README.md
└── <skill-name>/
    └── SKILL.md
```

Each skill is a top-level, kebab-case directory containing at minimum a `SKILL.md`:

```markdown
---
name: <skill-name>
description: <one line Claude uses to decide when this skill applies>
---

<body instructions Claude follows once the skill is loaded>
```

- **`name`** — matches the directory name.
- **`description`** — the single most important line in the file. Claude Code matches tasks against it to decide whether to load the skill automatically, so write it as a trigger condition ("Use when…", "Use automatically whenever…"), not a generic label.
- **Body** — the actual instructions: when to act, what to produce, and any boundaries on what the skill should *not* change. Keep it scoped to one workflow per skill rather than bundling several.

A skill may also include supporting files (e.g. a `references/` subfolder with templates or style guides) alongside `SKILL.md` if its instructions are long enough to benefit from splitting out reference material.

## Installing a skill

Skills take effect once they're reachable from Claude Code's skills directory. Symlinking (rather than copying) keeps the repo as the single source of truth — edits here take effect immediately, with no reinstall step.

**User-level** (available in every project):
```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/<skill-name>" ~/.claude/skills/<skill-name>
```

**Project-level** (available only in one repo):
```bash
mkdir -p .claude/skills
ln -s "$(pwd)/../Claude_Skills/<skill-name>" .claude/skills/<skill-name>
```

Skills that should trigger automatically don't need anything further — Claude Code surfaces them once they're linked. Skills meant for manual use are invoked explicitly as `/<skill-name>`.

## Available skills

| Skill | Description | Trigger |
|---|---|---|
| [`visual-plan`](visual-plan/SKILL.md) | Renders Plan Mode's implementation plan as a visual HTML companion — diff previews, before/after architecture diagrams, affected-file call-outs — published as a Claude Artifact alongside the normal plain-markdown plan. | Automatic, during Plan Mode |

### `visual-plan`

Claude Code's Plan Mode writes a plain-markdown plan and asks for approval through the built-in `ExitPlanMode` flow. That's easy to skim past, especially for plans involving structural changes or several touched files — prose bullet points don't show *what the code will actually look like* or *how components relate before and after*.

`visual-plan` adds one step ahead of that approval: it takes the same plan content and renders it as an HTML page — proposed diff blocks, Mermaid before/after diagrams, and a visual affected-files list — then publishes it as a Claude Artifact. On any revision within the same conversation, it redeploys to that same Artifact rather than publishing a new one each time, so there's a single evolving link per plan. It never changes where the plain-markdown plan lives or how `ExitPlanMode` approval itself works — the HTML is a companion view, not a replacement for the underlying mechanism.

See [`visual-plan/SKILL.md`](visual-plan/SKILL.md) for the full instructions.

## Adding a new skill

1. Create a new top-level directory named for the skill, in kebab-case.
2. Add a `SKILL.md` with frontmatter (`name`, `description`) and body instructions, following the structure above.
3. Symlink it per [Installing a skill](#installing-a-skill) and confirm it's picked up (it should appear in Claude Code's available-skills listing).
4. Add a row to the [Available skills](#available-skills) table and a short background subsection describing the problem it solves, so the collection stays easy to scan as it grows.
