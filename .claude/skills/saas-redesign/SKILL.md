---
name: saas-redesign
description: Redesign this repo's Vue 3 client into a SaaS-style UI with a collapsible vertical sidebar instead of the top nav, consistent spacing tokens, and a polished professional look. Use when asked to "redesign the UI", "move the nav to a sidebar", "make it look like a SaaS app", or "unify spacing". Only for this inventory-management repo.
---

# saas-redesign

Orchestrates a shell-and-tokens redesign of `client/`. You (the main session) orchestrate; every `.vue`
file is written by the `vue-expert` subagent. Scope is the shell (sidebar, layout, a few shared
components) plus the global classes in `App.vue`. Per-view polish is out of scope.

## Fixed rules

1. Every `.vue` file you create or significantly modify is delegated to `vue-expert` (Agent tool,
   `subagent_type: vue-expert`). Do not write `.vue` files yourself.
2. `vue-expert` cannot spawn subagents and sees only its brief. Each brief must contain the full text
   of `references/tokens.md` and `references/sidebar-spec.md` (whichever apply), the exact file list,
   and the acceptance checks. Never say "see the spec".
3. Design system: slate palette, status colors green/blue/yellow/red, no emojis in the UI, inline SVG
   icons. Values live in `references/tokens.md`; do not invent others.
4. i18n applies to shell labels only. No hardcoded labels in the sidebar; new keys go in every locale
   file in `client/src/locales`. Translating `Reports.vue` is out of scope.
5. This repo is public. Do not touch `client/.npmrc` or the lockfile; commit no credentials or
   internal hostnames.
6. Work on a new local branch (`git checkout -b feat/saas-redesign`). Only modified tracked files count
   as a dirty tree; untracked files (for example `docs/architecture.html`) are ignored. All GitHub
   operations use the GitHub MCP tools (`mcp__github__*`).

## Definition of done

`npm run build` in `client/` is green, the screenshots in phase 5 are reviewed with no blockers, and
`code-reviewer` reports no blocking findings. There are no frontend tests (`client/package.json` has
no test script) and a UI change cannot affect the backend tests, so do not claim "tests green"; say
that explicitly in the final report.

## How phases run

Continue automatically between phases. Stop only on a blocker: dirty tracked tree, failing build, or
Playwright unavailable after one retry.

## Phase 1: Inspect

1. `git status -s`: stop if any tracked file is modified (see rule 6). Then create the branch.
2. Read `client/src/App.vue`, `client/src/main.js` (router), `client/src/locales/*`,
   `client/src/components/FilterBar.vue`, `ProfileMenu.vue`, `LanguageSwitcher.vue`.
3. Confirm the routes are still `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`. If
   they changed, update the route table you hand to `vue-expert` and tell Marco.

## Phase 2: Tokens

Paste the `:root` block from `references/tokens.md` at the top of the unscoped `<style>` block in
`client/src/App.vue`. This is a CSS-only edit inside an existing file with no template or logic
change, so you may do it directly. Do not run `frontend-design`; the tokens are fixed.

## Phase 3: Shell (delegate to vue-expert)

Brief it with `references/sidebar-spec.md` and `references/tokens.md` in full. Ask it to:

- create `client/src/components/AppSidebar.vue`;
- rewrite the template and layout styles of `client/src/App.vue` (remove the top nav, add the mobile
  top bar and overlay, own the `collapsed` and `drawerOpen` state);
- add `compact` and `placement` to `ProfileMenu.vue` and `LanguageSwitcher.vue`;
- make `FilterBar.vue` wrap and replace its `top: 70px`;
- add the new i18n keys to every locale file.

Acceptance: the app builds and every item in the spec's sections is implemented.

## Phase 4: Inherit (delegate to vue-expert)

Brief it with `references/tokens.md`. Ask it to migrate only the global unscoped classes in `App.vue`
(`.card`, `.card-header`, `.card-title`, `.stat-card`, `.stat-label`, `.stat-value`, `table`, `th`,
`td`, `.badge`, `.page-header`, `.loading`, `.error`) and the layout paddings to tokens, using the
migration table. It must not edit any view or any scoped style unless phase 5 shows breakage.

## Phase 5: Verify

1. Start the servers with the `/start` skill, or `cd server && uv run python main.py` and
   `cd client && npm run dev` (frontend on `http://localhost:3000`, API on `http://localhost:8001`).
2. `cd client && npm run build`. Fix failures (via `vue-expert` for `.vue` files).
3. Delegate screenshots to `vue-expert` (it has the Playwright MCP tools) so the images stay out of
   your context. Brief: for each route in `/`, `/inventory`, `/orders`, `/spending`, `/demand`,
   `/reports` and each width in 1440, 1024, 390, open the page, take a screenshot, and report only
   findings against this checklist:
   - no page-level horizontal scroll (tables may scroll inside `.table-container`);
   - the sidebar does not cover content; the drawer opens and closes at 390;
   - the collapsed sidebar shows tooltips and its dropdowns are not clipped;
   - modals stay centered; `FilterBar` is intact and stays visible while scrolling.
4. If Playwright fails to connect, retry once. If it still fails, stop and tell Marco; never skip this
   phase.
5. Fix every finding (via `vue-expert`), rebuild, and re-check the affected routes. Touching a view's
   scoped CSS is allowed here, and only for a finding.

## Phase 6: Review

Run the `code-reviewer` subagent on the diff. Fix blocking findings through `vue-expert`. Then open a
PR with the GitHub MCP tools. Report: what changed, that there are no frontend tests, the screenshot
findings and how they were resolved, and that `Backlog.vue` and `Reports.vue` text are untouched.

## Out of scope

Per-view polish, component and modal restyling, `Backlog.vue`, translating `Reports.vue`, frontend
test setup, any change under `server/`.
