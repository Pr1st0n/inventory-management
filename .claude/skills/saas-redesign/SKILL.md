---
name: saas-redesign
description: Use when asked to redesign or modernize this Factory Inventory app's UI — convert the top nav bar to a left vertical sidebar, add a collapsible/icons-only sidebar, give it a modern SaaS or polished professional look, add clean card layouts, or apply a consistent design system / spacing across the Vue views. Triggers include "redesign the UI", "sidebar nav instead of top nav", "make it look like a SaaS", "modern/polished dashboard", "consistent design system".
---

# Factory Inventory SaaS Redesign

## Overview

Transform this app's UI into a modern SaaS interface: a **left vertical sidebar** (replacing the top
nav), clean card layouts, consistent spacing, and one cohesive design system — **without changing any
behavior**. This is a **restyle**, not a rebuild.

The redesign is an orchestrated pipeline that reuses two mandated tools:
- **`frontend-design`** authors the design system (Phase 1).
- **`vue-expert`** implements every `.vue` change (Phases 2–3) — a hard project rule.

**Core principle:** the design system is built once, up front, and every view is built to *consume*
its tokens — never ad-hoc colors, spacing, or fonts.

## When to use

- Converting the top nav to a left sidebar; adding a collapsible/icons-only sidebar.
- Giving the app a modern SaaS / professional look, card layouts, or consistent spacing.
- Standardizing styling across views under one design system.

**Not for:** backend/data/API changes, new features, or fixing the WIP gaps (`/tasks`,
`/purchase-orders`, missing `PurchaseOrderModal`). Those are out of scope.

## The pipeline

Run the phases in order. Details live in the reference files — read them when you reach each phase.

1. **Preflight & safety.** Ensure dev servers are up (`start` skill). Cut a feature branch (never
   `main`; verify `git config user.email` == `philipp.shestakov@accenture.com`; **no `Co-Authored-By`
   trailer**; commit only when the user asks). Capture baseline Playwright screenshots of all 6 routes.
2. **Define the design system** → **REQUIRED SUB-SKILL: use `frontend-design`**, briefed by
   [references/design-system-brief.md](references/design-system-brief.md). Persist as
   `client/src/styles/design-system.css` (tokens + component classes) + `design-system.md`; import globally.
3. **Build the sidebar app-shell** (`App.vue`) — **via `vue-expert`**. Left sidebar + sticky top bar
   (page title + global `FilterBar`) + scrollable content. Collapsible/icons-only + responsive. See
   [references/view-redesign-brief.md](references/view-redesign-brief.md) (Shell section).
4. **Apply per view** — **via `vue-expert`, one subagent per view, in parallel** (each edits a distinct
   file; all only read the shared stylesheet). Use [references/view-redesign-brief.md](references/view-redesign-brief.md).
   Views: `Dashboard`, `Inventory`, `Orders`, `Demand`, `Spending` (Finance), `Reports`.
5. **Verify** with Playwright per [references/verification.md](references/verification.md), then report
   with before/after screenshots. Leave on the branch for review.

## Non-negotiable guardrail: restyle only

**Violating the letter of this rule is violating its spirit.** When restyling, you may change template
layout markup, CSS, and class names. You may **NOT** change: API calls, composables
(`useFilters`/`useI18n`/`useAuth`), `computed`/`watch` logic, router paths, i18n keys, emitted events,
prop contracts, or `v-for` keys. No emojis. The full rationalization table and red flags are in
[references/view-redesign-brief.md](references/view-redesign-brief.md) — every redesign subagent must follow them.

## App map (redesign targets)

| Area | File(s) |
|------|---------|
| Shell / top nav / global styles | `client/src/App.vue` |
| Routed views (6) | `client/src/views/{Dashboard,Inventory,Orders,Demand,Spending,Reports}.vue` |
| Global filters | `client/src/components/FilterBar.vue` + `composables/useFilters.js` |
| Nav-adjacent | `components/{LanguageSwitcher,ProfileMenu}.vue`, `composables/{useI18n,useAuth}.js` |
| New design system | `client/src/styles/design-system.css` + `design-system.md` (created in Phase 2) |

`Backlog.vue` is unrouted → out of scope. Charts are hand-built SVG (no chart lib); keep them, restyle
via tokens. Status colors green/blue/yellow/red must survive.

## Red flags — STOP

- About to edit a `.vue` file directly instead of dispatching `vue-expert`.
- About to hard-code a hex color, px spacing value, or font in a view instead of using a token.
- About to touch a `useFilters`/`computed`/`watch`/router/i18n line "while I'm in here".
- Skipping `frontend-design` and inventing the design system inline.
- Claiming done without Playwright verification + before/after screenshots.

Any of these → stop and return to the pipeline.
