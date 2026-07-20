# Design System — Factory Inventory (SaaS redesign)

One stylesheet: `client/src/styles/design-system.css`. It is imported globally in
`main.js`, so every view already has the tokens and component classes below.

**Rule for views:** never hard-code a color, spacing value, radius, shadow, or
font family. Reference a `var(--token)` or apply a component class. Fonts load
automatically via an `@import` at the top of the CSS (no HTML `<link>` needed).

Aesthetic: deep-ink sidebar + light content, **cobalt** accent, **Bricolage
Grotesque** display / **Figtree** body / **JetBrains Mono** numerics. No emojis.

---

## Tokens (`:root`)

### Color — content surfaces
| Token | Use |
| --- | --- |
| `--bg` | app canvas behind cards |
| `--surface` | card / panel background |
| `--surface-2` | subtle inset — row hover, tracks, bars |
| `--surface-3` | chart gridlines, muted chips |
| `--border` | hairline dividers / card edges |
| `--border-strong` | interactive control borders |

### Color — text ink
`--ink` (headings/values) · `--ink-2` (body, table cells) · `--muted` (secondary,
AA on surface) · `--faint` (decorative labels, placeholders) · `--on-accent` (text on accent fills).

### Color — sidebar (deep ink)
`--nav-bg`, `--nav-bg-2` (gradient) · `--nav-border` · `--nav-text`,
`--nav-text-dim`, `--nav-text-strong` · `--nav-active` · `--nav-hover`,
`--nav-active-bg`.

### Color — accent (cobalt)
`--accent`, `--accent-hover`, `--accent-pressed` · `--accent-soft` (tinted bg) ·
`--accent-glow` (shadows/rings).

### Color — status semantics
| Token pair | Meaning | Order/stock state |
| --- | --- | --- |
| `--success` / `--success-soft` | green — positive | Delivered · In stock |
| `--info` / `--info-soft` | blue — informational | Shipped |
| `--warning` / `--warning-soft` | amber — attention | Processing · Low stock |
| `--danger` / `--danger-soft` | red — critical | Backordered · Out of stock |

### Typography
- Families: `--font-display` (Bricolage Grotesque), `--font-sans` (Figtree),
  `--font-mono` (JetBrains Mono). Body defaults to `--font-sans`.
- Size scale: `--text-2xs` 10.5 · `--text-xs` 11.5 · `--text-sm` 13 · `--text-base` 13.5 ·
  `--text-md` 15 · `--text-lg` 18 · `--text-xl` 22 · `--text-2xl` 30 · `--text-3xl` 38 (px).
- Weights: `--fw-regular/medium/semibold/bold/extrabold` (400–800).
- Line-height: `--lh-tight/snug/normal/relaxed`. Tracking: `--tracking-tight/wide/wider`.
- Use `--font-display` for titles/KPI values, `--font-mono` + `font-variant-numeric: tabular-nums` for figures.

### Spacing — 4pt scale
`--space-1` 4 · `-2` 8 · `-3` 12 · `-4` 16 · `-5` 20 · `-6` 24 · `-7` 28 · `-8` 32 ·
`-10` 40 · `-12` 48 · `-16` 64 (px). `--space-0` is 0.

### Radius / elevation / motion / z / layout
- Radius: `--radius-xs` 6 · `-sm` 8 · `-md` 10 · `-lg` 12 · `-xl` 16 · `-full` 999.
- Shadow: `--shadow-1/2/3` (elevation), `--shadow-accent`, `--ring` (focus glow).
- Motion: `--ease-out`; `--transition-fast/base/slow`.
- Z-index: `--z-base/sticky/dropdown/overlay/modal/toast`.
- Layout: `--sidebar-w` (250px expanded), `--sidebar-w-collapsed` (76px icons-only),
  `--topbar-h`, `--content-max` (1280px).

---

## Layout shell

```html
<div class="app-shell">            <!-- add .is-collapsed for icons-only -->
  <aside class="sidebar"> … </aside>
  <div class="app-main">
    <header class="page-header"> … </header>
    <div class="filter-bar"> … </div>
    <main class="content"> … </main>
  </div>
</div>
```

`.app-shell` is a 2-col grid (`--sidebar-w` + fluid). Add `.is-collapsed` to swap
to `--sidebar-w-collapsed`. `.app-main` handles overflow for wide tables/charts.

---

## Component classes

