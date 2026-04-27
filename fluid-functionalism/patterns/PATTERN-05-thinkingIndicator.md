# Pattern: ThinkingIndicator

**Source:** fluid-functionalism (`registry/default/thinking-indicator.tsx`)
**Steal:** Architecture — morphing SVG + cycling word animation

## What It Does

AI "thinking" indicator with dual animation:
1. SVG path morphs: circle → infinity symbol → circle (6s cycle)
2. Word cycles: "Thinking" → "Moonwalking" → "Planning" → "Refining" (4s interval, AnimatePresence popLayout)

## Architecture

```
ThinkingIndicator
  ├── morphing SVG (motion.path with animate.d)
  └── cycling text (AnimatePresence mode="popLayout")
        └── invisible placeholder span (prevents layout shift)
```

## Key Architecture Decisions

1. **SVG path morphing:** Three keyframe paths (`circleA → infinity → circleB → infinity → circleA`) animated via Framer Motion's `animate.d` with `repeat: Infinity`. Path data is pure geometry — no visual skin.

2. **Vertical word cycling:** `AnimatePresence mode="popLayout"` — each word enters from above, old word exits upward. Creates smooth vertical conveyor belt feel.

3. **Shimmer text class:** Active words get `shimmer-text` CSS class (subtle gradient animation — visual skin, can be dropped).

4. **Invisible placeholder:** Longest word rendered as invisible span `aria-hidden="true"` — container width stays constant, no layout shift as shorter/longer words cycle.

5. **Accessibility:** `role="status"` for screen readers. SVG has `aria-hidden="true"`.

## How to Steal for DF.com

- **Keep:** Path morph pattern, word cycling with AnimatePresence, invisible placeholder for layout stability
- **Drop:** `shimmer-text` visual effect, specific word list
- **DF adaptation:** 
  - Replace words with DF-specific: "Gândind" / "Analizând" / "Reflectând" (Romanian — DF's primary language)
  - SVG morph: strip strokeWidth/color, use DF's mono line style
  - No shimmer — DF is brutalist, keep it flat

## Core Logic (stripped)

```tsx
const words = ["Thinking", "Moonwalking", "Planning", "Refining"];

function ThinkingIndicator() {
  const [index, setIndex] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setIndex(i => (i + 1) % words.length), 4000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div role="status" className="flex items-center gap-2">
      {/* Morphing SVG */}
      <motion.svg width={20} height={20} viewBox="0 0 24 24" fill="none">
        <motion.path
          animate={{ d: [circleA, infinity, circleB, infinity, circleA] }}
          transition={{ duration: 6, ease: "easeInOut", repeat: Infinity }}
        />
      </motion.svg>

      {/* Cycling text + invisible placeholder */}
      <span className="inline-grid overflow-hidden">
        <span className="col-start-1 row-start-1 invisible" aria-hidden="true">
          {longestWord}
        </span>
        <AnimatePresence mode="popLayout">
          <motion.span
            key={words[index]}
            className="col-start-1 row-start-1"
            initial={{ y: "80%", opacity: 0 }}
            animate={{ y: 0, opacity: 1 }}
            exit={{ y: "-80%", opacity: 0 }}
          >
            {words[index]}
          </motion.span>
        </AnimatePresence>
      </span>
    </div>
  );
}
```
