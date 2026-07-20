# SaaS UI Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. This plan **drives the `saas-redesign` skill** — read `.claude/skills/saas-redesign/SKILL.md` and its `references/` first.

**Goal:** Redesign the Factory Inventory app's UI into a modern SaaS interface — a collapsible left sidebar replacing the top nav, clean card layouts, and one cohesive design system across all 6 views — with zero behavior changes.

**Architecture:** Orchestration, not hand-written code. Phase 1 uses `frontend-design` to author a design-system stylesheet (the single source of truth). Phases 2–3 use `vue-expert` (one subagent per `.vue` file) to build the sidebar shell and restyle each view to consume those tokens. Playwright verifies render + preserved behavior. The mockup `docs/redesign-mockup.html` is the visual seed/acceptance reference.

**Tech Stack:** Vue 3 + Vite; the `saas-redesign` skill; sub-skills `frontend-design` + `vue-expert`; Playwright MCP for verification.

## Global Constraints

- **`.vue` files:** every create/modify MUST be done by the `vue-expert` subagent (project rule). Never edit a `.vue` directly.
- **Restyle only:** never change `api.*` calls, composables (`useFilters`/`useI18n`/`useAuth`), `computed`/`watch`/`ref` logic, router paths, i18n keys (`t('…')`), emitted events, prop contracts, or `v-for` `:key`. Full rationalization table + red flags: `.claude/skills/saas-redesign/references/view-redesign-brief.md`.
- **Design tokens only** in views — no hard-coded hex colors, px spacing, or font-family. Source of truth: `client/src/styles/design-system.css`.
- **No emojis** in UI. Preserve status color semantics: green=delivered, blue=shipped, amber=processing, red=backordered.
- **Ports:** frontend `:3000`, backend `:8001`. Browser testing via Playwright MCP only.
- **Git:** work on a feature branch (never `main`); verify `git config user.email` == `philipp.shestakov@accenture.com`; **no `Co-Authored-By` trailer**; commit only when the user asks — otherwise leave changes staged/committed per the executor's chosen mode.
- **Out of scope:** `Backlog.vue` (unrouted), backend/data/API, WIP gaps (`/tasks`, `/purchase-orders`, `PurchaseOrderModal`), new features.

---

### Task 1: Preflight & baseline

**Files:**
- Create: `.redesign-scratch/baseline/` (screenshots; gitignored scratch, not committed)

**Interfaces:**
- Produces: a feature branch `ui-saas-redesign`; baseline screenshots `baseline-{overview,inventory,orders,demand,spending,reports}.png` for before/after comparison.

- [ ] **Step 1: Confirm servers up.** Use the `start` skill (or verify `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000` → `200` and `.../8001/docs` → `200`).
- [ ] **Step 2: Verify git identity.** Run: `git config user.email` — Expected: `philipp.shestakov@accenture.com`. If not, STOP and ask.
- [ ] **Step 3: Create branch.** Run: `git checkout -b ui-saas-redesign`.
- [ ] **Step 4: Capture baselines.** Playwright: resize to 1360×900, navigate each of `/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`, full-page screenshot each, and record `browser_console_messages` per route as the error baseline.
- [ ] **Step 5: Commit checkpoint.** `git add -A && git commit -m "chore: baseline before saas redesign"` (branch only; scratch screenshots excluded via .gitignore).

---

### Task 2: Design system (via `frontend-design`)

**Files:**
- Create: `client/src/styles/design-system.css`, `client/src/styles/design-system.md`
- Modify: `client/src/main.js` (add global import of `design-system.css` before app styles)

**Interfaces:**
- Consumes: mockup `docs/redesign-mockup.html` (seed); brief `.claude/skills/saas-redesign/references/design-system-brief.md`.
- Produces: CSS custom-property tokens (color/type/spacing/radius/shadow/transition/sidebar dims) + component classes `.card`, `.card__head`, `.stat-tile`, `.data-table`, `.btn`/`.btn-primary`/`.btn-ghost`, `.badge`/`.status`, `.page-header`, `.sidebar`, `.nav-item`, `.filter-bar`, `.fselect`. These names are the contract every later task consumes.

- [ ] **Step 1: Dispatch `frontend-design`** with the full brief at `references/design-system-brief.md` and the seed mockup. Instruct it to persist `design-system.css` + `design-system.md` and to import the CSS globally in `main.js`.
- [ ] **Step 2: Verify tokens exist.** Run: `grep -cE "^\s*--" client/src/styles/design-system.css` — Expected: a substantial count (tokens present). And `grep -nE "\.card|\.stat-tile|\.data-table|\.sidebar|\.nav-item|\.status" client/src/styles/design-system.css` lists the component classes.
- [ ] **Step 3: Verify global import.** Run: `grep -n "design-system.css" client/src/main.js` — Expected: one import line.
- [ ] **Step 4: Verify boot.** Playwright navigate `/`; check `browser_console_messages` for no new errors; confirm fonts load (no FOUT error). App may look partially unstyled until the shell task — that's fine.
- [ ] **Step 5: Commit.** `git add client/src/styles client/src/main.js && git commit -m "feat(ui): add SaaS design system tokens + components"`.

---

### Task 3: Sidebar app-shell (`App.vue`, via `vue-expert`)

**Files:**
- Modify: `client/src/App.vue`

**Interfaces:**
- Consumes: design-system classes/tokens from Task 2 (`.sidebar`, `.nav-item`, `.filter-bar`, `.page-header`, sidebar-dim tokens).
- Produces: the shell layout every view renders inside — a `<router-view />` content region with consistent page padding, plus a sticky top bar hosting `<FilterBar />`. Collapse state key: `localStorage['sidebar-collapsed']`.

