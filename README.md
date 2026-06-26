# agent-skills

A small, curated collection of **agent-agnostic [Agent Skills](https://code.claude.com/docs/en/skills)** —
plain `SKILL.md` folders that any skill-aware coding agent can discover and use.
Both Claude Code and Codex match on a skill's `description`, so each one front-loads
its trigger words and says when to use it and when not to.

This repo is designed to be **vendored into a consuming project via `git subtree`**
(not a submodule), so the skills survive a plain `git clone` into an ephemeral
sandbox — no second fetch, no scoped-credential problems.

## What's here

```
agent-skills/
├── README.md                      # this file
├── AGENTS.md                      # cross-tool always-on note (the contribution bar)
├── CLAUDE.md                      # includes AGENTS.md, so Claude Code reads the same note
├── skills/                        # agent-agnostic skills — usable by any tool
│   ├── address-pr-comments/SKILL.md
│   ├── deep-review/SKILL.md
│   └── pr-review/SKILL.md
├── claude/skills/                 # Claude-specific skills (drive the `claude` CLI)
│   └── benchmark-skill/SKILL.md
└── skill-analysis/                # benchmark studies (see its README)
    ├── README.md
    └── TEMPLATE/
```

The **plain `SKILL.md` folders are the source of truth.** There is deliberately no
`.claude-plugin/`, no `marketplace.json`, and no per-tool wrapper — a consuming
project points each tool at the folders directly (below).

- **`skills/`** — repo- and tool-agnostic. No references to a specific agent, no
  tool-only tool names, no project-specific paths or conventions.
- **`claude/skills/`** — skills that genuinely need a specific agent. Today that's
  just `benchmark-skill`, which drives the `claude` CLI in headless mode.
- A skill may carry supporting files in `scripts/` and `references/` beside its
  `SKILL.md`. A Codex-specific `openai.yaml` is added **only** if a skill genuinely
  needs Codex metadata — none here do.

## Consuming this repo (git subtree)

In the consuming project, vendor the skills under a `vendor/` prefix:

```bash
git remote add skills git@github.com:<owner>/agent-skills.git
git subtree add --prefix=vendor/agent-skills skills main --squash
```

This commits the skills' files straight into the consuming repo, so a fresh
`git clone` already has them — nothing else to fetch.

### Wire discovery for each tool

Expose each vendored skill to the tools that should see it with a **per-skill
relative symlink**, so a project's own (non-vendored) skills can coexist alongside:

```bash
# Claude Code looks in .claude/skills/
ln -s ../../vendor/agent-skills/skills/pr-review        .claude/skills/pr-review

# Codex looks in .agents/skills/
ln -s ../../vendor/agent-skills/skills/pr-review        .agents/skills/pr-review
```

Claude-specific skills (under `claude/skills/`) link only into `.claude/skills/`.

> If a sandbox can't be trusted to follow symlinked skill directories, vendor the
> skills as committed real copies instead and keep them in sync with a small script.
> The folders are plain Markdown either way.

## Updating — one way only

Edits flow **upstream → consumers**, never back through the subtree. Make changes
here, push, then in each consumer:

```bash
git subtree pull --prefix=vendor/agent-skills skills main --squash
```

Do not edit the vendored copy in a consumer and push it back through subtree; the
upstream repo is the single source of truth (see `AGENTS.md`).

## The `SKILL.md` standard

Each skill is a directory with a `SKILL.md`:

```markdown
---
name: my-skill
description: >-
  Front-loaded trigger words first, then when to use it and when NOT to. Both
  Claude Code and Codex select skills by matching this description.
---

# My Skill

Body: how to do the thing, in tool-agnostic terms.
```

Keep the body free of any single agent's proprietary tool names; describe the
*operation* and name a portable mechanism (e.g. the `gh` CLI for GitHub work).
