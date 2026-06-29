# AGENTS.md

Always-on context for any agent working **in this repository**. Claude Code reads it
via `CLAUDE.md` (which includes this file); Codex and other tools read `AGENTS.md`
directly. It is the contribution bar for the skills kept here.

## What this is

`agent-skills` is a curated set of **agent-agnostic Agent Skills** — plain `SKILL.md`
folders meant to be vendored into other projects via `git subtree` and discovered by
any skill-aware coding agent (Claude Code, Codex, and others). See `README.md` for
the layout and how consumers wire it up.

## Rules for the skills here

1. **The plain `SKILL.md` folder is the source of truth.** Do not add per-tool
   wrappers — no `.claude-plugin/`, no `marketplace.json`, no tool-specific manifest.
   Consumers point each tool at the folders directly.
2. **`skills/` stays agent- and project-agnostic.** No references to a specific agent
   by name, no proprietary tool-only tool names, no project-specific paths,
   conventions, or invariants. Describe the *operation* and name a portable mechanism
   (e.g. the `gh` CLI for GitHub work). If a skill can only work with one agent, it
   does not belong in `skills/`.
3. **Agent-specific skills live under that agent's folder.** A skill that genuinely needs
   one agent — and can't be described in terms of a portable mechanism — lives under that
   agent's folder (e.g. `claude/skills/`), not `skills/`. None do today: `benchmark-skill`
   moved into `skills/` once its harness was framed around *any* headless agent runner
   rather than one CLI.
4. **Every `SKILL.md` has YAML frontmatter with `name` and `description`.** The
   `description` front-loads trigger words and states when to use the skill and when
   **not** to — both Claude Code and Codex select skills by matching it, so vague
   descriptions mean the skill never fires.
5. **Supporting files go in `scripts/` and `references/`** beside the `SKILL.md`. Add
   a Codex `openai.yaml` only if a skill genuinely needs Codex-specific metadata.

## Updates flow one way

This repo is the single upstream source. Edits are made here and pulled into
consumers with `git subtree pull`; they are **never** pushed back through a
consumer's subtree. A change vendored into a project and edited there is a fork to
reconcile, not an update.

## Standard of work

These skills are themselves a display of craft. Hold them to the same bar a good
skill asks of code: single source of truth (no copy-pasted shape), say what's true,
fix what's not up to par rather than deferring it, and leave every change an
improvement. `skill-analysis/` measures whether a skill actually pays off — when you
run a benchmark, its results are saved there (see `skill-analysis/README.md`).
