---
name: deep-review
description: >-
  Run a thorough, high-effort, high-standard review of the EXISTING codebase and
  design (not a new feature). Use when asked to "review the whole solution",
  "audit the project before release", find "what can be done better", check for
  technical debt / duplication / payload / accessibility / convention drift, or
  otherwise assess current functionality and quality in depth. Fans out parallel
  dimension audits, fixes what is unambiguously right, and surfaces the judgment
  calls for discussion.
---

# Deep Review

A repository-wide quality review. It asks a lot on purpose: the point is to find
everything worth knowing before a release or milestone, fix what is objectively
correct, and hand back a tight list of the decisions only the user can make.

The engine is **parallel dimension audits + a hard safe-vs-needs-input split +
re-verify before acting**. Breadth comes from fanning out; trust comes from the
orchestrator validating every finding against source. Do not shortcut either.

## 1. Ground the review

- Read `AGENTS.md` / `CLAUDE.md` and any conventions docs. **These are the bar** —
  the review measures the code against the project's *own* stated standards, not
  generic best practice.
- Capture a baseline so findings are grounded, not vibes: build/test time, output
  size and the largest artifacts, and confirm the suite is green. Note the numbers.

## 2. Fan out one audit per dimension (parallel, read-only)

Spawn a subagent per dimension and run them in parallel (however your tooling
spawns subagents). The dimensions below lean toward a web project — **adapt them
to the project under review**; drop what doesn't apply and add what does:

1. **Performance / payload** — what ships per page; dead/oversized JS, CSS, images,
   fonts; lazy-loading; caching/precache; the biggest assets.
2. **Accessibility** — semantics, ARIA, keyboard, focus, contrast, reduced-motion,
   and the no-JS baseline.
3. **i18n + copy + SEO** — translation completeness/parity; copy quality at a
   native-reader bar; metadata, canonical/hreflang, structured data, sitemap/robots.
4. **Conventions / duplication / single source of truth** — measured against the
   conventions doc: duplicated *shape*, parallel paths, values defined twice,
   history-narrating comments, dead code.
5. **Build + test framework** — build-time hotspots; test flakiness/ordering;
   coverage gaps; untested invariants; build gates.
6. **Tidiness / complexity / tech debt** — oversized *multi-concept* files, needless
   indirection, leaks, magic numbers, lying names; plus **navigability**: where a
   one-line in-file signpost saves the next reader (human or LLM) — never by bloating
   the conventions doc.

**Each mandate must:** scope to specific files/areas; demand `file:line` evidence
for every finding; forbid generic advice and pattern-matching from other codebases;
require each finding be classified **SAFE-TO-FIX** (objectively correct — no
product/taste/brand judgment) or **NEEDS-INPUT** (a tradeoff, design, or voice
call); be read-only (the orchestrator fixes); and **note what's already excellent**
so it isn't re-investigated.

## 3. Synthesize and verify (do not trust blindly)

- **Re-verify every finding against source yourself** before acting. Subagents go
  shallow or wrong; for an a11y/contrast/visual claim, look at the actual file or
  image. A finding you haven't confirmed is a hypothesis.
- **Reconcile cross-audit disagreements by reading the code** — audits correct each
  other (a "duplication" may be generated output; a suggested image optimization may
  degrade quality). Demote anything that doesn't survive a look.

## 4. Fix the safe set

Apply the SAFE-TO-FIX findings, honoring the project's idioms and existing seams
(reuse, don't add a parallel path). Keep every gate green — run lint, typecheck, and
the test suite after. When a fix protects an untested invariant, add the test.
Scope deliberately: do the unambiguous fixes; do **not** start the large refactor a
NEEDS-INPUT finding implies. Verify, then state plainly what was changed.

## 5. Report and present

- Write a committed report: **what was fixed** (with rationale), **what needs input**
  (each with evidence + a recommendation + why it's not unilateral, ordered by value),
  and **verified strengths** (so they aren't re-litigated). Quote copy findings
  exactly.
- Then present in chat as a **discussion opener, not a survey** — the user decides
  the NEEDS-INPUT items with you. Don't use a multiple-choice picker for these; lay
  out the highest-value decisions and let the conversation proceed.

## What makes it high-effort (the disciplines, not the length)

Breadth via parallel audits · evidence over assertion (`file:line`, look at the real
artifact) · the SAFE-vs-NEEDS-INPUT split (act autonomously on the clear, defer the
judgment) · verify before acting · measure against the project's own conventions ·
fix without degrading anything · keep the gates green.