### Sidebar — `.sidebar` / `.nav-item`
```html
<aside class="sidebar">
  <div class="sidebar__brand">
    <span class="sidebar__logo"><svg …/></span>
    <span class="sidebar__brand-name"><b>Northwind Ops</b><span>Inventory</span></span>
  </div>
  <nav class="nav-group">
    <div class="nav-label">Operations</div>
    <router-link to="/" class="nav-item"><svg …/><span>Overview</span></router-link>
    <router-link to="/orders" class="nav-item">
      <svg …/><span>Orders</span><span class="nav-item__count">250</span>
    </router-link>
  </nav>
  <div class="sidebar__foot"> … </div>
</aside>
```
Active state: `.nav-item.is-active` (also matches `.active` and vue-router's
`.router-link-active`). Collapsed: adding `.is-collapsed` to `.sidebar` hides
labels/counts and centers icons.

### Page header — `.page-header`
```html
<header class="page-header">
  <div class="page-header__titles">
    <h1 class="page-header__title">Overview</h1>
    <p class="page-header__subtitle">Inventory & order performance</p>
  </div>
  <div class="page-header__actions">
    <button class="btn btn-primary"><svg …/>New Order</button>
  </div>
</header>
```

### Buttons — `.btn` + `.btn-primary` / `.btn-ghost`
```html
<button class="btn btn-primary">Save</button>
<button class="btn btn-ghost">Export</button>
<button class="btn btn-ghost btn-sm">Compact</button>
```
`.btn` is the base (layout/type); always pair with a variant. Inline `<svg>` is
auto-sized to 15px and inherits `currentColor`.

### Filter bar — `.filter-bar` / `.fselect`
```html
<div class="filter-bar">
  <span class="filter-bar__label">Filters</span>
  <select class="fselect"> … </select>            <!-- native select, styled -->
  <select class="fselect is-armed"> … </select>   <!-- add .is-armed when narrowing -->
  <button class="filter-bar__reset">Reset</button>
</div>
```
`.fselect` styles a native `<select>` (custom chevron, no browser chrome) or a
composed `<div>` (use `.fselect__key` for the inline label). Add `.is-armed`
(alias `.armed`) when the filter is set to a non-default value.

### Card — `.card` / `.card__head` / `.card__body`
```html
<div class="card">
  <div class="card__head">
    <h3 class="card__title">Revenue Trend</h3>
    <span class="card__meta">2025 · monthly</span>
  </div>
  <div class="card__body"> …chart / content… </div>
</div>
```
For a card whose body is a table, drop `.card__body` and put the `<table
class="data-table">` directly inside `.card`.

### Stat tile — `.stat-tile`
```html
<div class="stat-tile">
  <div class="stat-tile__head">
    <span class="stat-tile__label">Inventory Value</span>
    <span class="stat-tile__icon stat-tile__icon--info"><svg …/></span>
  </div>
  <div class="stat-tile__value">$2.84M</div>
  <div class="stat-tile__foot">
    <span class="delta delta--up"><svg …/>4.2%</span>
    <span class="stat-tile__since">vs last month</span>
  </div>
  <svg class="spark" viewBox="0 0 120 34" preserveAspectRatio="none"> … </svg>
</div>
```
Icon tint variants: `--info` / `--success` / `--warning` / `--danger`. Trend
pill: `.delta--up` (green) / `.delta--down` (red). `.spark` sizes the sparkline SVG.
Lay tiles out with `<div class="grid grid--kpis">`.

### Data table — `.data-table`
```html
<table class="data-table">
  <thead><tr><th>Order</th><th class="is-numeric">Value</th></tr></thead>
  <tbody>
    <tr>
      <td><span class="data-table__code">ORD-2041</span></td>
      <td class="is-numeric"><span class="data-table__num">$18,420</span></td>
    </tr>
  </tbody>
</table>
```
Helpers: `.is-numeric` (right-align cell), `.data-table__num` (mono tabular
figures), `.data-table__code` (mono IDs/SKUs), `.data-table__strong` (emphasis).

### Badge — `.badge`
Count/label pill: `<span class="badge badge--warning">Low</span>`. Variants:
`--accent`, `--success`, `--info`, `--warning`, `--danger` (default is neutral).

### Status — `.status`
Dot + label pill for states: `<span class="status status--success">Delivered</span>`.
Variants: `--success`, `--info`, `--warning`, `--danger`, `--neutral`. See the
status mapping table above.

### Chart primitives
- Legend: `.legend` > `.legend__item` ( `.legend__swatch` + text + `.legend__value` ).
- Progress bar: `.progress` > `.progress__fill` (set `width` inline/computed).
- Donut label: `.donut-center` wraps the SVG; `.donut-center__value` (b + span)
  overlays the centered figure.

### Grid helpers
`.grid` (gap only) + a template modifier: `.grid--kpis` (4-up), `.grid--2`
(equal halves), `.grid--wide-narrow` (1.7fr / 1fr). All collapse to one column
under 900px.

### Motion
Add `.rise` to a block for a staggered fade-up on load; stagger siblings with an
inline `animation-delay`. Respects `prefers-reduced-motion`.
