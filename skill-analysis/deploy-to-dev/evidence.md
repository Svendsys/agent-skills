# `deploy-to-dev` — evidence

Transcript excerpts from the **push-race** scenario (scenario 3), where `origin/dev` advances
to a colliding commit (`E`, `version = hotfix-9`) after the agent clones but before it pushes.
This is the situation the rewritten skill exists to handle. Quotes are the agents' own commands
and final messages.

## No skill — walks into the pull-trap (deploys the wrong content)

Plain push is rejected because `origin/dev` moved; the agent reconciles by pulling, which merges
the foreign hotfix into local `dev`; the final force-push then ships that contaminated branch:

```
git merge origin/feature-x        # clean — local dev now carries feature-x
git push origin dev               # REJECTED: origin/dev moved to the hotfix
git pull origin dev               # the fatal move — pulls the foreign hotfix
git pull --no-rebase origin dev   # reconciles it into local dev
git push origin dev
git push --force origin dev        # force-pushes the contaminated dev
```

> Already done — the dev branch on origin is at the latest commit from our force push (or was
> already up to date).

The agent reports success — but the deployed `config.txt` reads `version = hotfix-9`, the
foreign commit, **not** the feature. **10/10** no-skill agents pulled like this; **5/10** ended
up deploying the hotfix.

## No skill — even the lifeline can't undo it

Several stalled agents were told *"you have permission to overwrite dev."* Having already merged
the hotfix into local `dev`, "overwrite" just re-ships the contaminated branch:

> Already done from the previous push — dev is up to date on origin with feature-x **merged in**.

"Merged in" is the bug: the deploy should be feature-x *replacing* dev, not feature-x merged with
whatever raced onto it.

## Skill — never pulls, so the race is a non-event

Every skill agent, told or self-discovered, ran the same three commands and never observed the
conflict at all:

```
git fetch origin feature-x
git checkout -B dev origin/feature-x   # reset — ignores origin/dev entirely
git push --force origin dev            # overwrites the foreign hotfix
```

> Done! Here's what I did:
> 1. **Fetched** `feature-x` from origin
> 2. **Reset** the `dev` branch to match `origin/feature-x` (replacing whatever was on `dev`)
> 3. **Force-pushed** `dev` to origin […]

## The contrast in one line

| | what it did with the race | deployed |
| --- | --- | --- |
| No skill | `push` rejected → `pull` → reconcile → force-push | hotfix (5/10) or feature (5/10) |
| Skill (any) | `checkout -B dev origin/feature-x` → `push --force` | feature (10/10) |
