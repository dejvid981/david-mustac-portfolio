# Plausible Events + UTM Helper — Design

**Date:** 2026-05-05
**Status:** Approved, in implementation
**Owner:** David Mustac

## Goal

Two complementary additions to the portfolio's analytics setup:

1. **Project Card View event** — measure which projects get *seen* (not just clicked), so we can compute click-through rate per project and identify weak hooks vs. weak placement.
2. **UTM helper Notion page** — give David a no-think reference of pre-tagged share URLs, so Plausible's source breakdown becomes "this LinkedIn post drives visitors" instead of the useless "linkedin.com drives visitors."

## What's already in place

- Plausible script installed in `<head>` (commit `4c51a3c`)
- Custom events live: `Project Click` (prop: `name`), `Get In Touch` (prop: `location`), `See My Work`, `Footer Link` (prop: `type`)
- Single delegated click listener at the bottom of `<body>` reads `data-event` + `data-prop-*` attrs

## Out of scope

- **Company-level visitor identification.** Plausible cannot do this by design (no cookies, no fingerprinting). Separate decision: install RB2B or Leadfeeder later if the question matters enough. Tracked as a follow-up todo, not part of this design.
- **Scroll-depth events** (25/50/75/100%). Considered, deprioritized — Plausible already shows session duration and bounce rate; scroll-depth adds complexity without much new signal for a one-pager.
- **Time-on-page bands.** Same reason as above.

## Decisions

| Decision | Choice | Reason |
|---|---|---|
| Card View threshold | 50% of card visible | Below 50% = scrolled past quickly; above 50% = actually saw it. Standard impression threshold. |
| Card View de-duplication | Once per page-load per card | Metric is "% of visitors who reached this card" — clean denominator for CTR. |
| Implementation method | Native `IntersectionObserver` | Same pattern already used for nav active-state. Zero deps, ~10 lines. |
| UTM helper location | Notion page in Karijera workspace | David runs Notion for execution. Lowest friction. |
| UTM naming convention | lowercase, hyphens, dates as YYYYMMDD | Prevents Plausible from showing 47 case-variants of the same campaign. |

## Section 1 — Project Card View event spec

**Event name:** `Project Card View`
**Prop:** `name` — same values as `Project Click` (D365 Migration / CPaaS Pricing / Revolut Croatia / Bolt Food Croatia / SDR Cost Calculator / Croatian Payments / Croatian SaaS)
**Markup change:** add `data-card-view="<project name>"` to each `.project-card` (7 cards).
**JS change:** add an `IntersectionObserver` block alongside the existing nav-state observer. Threshold 0.5. On first intersection per card, fire `plausible('Project Card View', { props: { name } })` and unobserve.

**Resulting metric in Plausible:** click-through rate per project = `Project Click count ÷ Project Card View count`. Both events should be set as Goals in the Plausible dashboard so they appear side-by-side.

## Section 2 — UTM Helper Notion Page

**Title:** `Portfolio Sharing — UTM Templates`

**Structure:**

1. Short "How to use" header (3 lines): paste template, replace `{placeholders}`, copy.
2. Naming convention rule (2 lines): lowercase, hyphens, dates YYYYMMDD.
3. Single table — 7 rows × 4 columns: Channel | Template | When to use | Example.

**Channels covered:**
- LinkedIn DM (1-1 outreach)
- LinkedIn post
- LinkedIn profile (Featured / About link)
- Job application (with platform sub-tag)
- Cold email
- GitHub README
- X / Twitter

## Verification

After deploy:

1. Open `david-mustac-portfolio.vercel.app` in a browser with DevTools Network tab open, filter `plausible.io`.
2. Scroll down through the Projects section. Watch each `Project Card View` event fire as cards cross 50% visibility, with the correct `name` prop.
3. Reload — scroll again. Each card should fire exactly once per reload.
4. Click a project. Confirm `Project Click` fires after `Project Card View` for that card (so CTR math is `clicks ≤ views`).
5. In the Plausible dashboard → Goals: add `Project Card View` and confirm it appears alongside `Project Click`.

## Follow-up todos (not in this design)

- [ ] Install RB2B pixel for visitor company identification (US-only mostly).
- [ ] Re-share existing portfolio links with UTM tags retroactively where it makes sense (top of LinkedIn profile, GitHub readmes).
