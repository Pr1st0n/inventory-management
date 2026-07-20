# Verification checklist (Phase 5 → Playwright MCP)

Run against `http://localhost:3000` (frontend) with the backend up on `:8001`. Do not claim the
redesign is done until every check passes with evidence.

## 1. Build / boot
- [ ] Frontend compiles with no Vite errors; no missing-import or template-parse errors in the dev log.
- [ ] `design-system.css` is imported globally and loads (fonts resolve).

## 2. Per-route render (all 6)
For each of `/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`:
- [ ] Route renders inside the new sidebar shell; sidebar active state matches the route.
- [ ] **No new console errors** (`browser_console_messages`) vs. the baseline capture.
- [ ] Capture an "after" screenshot; compare against the baseline + the mockup for the Dashboard.

## 3. Behavior preserved (spot-check, not just visual)
- [ ] Change a filter in `FilterBar` (e.g. Warehouse) → data updates on Dashboard/Orders (wiring intact).
- [ ] Language toggle (EN/JA) still switches labels (i18n keys unchanged).
- [ ] A row/card that opened a modal before still opens it (emits/props intact).
- [ ] Nav links route correctly; deep-linking to each path works (history mode).

## 4. Sidebar collapse
- [ ] Manual toggle collapses to icons-only (labels hidden, tooltips on hover) and restores.
- [ ] Auto-collapses below ~1024px; off-canvas/overlay below ~640px (`browser_resize` to test).
- [ ] Collapsed state persists across reload (localStorage).

## 5. Design-system consistency
- [ ] Scan view files for rogue hard-coded styles:
      `grep -rnE "#[0-9a-fA-F]{3,6}|[0-9]+px|font-family" client/src/views client/src/App.vue`
      — expected matches only in `design-system.css`; flag any in views.
- [ ] Status colors render correctly (delivered=green, shipped=blue, processing=amber, backordered=red).
- [ ] No emojis introduced.

## 6. Report
- [ ] Summarize changed files, before/after screenshots per route, and any residual issues.
- [ ] Leave work on the feature branch; offer merge/PR via `finishing-a-development-branch`. Commit only
      if the user asks (no `Co-Authored-By`).
