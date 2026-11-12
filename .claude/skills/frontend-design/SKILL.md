---
name: frontend-design
description: Redesign this Vue 3 app's UI into a modern SaaS-style interface with a dark vertical sidebar on the left instead of a top nav bar, consistent spacing, and polished light content cards. Use when asked to restyle, modernize, or redesign the frontend layout, navigation, or shared visual styles.
---

# Frontend Design: SaaS-Style Redesign

Turn the Factory Inventory Management frontend (Vue 3 + Vite, `client/`) into a polished SaaS-style dashboard. The target look is a **dark slate sidebar on the left with a light content area**.

## Scope

Change the app shell and shared styles only:

- `client/src/App.vue` (layout, navigation, global styles)
- `client/src/components/FilterBar.vue` (restyle to fit the new layout)
- Shared card, table, button and modal styling

Do **not** rewrite view internals (`client/src/views/*.vue`), chart logic, data loading or API calls. Views may get small class or spacing tweaks only when needed to match the shared styles.

## Hard constraints (from CLAUDE.md)

- **Delegate `.vue` edits to the `vue-expert` subagent.** Any time a `.vue` file is created or significantly modified, use it.
- **No emojis** anywhere in the UI. Use inline SVG icons for navigation.
- **Keep the custom SVG charts** and CSS Grid layouts. Add no charting, UI component or icon libraries.
- **Keep i18n.** Nav labels go through `useI18n` (`t('nav....')`). Add keys to both `client/src/locales/en.js` and `ja.js` for any new label. Keep the existing hard-coded "Reports" label working, or move it into both locale files.
- Keep unique `v-for` keys, not the array index.
- The repository is public: add no credentials, internal hostnames or private URLs.

## Layout

```
+-----------+--------------------------------------------+
| Sidebar   |  Page header: title + language + profile   |
| (dark)    |--------------------------------------------|
|           |  FilterBar                                 |
| logo      |--------------------------------------------|
| nav items |  <router-view /> inside max-width wrapper  |
|           |                                            |
| (footer)  |                                            |
+-----------+--------------------------------------------+
```

- Sidebar: fixed on the left, full height, width 240px.
- Content: fills the rest, scrolls independently, padding 24px to 32px, content max-width around 1400px.
- Move `LanguageSwitcher` and `ProfileMenu` out of the old top nav into a slim header bar at the top of the content area (or the sidebar footer). Keep their events (`show-profile-details`, `show-tasks`) wired.
- Keep all six routes: `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`.

## Design tokens

Define once as CSS variables on `:root` in `App.vue` and use them everywhere:

| Token | Value | Use |
|---|---|---|
| `--sidebar-bg` | `#0f172a` | Sidebar background |
| `--sidebar-text` | `#94a3b8` | Inactive nav items |
| `--sidebar-text-active` | `#ffffff` | Active nav item |
| `--sidebar-active-bg` | `#1e293b` | Active/hover nav item |
| `--accent` | `#3b82f6` | Active indicator, focus ring |
| `--bg` | `#f8fafc` | Page background |
| `--surface` | `#ffffff` | Cards, tables, modals |
| `--border` | `#e2e8f0` | Card and divider borders |
| `--text` | `#0f172a` | Primary text |
| `--text-muted` | `#64748b` | Secondary text |
| `--success` / `--info` / `--warning` / `--danger` | green / blue / yellow / red | Status colors |
| `--radius` | `10px` | Cards and inputs (6px for small controls) |
| `--shadow` | `0 1px 2px rgba(15,23,42,.06)` | Card shadow |

Spacing scale: 4, 8, 12, 16, 24, 32px only. Font: system sans stack, 14px body, 12px labels, 20px to 24px page titles.

## Components

- **Nav item:** 40px tall, 8px radius, icon (18px inline SVG) plus label, 12px gap. Active state: lighter background plus a 3px accent bar on the left. Hover: sidebar-active background. Use `router-link` and keep the `active` class logic.
- **Cards:** white surface, 1px border, radius token, subtle shadow, 20px padding. Replace ad-hoc card styles with one shared `.card` pattern.
- **Tables:** sticky-looking header row in muted uppercase 12px text, 1px row dividers, row hover `#f8fafc`.
- **Buttons and inputs:** 36px height, radius 6px, visible focus ring in the accent color.
- **Status badges:** pill shape, tinted background with matching text color.
- **FilterBar:** a single horizontal card under the header with labelled selects in a responsive grid.

## Responsiveness and accessibility

- Below 900px width, collapse the sidebar (icons only, or an off-canvas drawer) so content gets the full width.
- Every nav icon needs a text label or `aria-label`. Keep visible focus states and a text contrast ratio of at least 4.5:1.
- Use semantic elements: `<aside>`, `<nav>`, `<main>`, `<header>`.

## Workflow

1. Read `client/src/App.vue` and `FilterBar.vue`, and list which global classes the views depend on (for example `.main-content`, `.card`) so nothing breaks.
2. Use the `AskUserQuestion` tool if anything is unclear (icon style, header contents, breakpoint behavior).
3. Delegate the `.vue` edits to `vue-expert` with the tokens and layout above.
4. Verify in the browser at `http://localhost:3000`: visit all six routes, switch language to Japanese, open the Tasks and Profile modals, and check the console for new errors.
5. Run `cd client && npm run build` to confirm it compiles.
6. Run the `code-reviewer` subagent on the result.
