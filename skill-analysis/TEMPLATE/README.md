<!--
TEMPLATE — copy this directory to skill-analysis/<skill-name>/ and fill it in.
Match this shape; the benchmark-skill SKILL.md is the authority on what each part
must contain. Delete these HTML comments in a real study.
-->

# Does the `<skill-name>` skill pay off?

**Bottom line.** _One bold paragraph, verdict first_ — e.g. "Yes, on every model and
metric" or "Yes, on the one case that matters; no measurable difference on the easy
ones." Name the lead metric and the headline number that backs the verdict.

## Scenarios

| # | Situation | Correct end-state (what `data.csv` scores) |
| --- | --- | --- |
| 1 | _easy / obvious_ | _the single mechanically-checkable right result_ |
| 2 | _moderate_ | _…_ |
| 3 | _subtle / complex — isolates the skill's essence_ | _…_ |

## Results

### Haiku (anchor, n = 10 per cell)

| Arm | <lead metric> | needed_human_help | output tokens | turns |
| --- | --- | --- | --- | --- |
| No skill | | | | |
| Skill (discovered) | | | | |
| Skill (told) | | | | |

_Then split the same grid by scenario, so the easy floor and the hard ceiling are
visible separately._

### Across models (cross-check, n = 3 for Sonnet / Opus)

| Model | No skill | Discovered | Told |
| --- | --- | --- | --- |
| Haiku | | | |
| Sonnet | | | |
| Opus | | | |

## What the data says

- _Tight, evidenced bullets — each cites numbers from the grids above._
- _…_

## Why the no-skill agents struggle

_The mechanism, not just the gap — what the bare agent reaches for instead, and why
it's wrong._

## Evidence

See [`evidence.md`](./evidence.md). _For a visual skill, embed cropped PNGs:
with-skill-correct vs the no-skill failure modes._

## Method

_The matrix (models × scenarios × arms), where the committed `harness/` lives, and
how outcomes were checked mechanically._

## Caveats

_Sample sizes, what was stubbed (and that stubbing undercounts the skill's value
where it does), what is otherwise undercounted._

_Generated <YYYY-MM-DD>._
