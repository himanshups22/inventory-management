---
name: saas-redesign
description: Redesigns a Vue 3 application's UI into a modern SaaS-style interface with vertical sidebar navigation, consistent spacing, and polished professional look. Use when asked to redesign, restyle, or modernize the app's layout.
---

# SaaS UI Redesign Skill

Transform the application from a top-nav horizontal layout into a modern SaaS-style interface with a fixed vertical sidebar, consistent design tokens, and a polished professional appearance.

## Execution Strategy

This is a large visual change. Delegate ALL `.vue` file modifications to the **vue-expert** agent. Coordinate the work by:

1. Planning the full set of changes
2. Delegating App.vue (layout shell + global styles) to vue-expert first
3. Delegating view-level adjustments to vue-expert in parallel
4. Verifying with Playwright after changes land

## Target Design System

### Layout Architecture

```
┌──────────────────────────────────────────────────┐
│ ┌────────┐ ┌──────────────────────────────────┐  │
│ │        │ │  Top Bar (breadcrumb + profile)  │  │
│ │  Side  │ ├──────────────────────────────────┤  │
│ │  bar   │ │                                  │  │
│ │        │ │         Main Content             │  │
│ │  Nav   │ │         (router-view)            │  │
│ │  Links │ │                                  │  │
│ │        │ │                                  │  │
│ │        │ │                                  │  │
│ └────────┘ └──────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

**Structure:**
- `.app` becomes a horizontal flex container (not vertical)
- `.sidebar` is fixed-width (260px), full viewport height, dark background
- `.main-area` is flex: 1, contains a slim top bar + scrollable content area
- FilterBar moves into the top bar area or just below it

### Design Tokens (CSS Custom Properties)

Replace all hardcoded color values with CSS custom properties on `:root`. This is critical for consistency and future theming.

```css
:root {
  /* Sidebar */
  --sidebar-width: 260px;
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-hover: #f1f5f9;
  --sidebar-active-bg: rgba(59, 130, 246, 0.1);
  --sidebar-active-text: #3b82f6;
  --sidebar-border: #1e293b;

  /* Surfaces */
  --bg: #f1f5f9;
  --surface: #ffffff;
  --surface-border: #e2e8f0;
  --surface-hover-border: #cbd5e1;
  --elevated-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.06);

  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #64748b;
  --text-faint: #94a3b8;

  /* Accent / Brand */
  --accent: #3b82f6;
  --accent-light: #eff6ff;
  --accent-text: #1e40af;

  /* Status */
  --success: #059669;
  --success-bg: #d1fae5;
  --success-text: #065f46;
  --warning: #ea580c;
  --warning-bg: #fed7aa;
  --warning-text: #92400e;
  --danger: #dc2626;
  --danger-bg: #fecaca;
  --danger-text: #991b1b;
  --info: #2563eb;
  --info-bg: #dbeafe;
  --info-text: #1e40af;

  /* Spacing scale (4px base) */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-5: 1.25rem;   /* 20px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;

  /* Typography */
  --font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 0.938rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.375rem;
  --font-size-2xl: 1.875rem;
}
```

### Sidebar Specifications

**Visual:**
- Background: `var(--sidebar-bg)` (#0f172a — dark slate)
- Full height: `height: 100vh; position: fixed; left: 0; top: 0`
- Width: `var(--sidebar-width)` (260px)
- Subtle right border or no border (dark bg provides enough separation)
- Overflow-y: auto for scroll on many items

**Logo Section (top of sidebar):**
- Company name in white, subtitle in muted text
- Padding: `var(--space-6) var(--space-5)`
- Bottom border: 1px solid `var(--sidebar-border)`

**Navigation Links:**
- Vertical list, one link per row
- Each link: `padding: var(--space-3) var(--space-5)`, full width
- Icon on the left (use inline SVG icons — no external icon library)
- Text: `var(--sidebar-text)`, font-weight 500
- Hover: background `rgba(255, 255, 255, 0.04)`, text `var(--sidebar-text-hover)`
- Active: background `var(--sidebar-active-bg)`, text `var(--sidebar-active-text)`, left border-left 3px solid `var(--sidebar-active-text)`, border-radius on right side only

**SVG Icons (inline, 20x20):**
Provide simple SVG path icons for each nav item. Use these specific icons:
- Overview/Dashboard: grid/squares icon
- Inventory: box/package icon
- Orders: clipboard/list icon
- Finance: dollar/chart icon
- Demand Forecast: trending-up icon
- Restocking: refresh/rotate icon
- Reports: bar-chart icon

Each icon should be a simple `<svg>` with `width="20" height="20" viewBox="0 0 24 24"`, stroke-based (stroke="currentColor", fill="none", stroke-width="2", stroke-linecap="round", stroke-linejoin="round"). This matches the Lucide/Feather icon style.

**Bottom Section:**
- Language switcher and profile menu pinned to bottom of sidebar
- Separated from nav links by `margin-top: auto` (use flex column on sidebar)

### Top Bar Specifications

**Structure:**
- Height: 56px
- Background: `var(--surface)` (white)
- Border-bottom: 1px solid `var(--surface-border)`
- Display: flex, align-items: center
- Padding: `0 var(--space-8)`

**Contents (left to right):**
- Page title (pulled from current route or view's h2)
- Spacer (flex: 1)
- FilterBar component (compact, inline)

### Main Content Area

- Background: `var(--bg)` (#f1f5f9)
- Padding: `var(--space-6) var(--space-8)`
- `margin-left: var(--sidebar-width)` to offset for fixed sidebar
- `min-height: 100vh`
- Max-width: none (fills available space — the sidebar already constrains)
- Overflow-y: auto

### Updated Global Component Styles

**Cards:**
```css
.card {
  background: var(--surface);
  border-radius: var(--radius-md);
  padding: var(--space-5);
  border: 1px solid var(--surface-border);
  margin-bottom: var(--space-5);
  box-shadow: var(--elevated-shadow);
}
```

**Stat Cards:**
```css
.stat-card {
  background: var(--surface);
  padding: var(--space-5);
  border-radius: var(--radius-md);
  border: 1px solid var(--surface-border);
  box-shadow: var(--elevated-shadow);
  transition: all 0.2s ease;
}
```

**Tables:** keep existing styles but replace hardcoded colors with variables.

**Badges:** keep existing styles but use variables.

### FilterBar Adjustments

The FilterBar should be made more compact for the top bar:
- Horizontal layout with smaller dropdowns
- Less padding
- Fits within the 56px top bar height
- If too many filters, allow horizontal scroll or wrap to second line

## Step-by-Step Instructions

### Step 1: Update App.vue — Layout Shell

Transform the template from vertical (header → content) to horizontal (sidebar | content):

**Template changes:**
1. Replace `<header class="top-nav">` with a `<aside class="sidebar">` element
2. Move `<nav>` links into the sidebar as vertical list with SVG icons
3. Move logo into sidebar top
4. Move LanguageSwitcher and ProfileMenu into sidebar bottom
5. Add a `<div class="main-area">` wrapping a `<div class="top-bar">` + `<main>`
6. Move FilterBar into or just below the top bar

**Style changes:**
1. Add all CSS custom properties to `:root`
2. Replace `.app` flex-direction from column to row
3. Remove `.top-nav`, `.nav-container`, `.nav-tabs` styles
4. Add `.sidebar`, `.sidebar-logo`, `.sidebar-nav`, `.sidebar-nav a`, `.sidebar-bottom` styles
5. Add `.main-area`, `.top-bar` styles
6. Update `.main-content` to remove max-width constraint and adjust padding
7. Update all global component styles (`.card`, `.stat-card`, `.badge`, etc.) to use CSS variables

### Step 2: Update FilterBar Component

Make the FilterBar layout more compact:
- Single horizontal row
- Smaller select elements
- Fits within the top bar or just below it

### Step 3: Update View Components (if needed)

Views should mostly work without changes since they use the global `.card`, `.stat-card`, `.stats-grid` classes. However, check each view for:
- Hardcoded widths that conflict with the new layout
- Any `max-width` constraints that should be removed
- Page headers that duplicate the top bar title

### Step 4: Verify

After all changes:
1. Check every route renders correctly
2. Sidebar active state highlights current route
3. Filters still work across all views
4. Modals render correctly (z-index above sidebar)
5. Language switching still works
6. Mobile considerations: sidebar could collapse (optional, not required)

## File Change Summary

| File | Change Type | Description |
|------|------------|-------------|
| `client/src/App.vue` | Major rewrite | Layout shell: sidebar + top bar + content area. All global styles updated to use CSS variables. |
| `client/src/components/FilterBar.vue` | Moderate edit | Compact horizontal layout for top bar integration |
| `client/src/views/*.vue` | Minor/none | Remove page-header h2 if top bar shows page title. Fix any hardcoded widths. |

## Anti-Patterns to Avoid

- Do NOT add any external CSS framework (Tailwind, Bootstrap, etc.)
- Do NOT add any icon library (Font Awesome, etc.) — use inline SVGs
- Do NOT change any JavaScript logic, API calls, or data flow
- Do NOT change router configuration
- Do NOT modify backend code
- Do NOT break existing functionality (filters, modals, i18n)
- Do NOT use emojis in the UI
- Do NOT add animations beyond subtle transitions (0.2s ease)
- Do NOT change the color palette dramatically — refine, don't reinvent

## Quality Checklist

Before considering the redesign complete:

- [ ] Sidebar renders with all nav links and correct SVG icons
- [ ] Active route is visually highlighted in sidebar
- [ ] Logo and subtitle appear at top of sidebar
- [ ] Profile menu and language switcher are at bottom of sidebar
- [ ] Top bar shows page context and filter bar
- [ ] Main content area has correct background and padding
- [ ] All CSS custom properties are defined on `:root`
- [ ] Cards, tables, badges, stat cards use CSS variables
- [ ] All 7 views render correctly (Dashboard, Inventory, Orders, Demand, Spending, Restocking, Reports)
- [ ] Filters work on all applicable views
- [ ] Modals open above the sidebar (z-index)
- [ ] i18n works (both EN and JA)
- [ ] No horizontal scroll on standard viewport widths (1280px+)
- [ ] No console errors
