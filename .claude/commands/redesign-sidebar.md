---
description: Redesign the Vue 3 app UI into a modern SaaS-style layout with a fixed vertical sidebar, replacing the top navigation bar
---

# Redesign: Top Nav → Vertical Sidebar (SaaS Style)

Transform this Vue 3 application's layout from a horizontal top navigation bar into a modern SaaS-style interface with a fixed left sidebar, consistent spacing, and a polished professional look.

## Phase 1 — Understand the current layout

Before writing any code, read and internalize:

1. **`client/src/App.vue`** — the root component. Understand:
   - How the top nav is structured (links, logo, user menu, filter bar)
   - What CSS variables and design tokens are defined
   - Any modals, overlays, or global UI elements rendered here
   - The overall layout (header + router-view, or something else)

2. **`client/src/main.js`** — the router definition. Extract every route path and its component name (you'll need these for the sidebar nav items).

3. **`client/src/components/FilterBar.vue`** — understand how the global filters are rendered so you can position them correctly in the new layout.

4. Any other components imported directly in App.vue.

## Phase 2 — Plan the new layout

Design a layout with these three structural zones:

```
┌─────────────────────────────────────────────────┐
│  ┌──────────┐  ┌──────────────────────────────┐ │
│  │          │  │  Topbar (page title + actions)│ │
│  │ Sidebar  │  ├──────────────────────────────┤ │
│  │  240px   │  │                              │ │
│  │  fixed   │  │   <router-view />            │ │
│  │          │  │   (scrollable content area)  │ │
│  │          │  │                              │ │
│  └──────────┘  └──────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

**Sidebar anatomy (top → bottom):**
- Logo / app name at the top (with subtle bottom border)
- Navigation links in the middle (fills remaining space with `flex: 1`)
- FilterBar component near the bottom (above user profile)
- User profile / account section pinned to the bottom

**Navigation link style:**
- Each link: icon (SVG) + label text, 40px tall, 12px horizontal padding
- Active state: accent-colored left border (3px) + light background tint
- Hover state: slightly lighter background, no border
- Icon: 18×18px, same color as label, slightly muted when inactive

**Topbar (the horizontal bar inside the main content area):**
- Sticky, 56px tall
- Shows current page title (derived from the active route name)
- Right side: any page-level actions the original header had (notifications, search, etc.)
- Subtle bottom border, same surface color as sidebar or slightly lighter

**Main content area:**
- Takes all remaining horizontal space
- Has its own vertical scroll (overflow-y: auto)
- Consistent inner padding: 24px (or match what views already use)

**Design tokens to use** (from the existing design system):
- `--bg: #0f172a` — page/sidebar background
- `--surface: #1e293b` — card and topbar background
- `--border: #334155` — dividers
- `--text: #e2e8f0` — primary text
- `--subtle: #94a3b8` — secondary text / inactive icons
- `--accent: #38bdf8` — active indicators, highlights
- Sidebar width: `240px`
- Transition: `0.15s ease` on hover/active states

## Phase 3 — Delegate implementation to vue-expert

**IMPORTANT:** You MUST delegate ALL Vue file creation and modification to the `vue-expert` subagent. Do not edit `.vue` files yourself.

Spawn a `vue-expert` agent with a detailed prompt that includes:

1. The full current content of `App.vue` (paste it verbatim so the agent has exact context)
2. The full route list from `main.js`
3. The full current content of `FilterBar.vue`
4. The complete layout plan from Phase 2 above
5. These specific implementation instructions:

   **App.vue rewrite:**
   - Replace the top `<header>` / nav bar with a fixed-position sidebar `<aside>`
   - The sidebar must use CSS `position: fixed; left: 0; top: 0; height: 100vh; width: 240px`
   - The main content wrapper gets `margin-left: 240px` and `display: flex; flex-direction: column; min-height: 100vh`
   - Include a `<nav>` inside the sidebar with a `<RouterLink>` for every route — include SVG icons appropriate to each route's domain (dashboard, inventory, orders, demand, spending, reports, backlog)
   - Keep ALL existing functionality: modals, profile menu, language switcher, tasks modal, auth logic — just relocate them into the sidebar or topbar
   - The FilterBar goes inside the sidebar, above the user profile section
   - Preserve all existing CSS variables and extend them with `--sidebar-width: 240px`
   - Active link: use `router-link-active` class with a left border accent + background tint
   - Keep scoped styles; do not break any existing component styles

   **Do NOT modify any view files** (`client/src/views/*.vue`) unless a view has hard-coded top-padding that compensates for the old sticky header. If so, adjust only that padding.

   **Preserve everything that works:** filter logic, composables, API calls, modals, i18n, auth — only the layout shell changes.

6. Ask the vue-expert to also verify the result makes sense by reviewing its own output before returning.

## Phase 4 — Verify in the browser

After the vue-expert completes the implementation:

1. Check if the dev server is already running on port 3000. If not, start it:
   ```
   cd client && npm run dev &
   ```
   Wait for it to be ready before proceeding.

2. Use Playwright MCP tools to:
   - Navigate to `http://localhost:3000`
   - Take a screenshot of the full page
   - Click each sidebar nav link and take a screenshot to confirm routing works
   - Confirm the sidebar is visible and fixed on all pages
   - Confirm the FilterBar is present in the sidebar
   - Check that no layout is broken (no overlapping elements, no missing content)

3. If you find visual bugs:
   - Describe them clearly
   - Spawn another vue-expert agent with the specific fix needed
   - Re-verify with Playwright

## Phase 5 — Report

Provide a concise summary:
- What changed (which files were modified)
- A description of the new layout
- Any known limitations or follow-up polish opportunities
- Screenshot of the final result (from Playwright)
