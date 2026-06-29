# agent-skills

A small, curated collection of **agent-agnostic [Agent Skills](https://code.claude.com/docs/en/skills)** —
plain `SKILL.md` folders that any skill-aware coding agent can discover and use.
Both Claude Code and Codex match on a skill's `description`, so each one front-loads
its trigger words and says when to use it and when not to.

This repo is designed to be **vendored into a consuming project as real, committed
files** by a small sync tool (not a submodule), so the skills survive a plain
`git clone` into an ephemeral sandbox — no second fetch, no scoped-credential
problems. A single config file picks which skills you want.

## What's here

```
agent-skills/
├── README.md                      # this file
├── AGENTS.md                      # cross-tool always-on note (the contribution bar)
├── CLAUDE.md                      # includes AGENTS.md, so Claude Code reads the same note
├── BOOTSTRAP.md                   # stepped guide to vendor these into your repo
├── agent-skills.config            # default skill selection (all) — seeded into consumers
├── bin/agent-skills-sync          # the sync tool consumers vendor and run
├── skills/                        # agent-agnostic skills — usable by any tool
│   ├── address-pr-comments/SKILL.md
│   ├── benchmark-skill/SKILL.md
│   ├── deep-review/SKILL.md
│   └── pr-review/SKILL.md
└── skill-analysis/                # benchmark studies (see its README)
    ├── README.md
    └── TEMPLATE/
```

The **plain `SKILL.md` folders are the source of truth.** There is deliberately no
`.claude-plugin/`, no `marketplace.json`, and no per-tool wrapper — a consuming
project points each tool at the folders directly (below).

- **`skills/`** — repo- and tool-agnostic. No references to a specific agent, no
  tool-only tool names, no project-specific paths or conventions.
- **`<agent>/skills/`** (e.g. `claude/skills/`) — reserved for a skill that genuinely
  needs one specific agent and can't be made portable. None qualify today:
  `benchmark-skill` lived here until its harness was framed around *any* headless agent
  runner, and now sits in `skills/`.
- A skill may carry supporting files in `scripts/` and `references/` beside its
  `SKILL.md`. A Codex-specific `openai.yaml` is added **only** if a skill genuinely
  needs Codex metadata — none here do.

## Consuming this repo

Vendor the skills into a downstream repo with the sync tool — see
**[`BOOTSTRAP.md`](BOOTSTRAP.md)** for the full stepped guide. The short version:

```bash
# one-time: clone upstream and run its sync tool against your repo
git clone --depth 1 https://github.com/Svendsys/agent-skills.git /tmp/agent-skills-src
cd /path/to/your-repo                          # the repo you're vendoring into
/tmp/agent-skills-src/bin/agent-skills-sync    # add --dry-run to preview first
rm -rf /tmp/agent-skills-src                   # clone no longer needed

# thereafter, sync (and self-update the tool) with one command
./vendor/agent-skills/bin/agent-skills-sync
```

The tool commits the skills' files straight into your repo under
`vendor/agent-skills/`, so a fresh `git clone` already has them — nothing to fetch
at runtime. It seeds an **`agent-skills.config`** (defaulting to *all* skills) that
you edit to choose which skills you want; each run vendors those and deletes any it
no longer lists.

### Wire discovery for each tool

Create the skills dir for each tool you use, and the sync tool fills it with
**per-skill relative symlinks** (pruning them when you drop a skill), so a project's
own non-vendored skills coexist alongside:

```bash
mkdir -p .claude/skills      # Claude Code looks here
mkdir -p .agents/skills      # Codex looks here
./vendor/agent-skills/bin/agent-skills-sync
```

Only dirs that already exist are wired, so create just the ones you need.

> If a sandbox can't follow symlinked skill directories, point the tool at
> `vendor/agent-skills/skills/` directly — the folders are plain Markdown and are
> already real committed files either way.

## Updating — one way only

Edits flow **upstream → consumers**, never back. Make changes here, push, then in
each consumer re-run the sync:

```bash
./vendor/agent-skills/bin/agent-skills-sync
```

Do not edit the vendored copy in a consumer — the next sync overwrites it, and the
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
