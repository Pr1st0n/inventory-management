# View redesign brief (Phases 3–4 → `vue-expert`)

Use this as the brief for each `vue-expert` subagent. Dispatch **one subagent per file**. Every
subagent gets: this brief + the target file + `client/src/styles/design-system.md`.

## The one rule: restyle only

**Violating the letter of this rule is violating its spirit.** You are changing how the UI *looks*,
never how it *works*.

**You MAY change:** template layout/markup for styling, CSS/`<style>` blocks, class names, adding
design-system classes and wrapper elements, reordering presentational markup.

**You MUST NOT change:** `api.*` calls, composables (`useFilters`, `useI18n`, `useAuth`),
`computed`/`watch`/`ref` logic, router paths or `<router-link>` targets, **i18n keys** (keep `t('…')`
calls identical), emitted events, prop names/contracts, or `v-for` `:key` expressions. Use design-system
**tokens/classes** — never hard-code a hex color, px spacing, or font-family in a view. No emojis.

### Rationalization table — do not negotiate

| Excuse | Reality |
|--------|---------|
| "This computed is messy, I'll tidy it while restyling." | Out of scope. Touch logic → behavior risk. Leave it. |
| "I'll rename this i18n key to something cleaner." | Keys are a contract with the locale files. Identical keys only. |
| "Index keys in this `v-for` are simpler." | Keys affect reactivity/DOM reuse. Keep the existing key expressions. |
| "A quick hard-coded `#2f6bed` is fine here." | Then the system isn't the source of truth. Use `var(--accent)`. |
| "The filter wiring would be cleaner if I…" | `FilterBar`/`useFilters` wiring is off-limits. Restyle around it. |
| "I'll just inline this SVG chart differently." | Keep chart data/logic; restyle container/colors via tokens only. |
| "It's basically the same behavior." | "Basically" = a diff in behavior. Preserve it exactly. |

### Red flags — STOP
- Editing anything inside `<script>` other than presentational-only refs you added.
- Typing a `#hex`, a raw `px` spacing value, or a `font-family` in a view.
- Changing a `t('...')` key, a route path, an `emit(...)`, or a prop name.
- Removing/altering a `:key` on a `v-for`.

Any red flag → revert that change and use a token/class instead.

## What "done" looks like per view
- A consistent `.page-header` (title + optional subtitle/actions).
- Content organized into `.card`s with consistent spacing tokens; KPIs as `.stat-tile`s.
- Tables use `.data-table`; statuses use `.status`/`.badge` with the correct status color.
- Charts kept (same data), reframed in cards, colors from tokens.
- No layout regressions at desktop; content flows within the shell's content area.

## Shell (Phase 3, `App.vue`)
- Replace the sticky **top nav** with a persistent **left vertical sidebar**: brand at top; the same 6
  `<router-link>`s (same paths, same i18n labels) as vertical `.nav-item`s with active states;
  `LanguageSwitcher` + `ProfileMenu` docked at the bottom (keep their emits/props intact).
- Layout: CSS grid `sidebar | main`. `main` = a sticky top bar holding the page title + the global
  `<FilterBar />` (unchanged wiring), above a scrollable, padded `<router-view />`.
- **Collapsible / icons-only:** a toggle collapses the sidebar to an icon rail (labels hidden, show
  `title`/tooltip on hover); **auto-collapse below a breakpoint** (~1024px), off-canvas/overlay on
  mobile (~640px). Persist the collapsed state in `localStorage` (match the app's existing persistence
  style used by `useI18n`). Keep all existing modals/tasks state in `App.vue` working.

## Per-view notes
- **Dashboard** (`/`) — flagship. KPI `.stat-tile` row, order-status donut, trend + category charts,
  top-products table, backlog section. Match the approved mockup's density and hierarchy.
- **Inventory** — searchable table (`.data-table`), stock-status sort preserved; row → `InventoryDetailModal`.
- **Orders** — grouped-by-status list; status `.badge`s; keep sort order.
- **Demand** — forecasts grouped by trend; keep computed change %.
- **Spending** (Finance) — cost summary `.stat-tile`s, monthly/revenue charts, category + transactions.
- **Reports** — quarterly + monthly-trend charts/stats. Note: uses Options API + direct axios; keep
  that behavior, restyle only.
