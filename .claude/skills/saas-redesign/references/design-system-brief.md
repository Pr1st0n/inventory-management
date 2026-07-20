# Design-system brief (Phase 1 → `frontend-design`)

Hand these constraints to the `frontend-design` skill. Its job: author a distinctive, professional
**SaaS dashboard** design system for this Factory Inventory app and persist it as reusable CSS.

## Seed
Start from the approved mockup `docs/redesign-mockup.html` (dark ink sidebar + light content,
Bricolage Grotesque / Figtree / JetBrains Mono type, cobalt accent, card/tile/table patterns). Refine
and pressure-test it — do not ship the mockup verbatim.

## Hard constraints
- **Professional B2B dashboard**, not maximalist. "Wow" comes from execution quality, not gimmicks.
- **No emojis** anywhere (project rule).
- **Distinctive typography** — no Inter/Roboto/Arial/system defaults. Load web fonts via `<link>` or
  `@import` with system fallbacks.
- Must accommodate the **hand-built SVG charts** and the **status color semantics**
  green=positive/delivered, blue=info/shipped, yellow/amber=warning/processing, red=danger/backordered.
- Light content surfaces; sidebar may be dark (per mockup). Accessible contrast (WCAG AA).

## Deliverables (persist these)
1. **`client/src/styles/design-system.css`** — a `:root` token layer + component classes:
   - **Tokens:** color (bg, surface, border, ink, muted, accent, + status set), typography (font
     families, type scale, weights, line-heights), spacing scale (4pt), radius, shadow/elevation,
     transitions, z-index, sidebar dimensions (expanded + collapsed widths).
   - **Component classes:** `.card`, `.card__head`, `.stat-tile`, `.data-table`, `.btn`
     (`.btn-primary`/`.btn-ghost`), `.badge`/`.status`, `.page-header`, `.sidebar`, `.nav-item`,
     `.filter-bar`, `.fselect`.
2. **`client/src/styles/design-system.md`** — one page documenting the tokens and how to use each
   component class (so per-view subagents apply them consistently).
3. Import `design-system.css` **globally** (in `client/src/main.js`, before app-specific styles).

## Acceptance
The token set + classes are sufficient to build the sidebar shell and all 6 views with **zero
hard-coded colors/spacing/fonts** in view files. The redesigned Dashboard should read as the same
product as the approved mockup.
