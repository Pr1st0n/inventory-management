# Design Spec — `saas-redesign` skill + app redesign

**Date:** 2026-07-20
**Status:** Approved design (pending spec review)
**Author:** brainstorming session

## 1. Goal

Two linked deliverables, in order:

1. **A reusable skill** (`saas-redesign`) that redesigns this Vue 3 inventory app's UI into a
   modern SaaS interface — a left vertical navigation sidebar (replacing the top nav), clean card
   layouts, consistent spacing, and a cohesive, professional design system. The skill is the
   **foundation**: it encodes *how* the redesign is done (which tools to orchestrate, in what
   order, with what guardrails).
2. **A redesign implementation plan** that *uses* the skill to redesign the actual app — the
   sidebar shell plus all 6 routed views.

**Hard constraint from the user:** no app code (`client/`) changes until the implementation plan
is written and approved. This spec and the skill are the setup; execution comes later.

## 2. Decisions (from brainstorming)

| Question | Decision |
|----------|----------|
| Skill scope | Tailored to **this** app (encodes its structure), not framework-generic. |
| Redesign depth | **Full overhaul** — sidebar shell + all 6 routed views. |
| Aesthetic source | Use the **`frontend-design`** skill to author a distinctive, professional "wow" design system (typography, color, components). The `docs/redesign-mockup.html` mockup is the **starting point**; `frontend-design` refines/pressure-tests it. |
| Skill structure | **Approach A — Orchestrator**: a fixed, verifiable pipeline. |
| Relationship | Skill first (foundation) → then a redesign plan that uses it. |
| Sidebar behavior | **Collapsible with an icons-only mode**, responsive for smaller screens (+ manual toggle). |

## 3. Current app facts (redesign targets)

- **Shell:** `client/src/App.vue` — sticky **top** nav (`router-link`s: Overview `/`, Inventory,
  Orders, Finance `/spending`, Demand Forecast `/demand`, Reports), plus `LanguageSwitcher`,
  `ProfileMenu`, the global `FilterBar`, and `<router-view>`. Global styles live here.
- **Routed views (6):** `Dashboard.vue` (`/`), `Inventory.vue`, `Orders.vue`, `Demand.vue`,
  `Spending.vue` (Finance), `Reports.vue`. `Backlog.vue` exists but is **unrouted** → out of scope.
- **State:** module-singleton composables — `useFilters` (4 global filters), `useI18n` (EN/JA +
  currency), `useAuth` (mock user). No Pinia.
- **Charts:** hand-built SVG/CSS (no chart library). **Status colors:** green/blue/yellow/red.
- **Project rules:** **no emojis in UI**; any `.vue` create/modify **must** go through the
  `vue-expert` subagent; browser testing via **Playwright MCP** against `:3000`/`:8001`.

## 4. The skill — `saas-redesign` (Approach A, orchestrator)

### Location & files
```
.claude/skills/saas-redesign/
├── SKILL.md                          # concise orchestrator procedure
└── references/
    ├── design-system-brief.md        # constraints handed to frontend-design
    ├── view-redesign-brief.md        # per-view brief template for vue-expert
    └── verification.md               # Playwright verification checklist
```

### Frontmatter
- **name:** `saas-redesign`
- **description:** "Redesign this Vue 3 inventory app's UI into a modern SaaS interface — a left
  vertical sidebar replacing the top nav, a cohesive design system, and consistent spacing. Use
  when asked to modernize/redesign the UI, convert the top nav to a sidebar, or give the app a
  polished professional/SaaS look."

### Pipeline (phases)

**Phase 0 — Preflight & safety**
- Ensure dev servers up (`start` skill if needed).
- Cut a feature branch (never `main`; verify `git config user.email` == `philipp.shestakov@accenture.com`; **no `Co-Authored-By` trailer**; commit only when the user asks).
- Baseline Playwright screenshots of all 6 routes → scratch dir for before/after.

**Phase 1 — Define the design system** (`frontend-design`)
- Invoke `frontend-design` with `references/design-system-brief.md` constraints: professional B2B
  dashboard (not maximalist), **no emojis**, must accommodate existing custom SVG charts + status
  colors, distinctive typography (no Inter/Roboto/system).
- Persist **`client/src/styles/design-system.css`** — CSS custom properties (color, type scale,
  spacing, radius, shadow, transitions) + component classes (`.card`, `.stat-tile`, `.data-table`,
  `.btn`, `.badge`, `.page-header`, sidebar classes). Plus a short `design-system.md`. Import globally.

