# Plan: Add Dark Mode Toggle

| Field | Value |
|-------|-------|
| Status | complete |
| Created | 2026-04-01 |
| Ticket | N/A |
| Branch | feature/dark-mode |

## Context

Users work in low-light environments and have requested a dark mode option. The app currently has no theme support — all colors are hardcoded in CSS. We need a toggle in the settings panel and persistent preference via localStorage.

## Architecture Decisions

- **CSS variables** — Swap a single `data-theme` attribute on `<html>`. No CSS-in-JS, no class juggling. One place to change, everything follows.
- **localStorage** — Persist the preference client-side. No backend change needed for MVP.
- **System preference as default** — Use `prefers-color-scheme` media query as the initial value if no stored preference exists.

## Milestones Overview

1. **Theme tokens** — Define CSS variables for light and dark palettes
2. **Toggle UI** — Settings panel toggle that writes to localStorage and updates `data-theme`

---

## Milestone 1: Theme Tokens

Replace hardcoded color values with CSS variables scoped to `[data-theme]`.

### 1.1 [x] Define CSS variable palettes *(completed 2026-04-01)*

- **Files:** `src/styles/theme.css`
- **What:**
  1. Add `:root` block with light-mode variables: `--bg-primary`, `--bg-secondary`, `--text-primary`, `--text-muted`, `--border`, `--accent`.
  2. Add `[data-theme="dark"]` block overriding the same variables with dark values.
  3. Replace all hardcoded `#fff`, `#1a1a1a`, etc. in `global.css` with the new vars.
- **Acceptance:** Manually setting `document.documentElement.dataset.theme = 'dark'` in the browser console makes the UI visually switch to dark mode.
- **Dependencies:** None

---

## Milestone 2: Toggle UI

### 2.1 [x] Add theme toggle to settings panel *(completed 2026-04-02)*

- **Files:** `src/components/Settings.tsx`, `src/hooks/useTheme.ts` (new)
- **What:**
  1. Create `useTheme.ts` hook:
     - Reads `localStorage.getItem('theme')` on mount; falls back to `prefers-color-scheme`.
     - Sets `document.documentElement.dataset.theme` on every change.
     - Returns `[theme, setTheme]`.
  2. Add a toggle switch to `Settings.tsx` that calls `setTheme`.
- **Acceptance:** Toggle switches theme. Refreshing the page preserves the selection.
- **Dependencies:** 1.1

---

## Verification

```bash
# Manual test steps
# 1. Open app, go to Settings
# 2. Toggle dark mode → UI goes dark
# 3. Refresh page → dark mode persists
# 4. Open DevTools, delete localStorage → falls back to system preference
```
