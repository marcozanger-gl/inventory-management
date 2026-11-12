# Design tokens

Fixed values for the saas-redesign skill. Values come from the styles currently in `client/src/App.vue`
so nothing shifts visibly, except where the migration table says otherwise. Do not generate or
reinterpret tokens at run time; edit this file instead.

## The `:root` block

Paste this at the very top of the unscoped `<style>` block in `client/src/App.vue`, before `* { ... }`.

```css
:root {
  /* Spacing, 4px base */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-7: 32px;
  --space-8: 48px;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Colors: surfaces and text */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-text: #1e293b;
  --color-text-strong: #0f172a;
  --color-text-muted: #64748b;
  --color-border: #e2e8f0;
  --color-border-hover: #cbd5e1;
  --color-hover-bg: #f1f5f9;
  --color-accent: #2563eb;
  --color-accent-bg: #eff6ff;

  /* Colors: status */
  --color-success: #059669;
  --color-warning: #ea580c;
  --color-danger: #dc2626;
  --color-info: #2563eb;

  /* Sidebar */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --sidebar-bg: #0f172a;
  --sidebar-border: #1e293b;
  --sidebar-text: #cbd5e1;
  --sidebar-text-muted: #94a3b8;
  --sidebar-hover-bg: #1e293b;
  --sidebar-active-bg: #2563eb;
  --sidebar-active-text: #ffffff;
  --topbar-height: 56px;

  /* Typography */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 0.938rem;
  --text-md: 1.125rem;
  --text-lg: 1.375rem;
  --text-xl: 1.875rem;
  --text-2xl: 2.25rem;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;
  --tracking-tight: -0.025em;

  /* Shadows */
  --shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.06);

  /* z-index scale: sticky content < sidebar < overlay < dropdown < modal */
  --z-sticky: 90;
  --z-sidebar: 100;
  --z-overlay: 200;
  --z-dropdown: 300;
  --z-modal: 400;
}
```

## Migration table (global classes in `App.vue` only)

Replace the old value with the token. Round any other spacing value to the nearest space token;
a tie rounds up.

| Old value | Token |
|---|---|
| `0.25rem` | `var(--space-1)` |
| `0.313rem` | `var(--space-1)` |
| `0.375rem`, `0.5rem` | `var(--space-2)` |
| `0.625rem`, `0.75rem` | `var(--space-3)` |
| `0.875rem`, `1rem` | `var(--space-4)` |
| `1.25rem` | `var(--space-5)` |
| `1.5rem` | `var(--space-6)` |
| `2rem` | `var(--space-7)` |
| `3rem` | `var(--space-8)` |
| radius `6px` | `var(--radius-sm)` |
| radius `8px` | `var(--radius-md)` |
| radius `10px` | `var(--radius-lg)` (grows by 2px, accepted) |
| `#f8fafc` | `var(--color-bg)` |
| `#ffffff`, `white` | `var(--color-surface)` |
| `#1e293b` | `var(--color-text)` |
| `#0f172a` | `var(--color-text-strong)` |
| `#64748b` | `var(--color-text-muted)` |
| `#e2e8f0` | `var(--color-border)` |
| `#cbd5e1` | `var(--color-border-hover)` |
| `#f1f5f9` | `var(--color-hover-bg)` |
| `#2563eb` | `var(--color-accent)` |
| `#eff6ff` | `var(--color-accent-bg)` |
| `#059669` | `var(--color-success)` |
| `#ea580c` | `var(--color-warning)` |
| `#dc2626` | `var(--color-danger)` |
| font sizes `0.75rem` to `2.25rem` | the `--text-*` token with the same value |
| `font-weight: 400/500/600/700` | the `--weight-*` token |
| `letter-spacing: -0.025em` | `var(--tracking-tight)` |
| box-shadow `0 1px 3px 0 rgba(0, 0, 0, 0.05)` | `var(--shadow-sm)` |
| box-shadow `0 4px 12px rgba(0, 0, 0, 0.06)` | `var(--shadow-md)` |

Leave alone: the badge background/text color pairs (`#d1fae5`/`#065f46` and the like), they are not
part of the token set; chart colors; anything inside a view's or component's scoped `<style>`.

## z-index

The shell uses the `--z-*` scale. Existing modals keep their current large z-index values (1000 and
2000 in component files); those are higher than every shell value, so the order still holds and the
modals are not edited. The sticky `FilterBar` uses `var(--z-sticky)` in place of its current `90`.
