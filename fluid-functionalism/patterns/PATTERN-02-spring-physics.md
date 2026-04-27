# Pattern: Spring Physics Token System

**Source:** fluid-functionalism (`registry/default/lib/springs.ts`)
**Steal:** Architecture — replace CSS transitions with spring physics

## What It Does

Defines 3 spring animation tokens used uniformly across all components. Springs feel more natural than CSS `ease-in-out` because they account for velocity and interrupt gracefully.

## Architecture

```typescript
export const springs = {
  fast:     { type: "spring", duration: 0.08, bounce: 0 },
  moderate: { type: "spring", duration: 0.16, bounce: 0.15 },
  slow:     { type: "spring", duration: 0.24, bounce: 0.15 },
} as const;
```

Multiplied by Framer Motion's default stiffness/damping spring curve.

## When to Use Each

| Token | Time | Usage |
|-------|------|-------|
| `fast` | 0.08s | Micro-interactions: hover, fade, icon stroke-weight change, opacity |
| `moderate` | 0.16s | Short movement: dropdown, tooltip, tab indicator slide, accordion expand |
| `slow` | 0.24s | Large movement: modal enter/exit, overlay fade, side panel |

## Enter/Exit Convention

- **Enter** uses `springs.slow` — feels deliberate, draws attention to new element
- **Exit** uses `springs.moderate` — faster exit keeps interface responsive
- Exit always faster than enter (signals finality, prevents sluggish feel)

## How to Use (Framer Motion)

```tsx
<motion.div
  initial={{ opacity: 0 }}
  animate={{ opacity: 1 }}
  transition={springs.slow}
/>
```

## How to Use (Vanilla CSS)

For DF's brutalist stack if not using Framer Motion:
```css
:root {
  --spring-fast: 80ms;
  --spring-moderate: 160ms;
  --spring-slow: 240ms;
}

/* Fast micro-interactions */
.element {
  transition: opacity var(--spring-fast) cubic-bezier(0.4, 0, 0.2, 1),
              transform var(--spring-fast) cubic-bezier(0.4, 0, 0.2, 1);
}

/* Moderate movement */
.indicator {
  transition: left var(--spring-moderate) cubic-bezier(0.4, 0, 0.2, 1),
              width var(--spring-moderate) cubic-bezier(0.4, 0, 0.2, 1);
}
```

## How to Steal for DF.com

- **Keep:** The 3-tier timing hierarchy + enter/exit asymmetry
- **Adapt:** Convert Framer spring to CSS cubic-bezier if DF.com uses CSS transitions
- **DF use cases:** Tab indicator slide, sidebar hover highlight, modals, tooltips
