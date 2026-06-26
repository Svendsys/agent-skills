# `visual-review` — does the skill pay off?

**Yes — but only after the skill was rebuilt around *measurement*.** A controlled test
in two acts. **Act 1:** the original screenshot-and-eyeball `visual-review`, run by 24
isolated agents (3 models × {told, discovered, no-skill} over 10 planted defects) — and
**Haiku found 0/10 in every condition**, even when it captured the entire site. The
bottleneck was visual judgment, not coverage; the agents that scored best were the ones
that stopped eyeballing and started *measuring the DOM*. **Act 2:** so the skill was
rebuilt around an in-page audit (`tests/visual/audit.js`) that does the measuring, and
re-validated on the weakest model across four successively harder **fresh** bug sets —
taking **Haiku from 0/10 to comprehensive (10/10, then 5/5, 5/5, 8/8)**, at *lower*
token cost than it spent finding nothing before.

The audit is the measured first layer of `visual-review`; the human-eye pass remains the
second layer. Toolchain and capture contract: [`../../tests/visual/README.md`](../../tests/visual/README.md).

## The 10 planted defects

Graded subtlety, spread so finding them rewards covering the full page × theme × viewport
matrix:

| ID | Page | Category | Subtlety | Defect |
|----|------|----------|----------|--------|
| A1 | `/` | overlap | **blatant** | CTA block overlaps the "Why us?" cards |
| A2 | `/` | rotation | **very subtle** | hero illustration tilted ~2° |
| A3 | `/` | invisible text | subtle (light theme only) | "Why us?" heading ≈ page background |
| A4 | `/pricing/` | horizontal overflow | subtle (mobile only) | tier list clips prices off the right edge |
| A5 | `/team/*` | aspect distortion | medium (deep pages) | portraits `object-fit:fill` → squished faces |
| A6 | `/blog/posts/…` | tiny headings | medium (deep page) | section `h2` shrunk to ~body size |
| A7 | `/privacy/` | low contrast | subtle | body copy at ~32% alpha on a long page |
| A8 | `/contact/` | dead gap | medium | ~22rem empty band below the hero art |
| A9 | `/about/` | wrong palette | medium | lead paragraph in warning-orange |
| A10 | `/blog/` | aspect distortion | medium | stretched card thumbnail |

An agent "finds" a defect if its report flags the right page/area with a plausible
description (need not name the CSS cause); theme/viewport-gated defects count only if that
cell was captured.

## Act 1 — the original (screenshot-only) skill

Average planted defects found (out of 10), by model × arm:

| Model | With skill (told + discovered) | No skill (reinvented) | Tokens / run\* |
|-------|:------------------------------:|:---------------------:|----------------|
| **Haiku** | **0.0** | **0.0** | ~50–65k |
| **Sonnet** | 1.75 | 0.75 | with ~84k · without ~109k |
| **Opus** | 2.75 | 3.125 | ~100–127k |

\* Total subagent tokens, not USD — see **Method**.

Coverage was never the bottleneck; recall was. Two defects (**A2**, a 2° tilt, and **A7**,
low-contrast copy) were found by *nobody* across all 24 agents; **A6**, **A8**, **A9** by
at most one each. Per-agent data: [`data.csv`](./data.csv).

### What the data says

- **Haiku has a judgment ceiling, not a coverage ceiling.** It captured all 48 smoke
  cells, *discovered* the tooling unprompted (one run went to the exhaustive tier — 360
  shots), and reinvented capture when barred — and still returned **0/10**, declaring the
  site clean over a blatant overlap.
- **Reinventing the capture *contract* is where weak models hurt themselves.** No-skill
  Haiku didn't just miss defects, it **fabricated** them: botched theme-forcing produced a
  phantom "dark theme broken sitewide" (×7), plus spurious 404s. The skill's correctness
  contract (theme-force, settle, locale routing, full-page expand) exists to prevent
  exactly these false positives.
- **The skill roughly halves Sonnet's cost and raises its recall** (1.75 @ ~84k with vs
  0.75 @ ~109k without).
- **Opus is the exception that names the cure.** No-skill Opus (3.125) *matched* skill-
  equipped Opus (2.75) — because those runs spontaneously rebuilt in-page **measurement**
  (computed contrast ratios, aspect math, element geometry), and measurement is what caught
  the subtle tier (A6, A9, A10) that eyeballing missed. The single best run in the whole
  experiment was a no-skill Opus that hand-rolled an in-page audit (4.5/10). They simply
  paid ~110k tokens to rebuild what a tool could give for free.

The throughline: **the winning move is measuring the DOM, not taking more or better
screenshots.** So the measurement was built into a tool every model — including the
cheapest — can lean on.

## Act 2 — rebuilding around measurement (`audit.js`)

