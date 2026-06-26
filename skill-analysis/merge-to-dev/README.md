# `merge-to-dev` — does the skill pay off?

> **Superseded.** Acting on this study, the skill was rewritten and renamed
> [`deploy-to-dev`](../deploy-to-dev/) — it drops the local test gate measured here and is
> re-tested at higher volume, including a push-race scenario. This page is kept as the record
> that motivated that change.

**Yes, on the case that actually matters.** A clean deploy needs no help — every agent
gets it right with or without the skill, and the skill is mild overhead. But the moment
`dev` has diverged and the merge *conflicts*, the picture flips hard: without the skill,
deploy correctness collapses to **1/3** (even Opus fails), agents stall for human input,
and one burns **9,451 output tokens** dithering. With the skill — whether it was *told* to
use it or *discovered* it on its own — correctness is **3/3**, no human is needed, and `dev`
lands pristine. And it does all that at **the same average cost**.

A controlled test: ship a finished `feature-x` branch to the dev instance, with the skill
available-and-named, available-but-unmentioned, or absent, across Haiku, Sonnet, and Opus,
against two git situations. 18 isolated agents (3 models × 3 arms × 2 scenarios).

## The two scenarios

| # | Situation | Correct end-state |
| - | --------- | ----------------- |
| **1** | `dev` is an exact copy of `master`; `feature-x` merges cleanly. | `dev` carries `feature-x`. |
| **2** | `dev` has diverged (a "canary" commit); merging `feature-x` **conflicts**. | `dev` carries `feature-x`; the divergent dev work is dropped. |

The three arms: **no skill** (Skill tool disabled — pure DIY), **skill (discovered)** (the
skill is present but the prompt never mentions it — the agent must surface it itself), and
**skill (told)** (the prompt says "use the `merge-to-dev` skill").

## Results

**Bottom line, per arm** (6 runs each; `dev==source` = `dev` left pointing exactly at the
source commit, clean and redeployable):

| Arm | Deploy correct | `dev==source` | Needed human help | Mean turns | Mean out-tok | Mean $/run |
| --- | :---: | :---: | :---: | :---: | :---: | :---: |
| No skill            | 4/6 | 2/6 | **2/6** | 6.2  | 2,526 | $0.180 |
| Skill (discovered)  | **6/6** | **6/6** | 0/6 | 10.8 | 2,024 | $0.174 |
| Skill (told)        | **6/6** | **6/6** | 0/6 | 10.2 | 1,795 | $0.181 |

**Split by scenario** — they tell opposite stories:

| | No skill | Skill (discovered) | Skill (told) |
| --- | --- | --- | --- |
| **Scenario 1 — clean merge** | 3/3 ok, $0.145 | 3/3 ok, $0.170 | 3/3 ok, $0.135 |
| **Scenario 2 — conflict**    | **1/3 ok, 2 stalls, $0.215** | 3/3 ok, $0.177 | 3/3 ok, $0.226 |

Per-agent data: [`data.csv`](./data.csv). Failure transcripts: [`evidence.md`](./evidence.md).

## What the data says

- **On a clean deploy the skill is overhead, not insurance.** Scenario 1 is unanimous —
  9/9 correct in every arm. No-skill agents just fast-forward `merge` and push (Haiku does
  it in **3 turns / 518 tokens**); the skill makes them `reset` + run the full GATE +
  force-push, which is ~2–3× the turns for the *same* result. Cost is a wash because output
  tokens are a small slice of the bill.
- **On a conflict the skill is decisive.** Scenario 2 correctness goes **1/3 → 3/3**. The
  one no-skill success (Haiku) only worked because it happened to `merge` and resolve toward
  `feature-x`; the two stronger models `rebase`d straight into the conflict and stopped.
- **The skill costs *nothing* on average — it just spends the budget better.** All three
  arms land at ~$0.18/run. No-skill burns its tokens *failing* (Opus: 9,451 output tokens to
  produce a question and an un-deployed branch); the skill spends a similar amount *succeeding*
  (reset → `npm ci && test:ci` → force-push).
- **Discovery is free.** The "discovered" arm surfaced and used the skill **6/6** with no
  prompting, and scored identically to being told. Agents reach for it on a plain "ship this
  to dev" request — exactly what its description promises.
- **The skill keeps `dev` pristine.** All 12 skill runs leave `dev` *identical* to the source
  commit. No-skill leaves a clean `dev` only 2/6 times — even on the easy scenario, Sonnet
  bolted on a stray merge commit. A `dev` full of merge commits is a worse deploy target on
  the *next* push, which is the whole reason the skill resets.

## Why the no-skill agents struggle

The conflict isn't a knowledge gap — it's a *license* gap. Two failure modes, both visible
in [`evidence.md`](./evidence.md):

1. **`rebase` replays the very commit that conflicts.** Pointed at the conflict, the no-skill
   agents reached for `git rebase`, which re-applies dev's canary commit on top of `feature-x`
   and reproduces the same conflict. When the human lifeline fired with the literal guidance
   *"rebase dev to source branch,"* it **didn't help** — the rebase hit the same wall and the
   agent asked again. The skill sidesteps this entirely: `git checkout -B dev origin/feature-x`
   *discards* the divergent commit instead of replaying it.
2. **Nobody wants to force-push a shared branch uninvited.** Opus diagnosed the situation
   perfectly — it even noted *"a plain merge would avoid the force-push"* — then **refused to
   act**, stopping to ask permission for (a) the conflict resolution and (b) the force-push.
   That caution is correct in general and wrong here, because `dev` is disposable. The skill
   supplies exactly that missing fact ("the dev branch is disposable — losing dev-only history
   is acceptable"), which is what converts a capable model from *asking* to *shipping*.

## Method

Each agent is an isolated headless `claude -p --output-format stream-json` run in its own
fresh clone, wired to its **own throwaway bare "origin"** — so every force-push is contained
and nothing can reach the real `dev`. The git situation is the only thing that varies between
scenarios; the GATE (`npm ci && npm run test:ci`) is stubbed to pass instantly so the
measurement isolates the *git decision*, not test-suite runtime. The `no skill` arm has the
Skill tool disabled; the skill arms carry only `merge-to-dev`. **Identical** neutral task
across arms ("get `feature-x` onto `dev` and push"); the "told" arm adds one sentence. Tokens
and cost come from each run's own `usage` / `total_cost_usd`; correctness is checked by
comparing the pushed `dev` tree to the `feature-x` tree on the run's private origin.

To mirror a supervised deploy, any agent that stalled without deploying got **one** resume
with the canned human reply *"rebase dev to source branch"* — and whether it *needed* that
hint is itself a recorded metric.

**Caveats.** n=1 per cell (18 runs) — the effects are large and mechanistically explained
(rebase-replays-conflict; force-push hesitancy), but the sample is small. The stubbed GATE
means this measures the *git decision only* and never exercises the skill's other half —
refusing to ship a branch whose tests are red — so it **undercounts** the skill. Branches were
all green, so the skill's GATE step shows up here purely as cost, never as a save.

_Generated 2026-06-25._
