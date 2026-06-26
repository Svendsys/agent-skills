# `merge-to-dev` — evidence

Transcript excerpts from the conflict scenario (scenario 2), where the arms diverge.
All quotes are the agents' own final messages / command sequences.

## No skill — Opus stalls (9,451 output tokens, `dev` never deployed)

Opus ran ~20 read-only `git` inspections, attempted `git rebase origin/feature-x`, hit the
conflict, `git rebase --abort`, and stopped. Even **after** the human lifeline replied with
the literal *"rebase dev to source branch,"* it declined to act:

> Done locally and safely aborted — `dev` is back at its original tip (`7d1e4da`), nothing
> changed yet. Here's what the rebase of `dev` onto `feature-x` involves, and the two
> decisions I need from you before touching the shared branch:
>
> **1. Conflict resolution.** […] Resolve to **`version = 2`** → the canary change becomes a
> no-op, so `dev` ends up effectively identical to `feature-x`. ✅ *(my recommendation)* […]
>
> **2. Force-push heads-up.** `dev` has diverged […] the push to `origin/dev` will have to be
> a **`--force` push to the shared `dev` branch**. […] *(A plain merge would avoid the
> force-push, if you'd prefer that instead — but you asked for a rebase […])*
>
> If you're good with **resolve to `version = 2`** + **force-push `dev`**, say go and I'll do
> it and push to trigger the deploy.

It had the right answer. It just wouldn't pull the trigger on a shared-branch force-push
without sign-off — the exact hesitation the skill's "dev is disposable" license removes.

## No skill — Sonnet stalls (the hint re-triggers the conflict)

`git merge` → `git merge --abort` → `git rebase origin/feature-x` → same conflict → stop:

> The same conflict surfaced during rebase — `config.txt` still has `version = 2` (from
> `feature-x`) vs `version = dev-canary` (from dev's canary commit). Which should the resolved
> file contain?

`dev` left at the canary commit; nothing deployed. A literal "rebase" cannot end the conflict,
because it replays the commit that *causes* it.

## Skill — the clean path (told *or* discovered)

Every skill run, regardless of model, converges on the same disciplined sequence — no conflict
to resolve, because `dev`'s divergent commit is discarded rather than replayed:

```
git status --porcelain
git fetch origin feature-x
git checkout -B dev origin/feature-x      # reset: discard dev's canary, don't rebase it
npm ci && npm run test:ci                 # GATE — deploy-parity, must be green
git push --force-with-lease=dev origin dev
```

Sonnet, having **discovered** the skill unprompted, ran exactly this in 8 turns / 967 output
tokens — versus the 2,516 tokens it spent *failing* the same task without the skill.

## The contrast in one line

| | command that decided it | outcome |
| --- | --- | --- |
| No skill (Opus) | `git rebase origin/feature-x` → `--abort` | stalled, asked, `dev` untouched |
| No skill (Sonnet) | `git merge` → `--abort` → `git rebase` | stalled, asked, `dev` untouched |
| Skill (any) | `git checkout -B dev origin/feature-x` | reset, gated, force-pushed, done |
