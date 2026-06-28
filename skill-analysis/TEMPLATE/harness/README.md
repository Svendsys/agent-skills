# harness/ — the committed, reproducible runner

This directory is the **canonical ruler** for the study. It is a committed artifact,
not scratch: persist everything needed to reproduce the benchmark byte-for-byte so
any later run — by anyone, in any session — compares cleanly against the existing
cohorts.

Commit here:

- **the runner** — the script that launches each isolated `claude -p` agent, one per
  model × scenario × arm cell;
- **fixtures** — the throwaway repo state / sandboxes each agent runs against, so
  side effects never reach anything real;
- **exact prompt files** — the byte-identical task text shared by all arms, plus the
  one extra sentence the *told* arm adds;
- **the pinned flag set** — full pinned model IDs (never the drifting tier aliases a CLI
  exposes for "the current `small` / `mid` / `large` model"), the turn/time/budget caps,
  and the clean/`--bare` isolation flags confirmed against the installed CLI.

A re-run uses this harness **unchanged** — that is what makes successive loop
iterations comparable. See the `benchmark-skill` SKILL.md, section 6, for the full
rationale (hermetic, bounded, one-variable runs).
