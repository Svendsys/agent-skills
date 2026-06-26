# `visual-review` — evidence

How the audit was built (the Act 2 iteration log) and what it actually emits. Every score
below is a Haiku run via the audit-driven skill; every "fix" is a *generic* check keyed off
geometry, not a selector.

## The iteration log

Each iteration introduced a **fresh** bug set on different pages/elements, measured Haiku's
recall, turned every miss into a new generic check, and re-measured.

### Iteration 1 — original set (A1–A10) · commit `a629852`

The in-page audit (contrast incl. `color(srgb)`, aspect distortion, overflow/clipping,
overlap, rotation, heading-size, empty-gap, broken-image), scoped to `<main>`, deduped to a
distinct-issue report + crops. Deterministic self-test **10/10**. (One planted defect, A7,
had to be re-planted as a genuine low-contrast defect — the original 32%-alpha was overridden
by prose and rendered inert.) Haiku: **10/10 ×3** (`ht1`, `ht2`, and `ht3`, which *discovered*
the skill unprompted before running it).

### Iteration 2 — generalization (G1–G5) · commit `3da83a3`

Fresh probes on new pages: `G1` contrast/qa-question, `G2` overflow/about-desktop, `G3`
overlap landing **text-on-image** at contact, `G4` aspect on a blog raster cover, `G5`
**vertical-clip** on a pricing card. First run **3/5**. Two blind spots:

- `G5` — `overflow-y` clipping wasn't checked (only `overflow-x`).
- `G3` — overlap was **text-vs-text only**, so text-over-imagery slipped through.

Fix: add a vertical-clipping check and a text-over-image overlap check. Both fire only on
their probe page (FP-clean). After: **5/5** (`hg1`).
*(`G6`, a broken `<img src>`, can't ship — the asset-fingerprint build step rejects unresolved
refs. The build is the guard; the audit needn't catch it.)*

### Iteration 3 — harder / rare classes (HX1–5) · commit `e69bf0e`

`HX1` off-screen-left (heading `translateX`), `HX2` tiny tap-target (mobile CTA), `HX3`
content-escape (oversized child in a non-clipping card), `HX4` collapsed-content
(`height:0; overflow:hidden` with text), `HX5` single-line ellipsis truncation. First run
**1/5** (only HX5, via the existing overflow-x). Four generic checks added: off-screen-left,
tap-target (mobile, both dims < 24px), content-escape (child wider than a non-clipping,
unrotated parent), collapsed-content (text but ~0 size; skips closed `<details>`,
`aria-hidden`, and the 1×1 sr-only trick). After: **5/5 ×2** (`hh1`, `hh2`).
*(Two more classes are platform-prevented: broken `<img src>` fails the build; oversized-image
escape is clamped by the global `img { max-width: 100% }`.)*

### Iteration 4 — frontier (EX1–3) · commit `8ff0a03`

Probing the boundary of deterministic detection:

- `EX1` z-index **occlusion** (text overlay over content) → **already caught** by the overlap
  check; geometry is z-order-independent.
- `EX3` off-brand **green** that still passes contrast → **caught**, because the planted green
  genuinely failed contrast (it wasn't actually adequate).
- `EX2` truly transparent text (`opacity:0`) → a **real gap** → added an invisible-text check
  (guarded against tooltips / reveal animations / `aria-hidden` / `<details>`; FP-clean).

After: **8/8** (`hf1`, covering HX1–5 + EX1–3 including the new invisible-text check).

**The boundary it stops at, by design (the eyeball pass's job):** an off-brand colour with
*adequate* contrast, a font-fallback substitution, an animation frozen in an ugly pose —
none deterministically knowable from the DOM.

## What the audit emits

Real output from a `--tier=smoke` run on the **clean** tree (no planted defects), showing the
format the model triages — and the recall-first false positives it deliberately surfaces for
the eyeball pass to dismiss:

```
# Automated visual audit — 8 distinct issue(s)
Tier `smoke` · 48 cell(s) · 52 raw finding(s) collapsed to 8 distinct issue(s).

## 1. [high] Overlapping elements (collision) — /qa/ — `div.qa-content-section-body > p + summary.qa-question > h3`
## 2. [high] Overlapping elements (collision) — /qa/ — `div.qa-content-section-body > p + div.qa-content-section-body > p`
## 3. [med] Unexpected rotation / tilt — /about/ — `li.team-grid-item > a.team-polaroid`
## 4. [med] Low text contrast — /about/ — `span.team-polaroid-caption > span.team-polaroid-role`
       text contrast 4.06:1 (min 4.5 for normal text)
## 5–8. [med] Unexpected rotation / tilt — /team/<member>/ — `header.team-profile-header > div.team-profile-polaroid`
```

All eight are known-acceptable: the polaroid tilts are intentional design, the `/qa/` overlaps
are a collapsed-`<details>` artifact, and the 4.06:1 caption is borderline. Each carries a
crop, so the reviewing model confirms or dismisses it in one look — which is exactly what
Haiku does (it buckets the rotations as "likely intentional"). 52 raw findings collapse to 8
distinct rows; one crop per row.

## The contrast in one line

| | what it did | result |
| --- | --- | --- |
| Haiku — original skill | captured 48–360 settled screenshots, eyeballed them | "no defects" — **0/10**, over a blatant overlap |
| Haiku — audit-driven skill | ran the audit, triaged the measured report + crops | the full set — **10/10**, ~half the tokens |