**Phase 2 — Build the sidebar app-shell** (`App.vue` via `vue-expert`)
- Top nav → persistent **left vertical sidebar**: brand top; vertical nav items with active states
  (same 6 routes, same i18n labels); `LanguageSwitcher` + `ProfileMenu` docked at bottom.
- Layout: CSS grid `sidebar | main`; `main` has a sticky top bar (page title + global `FilterBar`)
  above a scrollable, padded `<router-view>`.
- **Collapsible / icons-only:** manual toggle collapses the sidebar to an icon rail (labels hidden,
  tooltips on hover); auto-collapses below a defined breakpoint (off-canvas/overlay on mobile).
  Collapsed state persists (localStorage), consistent with the app's existing persistence pattern.

**Phase 3 — Apply the system per view** (fan out to `vue-expert`, one per view, parallel)
- Each of the 6 views adopts tokens/components: standardized page header, spacing, cards, stat
  tiles, tables, chart framing. Custom SVG charts kept, restyled via tokens.
- Views edit distinct files and only *read* the shared stylesheet → safe to run in parallel.

**Phase 4 — Verify** (Playwright)
- Frontend compiles clean; each route renders; **no new console errors**; a filter change still
  updates data; capture after screenshots; diff vs baseline; report regressions.

**Phase 5 — Wrap up**
- Summarize, present before/after, leave on branch for review (offer merge/PR via
  `finishing-a-development-branch`). No auto-commit unless asked.

### Behavior-preservation guardrails (non-negotiable, restyle-only)
- **Must NOT change:** API calls, composables (`useFilters`/`useI18n`/`useAuth`), `computed`/`watch`
  logic, router paths, i18n keys, emitted events, prop contracts, `v-for` keys.
- **May change:** template layout/markup for styling, CSS, class names, adding design-system classes.
- No emojis. Preserve `FilterBar` wiring and existing i18n labels.

### Design-system lifecycle & mockup reuse

**When it's built:** once, up front, in **Phase 1** — before any shell or view work.
`frontend-design` authors it and it is persisted as the single source of truth:
`client/src/styles/design-system.css` (CSS custom properties: color, type scale, spacing, radius,
shadow, motion) + reusable component classes (`.card`, `.stat-tile`, `.data-table`, `.btn`,
`.badge`, `.page-header`, sidebar classes) + a short `design-system.md`. Imported globally so every
component consumes it.

**How it's enforced:** by construction in **Phases 2–3**, not policed after the fact —
- The stylesheet is the *only* place tokens live; the shell and all 6 views consume `var(--…)` and
  the shared component classes instead of ad-hoc colors/spacing/fonts.
- The `vue-expert` brief (`references/view-redesign-brief.md`) mandates token/class usage and
  **forbids hard-coded hex colors, pixel spacing, or font declarations** in views.
- **Phase 4** verification includes a consistency check (visual diff vs the mockup + a scan for
  rogue hard-coded styles in the views) and flags any drift.

**Mockup reuse:** yes — `docs/redesign-mockup.html` is reused two ways:
1. **As the seed** for Phase 1: its concrete values (dark sidebar, Bricolage/Figtree/JetBrains Mono
   type, cobalt accent, spacing/radius/shadow, card/tile/table patterns) are the starting point
   `frontend-design` refines into the production tokens. It is **not** shipped verbatim — the real
   UI is Vue components consuming the CSS tokens.
2. **As the acceptance reference** for Phase 4: the redesigned Dashboard is checked against the
   approved mockup.

## 5. The redesign plan (produced next, via `writing-plans`)
A separate implementation plan will sequence the actual redesign using the skill: preflight/branch →
design system → shell (with collapsible sidebar) → the 6 views (parallel) → verification. Scope =
shell + all 6 views. The mockup (`docs/redesign-mockup.html`) is the visual starting point.

## 6. Success criteria
- Skill: `SKILL.md` + references authored; validated by running it end-to-end on this app.
- Redesign: all 6 routes render with the new left sidebar (collapsible/icons-only) + cohesive design
  system; **zero behavior regressions** (filters, i18n, routing, charts intact); before/after
  screenshots demonstrate the transformation; no new console errors.

## 7. Out of scope
- `Backlog.vue` (unrouted) — optional, not in this pass.
- Backend, data, or API changes. WIP gaps (`/tasks`, `/purchase-orders`, `PurchaseOrderModal`) are
  not addressed by this redesign.
- New features or content changes — this is a **restyle**, not a rebuild.

## 8. Validation approach
The skill is validated by executing it once on this app (which also delivers the redesign). Success
is measured by the criteria in §6, verified with Playwright before completion is claimed.
