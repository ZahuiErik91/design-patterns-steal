# Pattern: NavMenu with Proximity Hover

**Source:** fluid-functionalism (`registry/default/nav-menu.tsx`)
**Steal:** Architecture — active-route tracking + proximity hover indicator

## What It Does

Navigation sidebar/menu that:
1. Shows persistent background on currently active route
2. Shows proximity-based hover highlight on nearest item
3. When hovering an item *other than* the active route, the active-route bg dims (opacity 0.8) and hover highlight appears
4. Supports keyboard navigation (ArrowUp/Down, Home/End)

## Architecture

```
NavMenu (container)
  │
  ├── ActiveRoute Background (persistent, follows activeSlug)
  │     └── AnimatePresence — appears/disappears when route changes
  │
  ├── Hover Background (temporary, follows cursor proximity)
  │     └── sessionRef as key — fresh motion.div per hover session
  │
  └── NavItem[] (children)
        └── data-nav-index attribute for keyboard nav
```

## Key Architecture Decisions

1. **Two-layer indicator system:**
   - Layer 1: Active route background (persistent, spring-animated)
   - Layer 2: Hover background (ephemeral, AnimatePresence, faster spring)
   - Both use `position: absolute` inside the relative container

2. **Slug-to-Index mapping:** `slugToIndexRef` Map maps route slugs to item indices. Active route derived from `activeSlug` prop → index → rect.

3. **Opacity dim on hover-other:** When hovering non-active item, active bg opacity drops to 0.8 (user attention goes to hover target, but route context preserved).

4. **Keyboard nav:** Arrow keys wrap around, Home/End jumps to first/last.

5. **Focus management:** `onFocus` checks `:focus-visible` to show keyboard focus ring separately from mouse hover.

## How to Steal for DF.com

- **Keep:** Two-layer indicator, slug mapping, keyboard nav, blur/focus handling
- **Drop:** `shape.bg`, pill rounding, shadow, bg opacity values — DF uses thin dividers, no bg blocks
- **DF adaptation:** Instead of background highlights, use thin border-left on active item + proximity offset on adjacent item borders
- **DF uses:** Sidebar navigation for DF.com system dashboard

## Core Logic (stripped)

```tsx
<nav
  ref={containerRef}
  className="relative flex flex-col gap-0.5"
>
  {/* Active route indicator */}
  <AnimatePresence>
    {activeRouteRect && (
      <motion.div
        className="absolute pointer-events-none"
        animate={{ top, left, width, height }}
      />
    )}
  </AnimatePresence>

  {/* Hover indicator */}
  <AnimatePresence>
    {hoverRect && (
      <motion.div
        className="absolute pointer-events-none"
        animate={{ top, left, width, height }}
      />
    )}
  </AnimatePresence>

  {children}
</nav>
```
