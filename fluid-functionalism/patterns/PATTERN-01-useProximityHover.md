# Pattern: useProximityHover

**Source:** fluid-functionalism (`registry/default/hooks/use-proximity-hover.ts`)
**Steal:** Architecture only — cursor proximity tracking, not visual highlight

## What It Does

Tracks which item cursor is NEAREST to in a list. Unlike CSS `:hover` which only activates when cursor is *over* an element, proximity triggers on the *nearest* element even before cursor arrives.

## Architecture

```
containerRef ──→ onMouseMove ──→ measure distance to each item center
                                     │
                                     ▼
                              activeIndex (closest item)
                                     │
                                     ▼
                         registerItem(index, element)
```

## Key Design Decisions

1. **Uses `offsetTop`/`offsetLeft`** instead of `getBoundingClientRect` — immune to CSS transforms on parent elements (scale, translate). Rects are layout-space, not viewport-space.

2. **Scale correction** — Computes `layoutSize / visualSize` ratio to map layout coords into viewport space where mouse cursor lives. Handles scaled parents.

3. **requestAnimationFrame throttling** — MouseMove handler queues measurement in RAF, skips redundant recalculations.

4. **Axis-agnostic** — `axis: "x" | "y"` config. For horizontal lists (Tabs) use `"x"`, vertical lists (Nav) use `"y"` (default).

5. **Session tracking** — `sessionRef` increments on mouseEnter, used as React `key` to create new motion.div instances per hover session (avoids animation glitches on fast re-entries).

## API

```typescript
const {
  activeIndex,     // number | null — closest item index
  setActiveIndex,  // manual override
  itemRects,       // ItemRect[] — measured positions
  sessionRef,      // for React key in motion.div
  handlers,        // { onMouseMove, onMouseEnter, onMouseLeave }
  registerItem,    // (index, element) => void
  measureItems,    // force remeasure
} = useProximityHover(containerRef, { axis: "y" });
```

## How to Steal for DF.com

- **Keep:** The entire hook logic — it's framework-agnostic interaction detection
- **Drop:** The highlight/indicator render (that's visual skin)
- **DF use cases:**
  - Sidebar nav items — highlight nearest without pill background
  - Tab bar — sliding indicator follows proximity
  - Accordion items — hover glow on nearest
  - Any list where cursor position preview matters

## Standalone Source

Full hook source at: `https://raw.githubusercontent.com/mickadesign/fluid-functionalism/main/registry/default/hooks/use-proximity-hover.ts`
Or in our fork: `ZahuiErik91/fluid-functionalism`
