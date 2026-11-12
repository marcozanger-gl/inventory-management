# Sidebar specification

Contract for the left sidebar that replaces the top nav. A `vue-expert` brief must include this whole
file plus `tokens.md`. Use only token names defined in `tokens.md`. Follow `client/CLAUDE.md`:
Composition API with `export default { setup() }`, unique keys in `v-for`, no emojis, inline SVG icons.

## Component

`client/src/components/AppSidebar.vue`

- Props: `collapsed` (Boolean), `open` (Boolean, drawer open on mobile).
- Emits: `toggle-collapse`, `close`.
- `App.vue` owns the state: `collapsed` (persisted) and `drawerOpen` (not persisted), and renders the
  mobile top bar and the overlay.
- Routes (label key, path, icon):

| Label key | Path | Icon |
|---|---|---|
| `nav.overview` | `/` | layout |
| `nav.inventory` | `/inventory` | package |
| `nav.orders` | `/orders` | clipboard |
| `nav.finance` | `/spending` | coins |
| `nav.demandForecast` | `/demand` | trend |
| `nav.reports` | `/reports` | file |

Use `<router-link>` and style `.router-link-exact-active` for the active state; do not keep the manual
`:class="{ active: ... }"`. Backlog is not a route and gets no link.

## Structure, top to bottom

1. Header: company name (`nav.companyName`) and, when expanded, the subtitle (`nav.subtitle`). A
   collapse button on desktop only.
2. Links: one per route, icon then label. When collapsed, the label is visually hidden but stays in
   the DOM for screen readers, and a tooltip (the label) shows on hover and on focus.
3. Footer: `LanguageSwitcher` and `ProfileMenu`, using their compact trigger when collapsed.

## States

| State | When | Behavior |
|---|---|---|
| Expanded | desktop, `collapsed` false | width `var(--sidebar-width)`, icons and labels |
| Collapsed | desktop, `collapsed` true | width `var(--sidebar-width-collapsed)`, icons only, tooltips |
| Drawer | below 768px | off-canvas, slides in from the left, width `min(280px, 85vw)`, always shows labels |

Breakpoint: `768px`. At 768px and above the sidebar is `position: sticky; top: 0; height: 100vh`, and
the page is a row: sidebar then content. Below 768px the sidebar is `position: fixed`, translated
`-100%` when closed, and a top bar of height `var(--topbar-height)` (hamburger, company name) is
`position: sticky; top: 0`.

Colors: `--sidebar-bg` background, `--sidebar-text` for links, `--sidebar-hover-bg` on hover,
`--sidebar-active-bg` with `--sidebar-active-text` for the active link, `--sidebar-border` for
dividers. Stacking: sidebar `var(--z-sidebar)`, overlay `var(--z-overlay)`.

## Persistence

Key `inventory.sidebar.collapsed`, value `"1"` or `"0"`. Wrap every `localStorage` read and write in
`try/catch`; if it throws, the sidebar still works, just without memory. Only applies on desktop.

## Drawer behavior

- Hamburger in the top bar opens it; overlay click, Escape, and picking a link close it.
- While open: lock body scroll (restore on close), trap focus inside the drawer (or set `inert` on the
  main content), and move focus to the first link.
- On close: return focus to the hamburger.
- The hamburger has a translated `aria-label` (`nav.openMenu` / `nav.closeMenu`).

## Accessibility

- `<nav>` with `aria-label` from `nav.primary`.
- The active link gets `aria-current="page"` from vue-router's exact-active class.
- Visible keyboard focus on every control (`outline` using `--color-accent` with an offset).
- Collapse button has `aria-expanded` and a translated `aria-label` (`nav.collapse` / `nav.expand`).
- Respect `prefers-reduced-motion`: no slide or width transitions when it is set.

## i18n

Add to every locale file in `client/src/locales`: `nav.reports`, `nav.primary`, `nav.openMenu`,
`nav.closeMenu`, `nav.collapse`, `nav.expand`. Do not translate `Reports.vue` itself.

## Dropdown components

`ProfileMenu.vue` and `LanguageSwitcher.vue` open downward and to the right today, which clips them
at the foot of the sidebar. Add to both:

- `compact` (Boolean, default `false`): trigger shows only the icon or avatar, no text.
- `placement` (String, default `'bottom-end'`, which keeps today's behavior). The sidebar passes
  `'top-start'` when expanded and `'right-end'` when collapsed (opens to the right of the sidebar,
  bottom aligned). Dropdown z-index is `var(--z-dropdown)`.

## FilterBar

`client/src/components/FilterBar.vue`:

- The filter row wraps (`flex-wrap: wrap`) so it never overflows at 390px.
- `position: sticky` keeps working: `top: 0` at 768px and above (there is no top bar), and
  `top: var(--topbar-height)` below 768px. Replace the hardcoded `top: 70px`. z-index `var(--z-sticky)`.
- Revisit its `max-width` and centering so it lines up with `.main-content` next to the sidebar.

## App.vue layout

- `.app` becomes a row at 768px and above (`display: flex; flex-direction: row`) and a column below.
- Remove `.top-nav`, `.nav-container`, `.logo`, `.nav-tabs` and their rules.
- `.main-content` keeps `max-width: 1600px`, uses `flex: 1; min-width: 0`, and its padding comes from
  `--space-6` and `--space-7` (so wide tables scroll inside `.table-container`, not the page).

## Icons

All icons: `viewBox="0 0 24 24"`, `fill="none"`, `stroke="currentColor"`, `stroke-width="2"`,
`stroke-linecap="round"`, `stroke-linejoin="round"`, size 20px, `aria-hidden="true"`.

```html
<!-- layout (Overview) -->
<rect x="3" y="3" width="7" height="9"/><rect x="14" y="3" width="7" height="5"/>
<rect x="14" y="12" width="7" height="9"/><rect x="3" y="16" width="7" height="5"/>

<!-- package (Inventory) -->
<path d="M21 8l-9-5-9 5v8l9 5 9-5V8z"/><path d="M3 8l9 5 9-5"/><path d="M12 13v8"/>

<!-- clipboard (Orders) -->
<rect x="8" y="2" width="8" height="4" rx="1"/>
<path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/>
<path d="M9 12h6M9 16h6"/>

<!-- coins (Finance) -->
<circle cx="12" cy="12" r="9"/><path d="M12 6v12"/>
<path d="M15 9c-.6-.8-1.700-1.200-3-1.200-1.600 0-2.800.7-2.800 1.900 0 2.600 5.800 1.300 5.800 3.900 0 1.200-1.200 1.900-2.900 1.900-1.300 0-2.400-.4-3.100-1.300"/>

<!-- trend (Demand forecast) -->
<path d="M3 17l6-6 4 4 8-8"/><path d="M15 7h6v6"/>

<!-- file (Reports) -->
<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
<path d="M14 2v6h6"/><path d="M8 13h8M8 17h5"/>

<!-- menu (hamburger) -->
<path d="M4 6h16M4 12h16M4 18h16"/>

<!-- chevron-left (collapse; rotate 180 degrees when collapsed) -->
<path d="M15 6l-6 6 6 6"/>
```