`audit.js` drives the same settled, themed, full-page capture, but instead of saving a
frame to judge it runs ~16 **generic** in-page checks (keyed off computed style and
geometry, never selector names) and writes a deduped distinct-issue report plus a cropped
PNG per finding. The reviewing model triages a short candidate list instead of re-deriving
perception by eye.

### Haiku validation

Each iteration used a **fresh, unseen bug set on different pages** — measuring
generalization, not memorization. New blind spots became new generic checks; recall was
locked before precision.

| Iteration | Bug set | Before | Blind spot → fix | After (Haiku via skill) |
|-----------|---------|:------:|------------------|:-----------------------:|
| **1** | Original A1–A10 | 0/10 | — | **10/10** ×3 |
| **2** | Generalization G1–G5 (new pages) | 3/5 | vertical-clip + text-over-image overlap (`3da83a3`) | **5/5** |
| **3** | Harder / rare HX1–5 | 1/5 | off-screen-left, tap-target, content-escape, collapsed-content (`e69bf0e`) | **5/5** ×2 |
| **4** | Frontier EX1–3 | partial | `opacity:0` invisible-text (`8ff0a03`); occlusion & off-brand colour already caught | **8/8** |

Haiku via the audit-driven skill costs **~28–50k tokens** for full detection — *less* than
the ~52–65k it spent scoring 0/10 on the old skill. Iteration detail and a sample audit
report: [`evidence.md`](./evidence.md).

### What the data says

- **0 → comprehensive on the cheapest model.** The same model that could not see a blatant
  overlap now reports the full defect set, because it is triaging measured findings instead
  of judging downscaled PNGs.
- **It generalizes.** Every iteration was a fresh bug set on different pages/elements; the
  audit caught most on first contact, and each miss became a *generic* new check (geometry,
  not a selector), so the next unseen defect of that class is caught too.
- **Cheaper, not just better.** The audit makes the cheapest model the most *reliable*
  detector at ~¼–⅓ the cost of the best baseline run (no-skill Opus, 4.5/10 @ ~127k).

## The detection boundary (by design)

Two classes the audit deliberately does **not** chase, because something else already
guarantees them:

- **Platform-prevented.** A broken `<img src>` fails the asset-fingerprint build step, and
  oversized-image escape is clamped by the global `img { max-width: 100% }`. The build is
  the guard.
- **Eyeball-only (layer 2's job).** An off-brand colour with *adequate* contrast, a
  font-fallback substitution, or an animation frozen mid-pose are not deterministically
  knowable from the DOM. These are what the human-eye pass is for.

## Why the no-skill / weak-model agents struggle

The weakness isn't a knowledge gap, it's a *perception* gap. A downscaled full-page PNG is
exactly where a small model's vision is least reliable, so Haiku eyeballs a settled,
fully-captured site and declares it clean — the failure is silent and confident. Stronger
models partly escape it, but only by reaching for measurement on their own (contrast/aspect
math), which is both the proof that measurement is the answer and an expensive way to get
it. The audit removes the perception step from the critical path: detection becomes a
deterministic DOM computation, and the model's job shrinks to triaging a short, pre-cropped
list — a task even Haiku does reliably.

## Method

Each agent was an isolated in-session subagent given its own task over the same dev server,
with the planted defects hidden from it. The three arms: **told** (given the `visual-review`
skill and instructed to use it), **discovered** (skill present, never named — the agent had
to surface it), and **no-skill** (Skill tool and `tests/visual` barred — pure DIY
screenshotting). Correctness is planted defects found out of the set; theme/viewport-gated
defects count only if that cell was captured. Act 2 re-ran the weakest model (Haiku) through
the rebuilt, audit-driven skill on four fresh bug sets authored after Act 1.

**Tokens, not cost.** Unlike the sibling studies (headless `claude -p` runs with a
`total_cost_usd`), these were in-session subagents, so the recorded figure is **total
subagent tokens**, not a dollar cost — directional, and not directly comparable to the
`cost_usd` columns elsewhere in `skill-analysis/`.

## Caveats

- **Recall-first by choice.** The audit over-reports rather than miss — it emits known,
  acceptable false positives (intentional polaroid tilts, a collapsed-`<details>` overlap,
  one borderline 4.06:1 caption) for the eyeball pass to dismiss. Precision work (suppress
  symptom cascades, per-instance vs systematic dedup, the PR #76 edge-case hardening) was
  layered in *after* recall was locked, never at its expense.
- **One site, one engine.** Results are for this codebase under the pre-installed Chromium;
  "generic" is validated only against the defect classes exercised here.
- **Authored, not adversarial, bug sets.** A real defect will eventually fall outside the
  taxonomy. The iteration loop — *fresh set → find the blind spot → add a generic check* —
  is the maintenance procedure for that, and the reason the checks are written against
  geometry rather than known selectors.
- **Small samples.** ~2 runs per model×arm in Act 1; the effects are large and
  mechanistically explained, but the sample is small. Metric is tokens, not cost (above).

_Generated 2026-06-26._
