# Pattern: ShapeContext (Global Radius Provider)

**Source:** fluid-functionalism (`registry/default/lib/shape-context.tsx`)
**Steal:** Architecture — global design token provider with smooth transitions

## What It Does

Global provider that switches all components between "pill" and "rounded" variants simultaneously. When toggled (keyboard `R`), ALL components smoothly transition to new border radius.

## Architecture

```
ShapeProvider
  └── shapeMap { pill, rounded }
        └── per-component classes (item, bg, focusRing, container, button, input)
              └── useShape() hook reads current
```

## Key Architecture Decisions

1. **Shape class map:** Not a single radius value but a *group of classes* — each component type gets its own radius via the same shape token. E.g., `container` uses `rounded-xl` while `item` uses `rounded-lg`. Consistent ratio regardless of shape mode.

2. **Transitioning class:** When shape changes, `document.documentElement.classList.add("transitioning")` plus `void root.offsetHeight` (forces reflow). CSS captures all mutable properties:
   ```css
   html.transitioning *,
   html.transitioning *::before,
   html.transitioning *::after {
     transition: border-radius 180ms ease-in-out,
                 background-color 180ms ease-in-out,
                 color 180ms ease-in-out !important;
   }
   ```
   Removed after 200ms. This gives smooth shape transition without per-component logic.

## How to Steal for DF.com

- **Keep:** Class group provider pattern, transitioning class + forced reflow trick
- **Drop:** pill/rounded as the modes — DF doesn't do pill shapes
- **DF adaptation:** Instead of pill/rounded, use:
  - `sharp` (brutalist, radius=0) vs `soft` (radius=4px) vs `rounded` (radius=8px)
  - Or just keep `sharp` as the only option (DF is brutalist)
- **DF use case:** If DF ever wants a "soft mode" toggle, this is the pattern.

## Core Provider

```tsx
const shapeMap = {
  sharp: {
    container: "rounded-none",
    button: "rounded-none",
    input: "rounded-none",
    item: "rounded-none",
    bg: "rounded-none",
    focusRing: "rounded-sm",
  },
  soft: {
    container: "rounded-lg",
    button: "rounded-md",
    input: "rounded-md",
    item: "rounded-md",
    bg: "rounded-md",
    focusRing: "rounded-md",
  },
};
```
