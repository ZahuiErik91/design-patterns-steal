# Pattern: Tabs with Sliding Indicator + Proximity

**Source:** fluid-functionalism (`registry/default/tabs.tsx`)
**Steal:** Architecture — indicator that slides between tabs with proximity preview

## What It Does

Tab bar where:
1. **Active tab** has a sliding background that springs between positions
2. **Hovering a non-active tab** shows a lighter preview highlight there
3. **Leaving** snaps preview back to active tab
4. Works with `selectedIndex` or `value` control

## Architecture

```
Tabs (root)
  └── TabsList (container)
        ├── Selected Indicator (position: absolute, follows active index)
        ├── Hover Indicator (AnimatePresence, appears on proximity hover)
        ├── Focus Ring (AnimatePresence, keyboard navigation)
        └── TabItem[] (children)
              └── data-proximity-index attribute
```

## Key Architecture Decisions

1. **Three layer system:**
   - `selectedRect`: Animated indicator at active tab — always present
   - `hoverRect`: Ephemeral indicator at hovered tab — AnimatePresence enter/exit
   - `focusRect`: Blue border for keyboard focus — separate from both

2. **Scale+opacity dim on hover:** When `hoveredIndex !== activeSelectedIdx`, the selected indicator opacity drops to 0.85 and hover indicator shows at 0.4 opacity. User sees where they are AND where they're going simultaneously.

3. **Optimistic index:** `setOptimisticIdx` updates the indicator position *immediately* on click (before Radix event propagates). Feels instant.

4. **Icon stroke animation:** On hover, icon `strokeWidth` transitions from 1.5 → 2. On active tab, font-weight transitions from normal → semibold. Micro-interactions within the tab.

5. **Font-weight via font-variation-settings:** Uses variable font weight axis to animate between normal/medium/semibold — no layout shift from font swapping.

6. **Invisible placeholder text:** `aria-hidden="true"` span with semibold weight prevents layout shift when text weight changes.

## How to Steal for DF.com

- **Keep:** Three-layer indicator system, optimistic click, proximity preview, invisible placeholder
- **Drop:** Muted bg on container, pill rounding, shadow, opacity values
- **DF adaptation:** Instead of bg highlights, use thin bottom border that springs between tabs. Hover preview = lighter border color at hovered position.
- **DF uses:** Section navigation in DF.com dashboard (Home → Journal → Profile → Settings)

## Simplified DF Adaptation

```tsx
<div className="relative flex gap-0 border-b border-neutral-300">
  {/* Active indicator — thin bottom border */}
  {selectedRect && (
    <motion.div
      className="absolute bottom-0 h-px bg-foreground"
      animate={{ left: selectedRect.left, width: selectedRect.width }}
      transition={{ type: "spring", duration: 0.16, bounce: 0.15 }}
    />
  )}
  {/* Hover preview — even thinner */}
  {hoverRect && (
    <motion.div
      className="absolute bottom-0 h-[0.5px] bg-muted-foreground/40"
      animate={{ left: hoverRect.left, width: hoverRect.width }}
    />
  )}
  {tabs.map(tab => (
    <button key={tab.value} className="px-3 py-2 text-sm font-mono">
      {tab.label}
    </button>
  ))}
</div>
```
