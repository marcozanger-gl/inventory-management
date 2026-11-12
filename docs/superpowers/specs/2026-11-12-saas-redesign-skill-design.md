# Design: `saas-redesign` skill

Date: 2026-11-12
Status: approved by Marco, pending written-spec review

## Goal

A Claude Code skill, specific to this repo, that redesigns the Vue 3 client into a modern SaaS-style
interface: vertical left sidebar instead of the top nav, consistent spacing through tokens, and a
polished professional look.

## Decisions

| Topic | Decision |
|---|---|
| Location | `.claude/skills/saas-redesign/` (this repo only; may name routes, files, palette) |
| Scope | Shell + tokens. Views inherit; no per-view polish |
| Sidebar | Collapsible (240px / 64px) on desktop, drawer below 768px |
| Verification | Build + Playwright screenshots, 6 routes x 3 widths; stop if Playwright is down |
| Tokens | Fixed now in `references/tokens.md`; `frontend-design` is not part of the flow |
| Alternative rejected | Plan + subagents without a skill (reviewer's objection: one-off work does not need a skill). Marco chose the skill |

## Layout

```
.claude/skills/saas-redesign/
├── SKILL.md                 # frontmatter, fixed rules, phases, verification
└── references/
    ├── tokens.md            # fixed design tokens + migration table
    └── sidebar-spec.md      # sidebar structure, states, a11y, icons
```

`verify.md` was merged into `SKILL.md` (a few commands and a checklist).

## SKILL.md

Frontmatter: `name: saas-redesign`; `description` triggers on "rediseñar la UI", "sidebar en vez de top
nav", "estilo SaaS", "unificar spacing"; states it is for this repo only.

### Fixed rules

1. Every `.vue` file created or significantly modified is delegated to `vue-expert`. The skill only
   orchestrates and writes briefs.
2. `vue-expert` cannot spawn subagents, so each brief must carry the full tokens, spec and file list.
3. Design system: slate palette, status colors green/blue/yellow/red, no emojis, inline SVG icons.
4. i18n applies to shell labels: no hardcoded labels in the sidebar; new keys go in every locale in
   `client/src/locales`. Add `nav.reports` (currently hardcoded in `App.vue`). Translating
   `Reports.vue` is out of scope.
5. Public repo: do not touch `client/.npmrc` or the lockfile; no credentials or internal hostnames.
6. Work on its own branch. Only modified tracked files count as a dirty tree; untracked files (for
   example `docs/architecture.html`) are ignored. GitHub operations use the GitHub MCP tools.

### Definition of done

Build green, screenshots reviewed with no blockers, `code-reviewer` reports no blocking findings.
There are no frontend tests (`client/package.json` has no test script); backend tests are not
affected by a UI change. The skill states this instead of claiming "tests green".

### Phases

Between phases the skill stops only on a blocker (Playwright down after one retry, build failing,
dirty tracked tree). Otherwise it continues.

1. **Inspect.** Read `App.vue`, the router (`client/src/main.js`), locales, and the hardcoded values
   in views. Check the tree.
2. **Tokens.** Paste the tokens from `tokens.md` into `:root` in `App.vue`. No generation step.
3. **Shell** (`vue-expert`, with `sidebar-spec.md`):
   - Create `AppSidebar.vue`; replace `.top-nav` and the column-flex `.app` layout in `App.vue`.
   - Active route via vue-router's `router-link-exact-active`.
   - `FilterBar` responsive: flex with wrap; replace `position: sticky; top: 70px` with
     `top: var(--topbar-height)` where relevant, and revisit its `max-width`/margin centering.
   - `ProfileMenu` and `LanguageSwitcher`: compact trigger; dropdowns open upward and outward so they
     are not clipped at the sidebar foot or in the 64px state.
   - Drawer: body scroll lock, focus trap (or `inert` on content), focus returned to the hamburger on
     close, translated `aria-label` on the hamburger.
4. **Inherit** (`vue-expert`, with `tokens.md`): migrate only the global unscoped classes in
   `App.vue` (`.card`, `.stat-card`, `table`, `.badge`, `.page-header`) and layout paddings to tokens.
   Scoped CSS in each view is touched only if screenshots show breakage. Components and modals keep
   their current look (modals use `Teleport` + `position: fixed`, unaffected by the new layout).
5. **Verify.**
   - Start servers (`/start`), run `npm run build` in `client/`.
   - Delegate screenshots to `vue-expert` (it has `mcp__playwright__*`); it returns findings only.
   - Routes: `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`.
   - Widths: 1440, 1024, 390.
   - Checks: no page-level horizontal scroll at any width (tables may scroll inside
     `.table-container`); sidebar does not cover content; modals stay centered; `FilterBar` intact;
     dropdowns not clipped.
   - If Playwright fails to connect, retry once, then stop and report. Never skip.
6. **Review.** Run `code-reviewer` on the diff, fix blockers, leave the PR ready (GitHub MCP).

`Backlog.vue` has no route or nav link today and is out of scope.

## references/tokens.md (fixed values)

- Spacing (`--space-1..8`): 4, 8, 12, 16, 20, 24, 32, 48 px.
- Radii: 6, 8, 12 px.
- Sidebar: 240px expanded, 64px collapsed. `--topbar-height: 56px` (mobile header).
- Colors: background `#f8fafc`, surface `#ffffff`, text `#0f172a` / `#64748b`, border `#e2e8f0`,
  sidebar `#0f172a`. Status colors keep green/blue/yellow/red.
- z-index scale: sidebar 100, overlay 200, dropdown 300, modal 400. Existing loose values
  (90/100/1000/2000) are migrated to this scale where they belong to the shell.
- Typography sizes and weights, shadows and borders: concrete values are written into `tokens.md`
  during implementation (derived from the values currently used in `App.vue` so nothing shifts
  visibly), not left to the executor.
- Migration table: old value to token, for the global classes in `App.vue`.

## references/sidebar-spec.md

- Structure: title at top; one link with an inline SVG icon per route in the middle;
  `LanguageSwitcher` and `ProfileMenu` at the bottom.
- States: expanded; collapsed (icons only, tooltip on hover/focus); drawer below 768px with overlay.
- Collapsed state persisted in `localStorage`, every read and write wrapped in `try/catch`; the UI
  works without it.
- Accessibility: `nav` with `aria-label`, `aria-current` (given by `router-link-exact-active`),
  visible focus, Escape closes the drawer.
- Six icons, one per route (overview, inventory, orders, finance, demand forecast, reports), no emojis.

## Risks and open points

- Phase 4 is bounded on purpose: roughly 168 hex colors and 277 rem/px values live in the views and
  more in components, so a full migration is explicitly not attempted.
- A dark sidebar on a light content area is the default; if a light sidebar is preferred, only the
  sidebar token changes.