- [ ] **Step 1: Dispatch `vue-expert`** with `references/view-redesign-brief.md` (Shell section) + `design-system.md`. Requirements: top nav → left vertical sidebar (brand top; same 6 `<router-link>`s, same paths + i18n labels, as `.nav-item`s with active state; `LanguageSwitcher` + `ProfileMenu` at bottom, emits/props intact); grid `sidebar | main`; sticky top bar with page title + unchanged `<FilterBar />`; scrollable padded `<router-view />`. Collapsible/icons-only with hover tooltips; auto-collapse ≤1024px; off-canvas ≤640px; persist collapsed state in `localStorage`. Keep existing modals/tasks state working.
- [ ] **Step 2: Verify render.** Playwright navigate `/`; sidebar visible on the left, top nav gone; `browser_console_messages` shows no new errors; screenshot vs mockup.
- [ ] **Step 3: Verify nav + i18n.** Click each `.nav-item` → correct route; toggle EN/JA → labels change (keys unchanged).
- [ ] **Step 4: Verify collapse.** Trigger toggle → icons-only + tooltips; `browser_resize` to 1000×800 → auto-collapsed; to 600×800 → off-canvas; reload → collapsed state persisted.
- [ ] **Step 5: Verify filters still wired.** Change a `FilterBar` dropdown → confirm a network request fires / data reacts (wiring intact).
- [ ] **Step 6: Commit.** `git add client/src/App.vue && git commit -m "feat(ui): convert top nav to collapsible left sidebar shell"`.

---

### Tasks 4–9: Restyle each view (via `vue-expert`, parallelizable)

Tasks 4–9 each depend only on Tasks 2 & 3, edit a distinct file, and only *read* `design-system.css` — so they may be dispatched **in parallel**. Each follows the same 4-step cycle; the per-view specifics differ.

**Shared cycle per view:**
- [ ] **Step 1: Dispatch `vue-expert`** with `references/view-redesign-brief.md` (the rule + rationalization table + red flags + the matching "Per-view notes" bullet) + `design-system.md` + the target file. Restyle to `.page-header` + `.card`s + tokens; keep all logic/data/charts.
- [ ] **Step 2: Verify render + no regressions.** Playwright navigate the route; no new console errors; after-screenshot; run the view's behavior spot-check (below).
- [ ] **Step 3: Token scan.** Run the grep from `references/verification.md` §5 against the file — Expected: no hard-coded hex/px/font in the view.
- [ ] **Step 4: Commit.** `git add <file> && git commit -m "feat(ui): restyle <view> to SaaS design system"`.

| Task | View file | Route | Behavior spot-check (Step 2) |
|------|-----------|-------|------------------------------|
| **4: Dashboard** (flagship) | `client/src/views/Dashboard.vue` | `/` | KPI stat-tiles, status donut, trend/category charts, top-products table, backlog all present; a filter change updates the data; a product row still opens `ProductDetailModal`. Match mockup density. |
| **5: Inventory** | `client/src/views/Inventory.vue` | `/inventory` | Search filters rows; stock-status sort order preserved; row click opens `InventoryDetailModal`. |
| **6: Orders** | `client/src/views/Orders.vue` | `/orders` | Grouped-by-status rendering intact; status `.badge` colors correct; sort order unchanged. |
| **7: Demand** | `client/src/views/Demand.vue` | `/demand` | Forecasts grouped by trend; computed change % still shown. |
| **8: Spending (Finance)** | `client/src/views/Spending.vue` | `/spending` | Cost stat-tiles, monthly/revenue charts, category + transactions render; period filter still recomputes client-side; cost row opens `CostDetailModal`. |
| **9: Reports** | `client/src/views/Reports.vue` | `/reports` | Quarterly + monthly-trend charts/stats render; Options-API + direct-axios behavior preserved (restyle only). |

---

### Task 10: Full verification & handoff

**Files:** none (verification + report only)

**Interfaces:**
- Consumes: all prior tasks.
- Produces: a before/after report and a branch ready for review.

- [ ] **Step 1: Full sweep.** Run the entire `.claude/skills/saas-redesign/references/verification.md` checklist across all 6 routes (render, console, behavior, collapse, consistency).
- [ ] **Step 2: Regression grep.** Run: `grep -rnE "#[0-9a-fA-F]{3,6}|[0-9]+px|font-family" client/src/views client/src/App.vue` — Expected: only expected matches (flag any rogue values for a follow-up `vue-expert` fix).
- [ ] **Step 3: Before/after.** Assemble baseline vs after screenshots per route; confirm Dashboard matches the approved mockup.
- [ ] **Step 4: Report.** Summarize changed files, screenshots, and any residual issues.
- [ ] **Step 5: Handoff.** Offer merge/PR via `superpowers:finishing-a-development-branch`. Do not merge or push unless the user asks.

---

## Self-Review

**Spec coverage:** sidebar shell (Task 3) ✓ · collapsible/icons-only responsive (Task 3, Steps 1&4) ✓ · design system via frontend-design (Task 2) ✓ · all 6 views (Tasks 4–9) ✓ · card layouts/consistent spacing (Tasks 3–9 via tokens) ✓ · restyle-only guardrails (Global Constraints + every task) ✓ · verification (Tasks 2–10) ✓ · mockup reuse as seed + acceptance (Tasks 2, 3, 4, 10) ✓ · out-of-scope items excluded (Global Constraints) ✓.

**Placeholder scan:** no TBD/TODO; every step has a concrete command or dispatch target; creative code is intentionally delegated to `frontend-design`/`vue-expert` with brief files named exactly.

**Type consistency:** the design-system class/token names defined in Task 2's Interfaces are the same names consumed in Tasks 3–9; the `localStorage['sidebar-collapsed']` key is defined in Task 3 and re-checked in Task 3 Step 4 and Task 10.
