# Pattern: ThinkingSteps (Chain-of-Thought Display)

**Source:** fluid-functionalism (`registry/default/thinking-steps.tsx`)
**Steal:** Architecture — sequential AI reasoning display with animated step reveal

## What It Does

Collapsible "thinking" display showing AI's chain-of-thought as sequential steps:
1. Steps animate in one-by-one (height:0→auto with spring)
2. Each step has: icon, label, optional description, status (complete/active/pending)
3. Active step gets shimmer text + ellipsis
4. Steps can have nested accordion for details, sources (badges), images
5. Entire thing wraps in an accordion (default open)

## Architecture

```
ThinkingSteps (Accordion wrapper)
  └── ThinkingStepsHeader (click to expand/collapse)
  └── ThinkingStepsContent (animated height)
        └── ThinkingStep[] (sequential)
              ├── Icon column + connector line
              ├── Label + description
              ├── ThinkingStepDetails (nested accordion)
              ├── ThinkingStepSources (badge row)
              ├── ThinkingStepSource (animated badge entry)
              └── ThinkingStepImage (animated image reveal)
```

## Key Architecture Decisions

1. **Two-layer animation:** Outer `motion.div` animates `height: 0 → auto` (creates space), inner `motion.div` fades content in (opacity 0 → 1 after 0.08s delay). Prevents content flash before space opens.

2. **Status-driven rendering:** `pending` steps return `null` (don't render at all), `active` steps get shimmer effect + trailing ellipsis, `complete` steps show static. Natural progressive rendering.

3. **Connector line:** Continuous vertical line from icon column spans between steps — creates visual chain. Only rendered for non-last items.

4. **Nested accordion:** ThinkingStepDetails uses same accordion pattern for optional detail expansion — clean without visual overload.

5. **Source badges:** Badges animate in with `scale + blur → scale:1 + blur:0` spring. Creates staggered reveal effect with configurable delay per source.

6. **Image reveal:** Similar blur→clear spring animation for attached images.

## How to Steal for DF.com

- **Keep:** Sequential reveal with two-layer animation, status-driven rendering, connector line pattern, nested detail accordion
- **Drop:** shimmer text, pill shape, badge visual style, source animation specifics
- **DF adaptation:**
  - Steps use monospace text, thin divider lines instead of pill backgrounds
  - Connector becomes thin mono line (`1px solid var(--border)`)
  - Status icons: `□ pending → ◇ active → ✓ complete` using ASCII/Unicode
  - No background fills — just text + line

## Core Logic (stripped)

```tsx
function ThinkingStep({ status, label, description, isLast, children }: Props) {
  if (status === "pending") return null;

  return (
    <motion.div
      initial={{ height: 0 }}
      animate={{ height: "auto" }}
      transition={{ type: "spring", duration: 0.24 }}
    >
      <motion.div
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        transition={{ duration: 0.24, delay: 0.08 }}
      >
        <div className="flex gap-2.5">
          {/* Icon + connector line */}
          <div className="flex flex-col items-center w-[14px]">
            <StatusIcon status={status} />
            {!isLast && <div className="w-px flex-1 bg-border" />}
          </div>

          {/* Content */}
          <div className="flex-1">
            <span className="font-mono text-sm">{label}</span>
            {description && <span className="text-xs text-muted-foreground">{description}</span>}
            {children}
          </div>
        </div>
      </motion.div>
    </motion.div>
  );
}
```
