# Steal: Interaction Patterns from Fluid Functionalism

**Forked from:** [mickadesign/fluid-functionalism](https://github.com/mickadesign/fluid-functionalism) (193★, MIT)
**Forked to:** [ZahuiErik91/fluid-functionalism](https://github.com/ZahuiErik91/fluid-functionalism)

## What We Stole

Pure **interaction architecture** — not visual skin. All patterns extracted without FF's pill shapes, shadows, Inter font, or smooth gradients. Ready to be skinned in DF.com's brutalist+Swiss aesthetic (IBM Plex + SF Mono, thin dividers, no shadows).

## Patterns

| # | Pattern | What It Does | DF Use Case |
|---|---------|-------------|-------------|
| 01 | **useProximityHover** | Cursor proximity detection on list items | Sidebar, tabs, any interactive list |
| 02 | **Spring Physics** | 3-tier spring timing (80/160/240ms) | Replace CSS transitions in all DF animations |
| 03 | **NavMenu** | Two-layer indicator (active + hover preview) | Sidebar navigation |
| 04 | **Tabs** | Sliding indicator with proximity preview | Dashboard section navigation |
| 05 | **ThinkingIndicator** | Morphing SVG + cycling word animation | AI chat thinking state |
| 06 | **ThinkingSteps** | Sequential chain-of-thought reveal | AI reasoning display in chat |
| 07 | **ShapeContext** | Global radius provider + smooth transition | If DF ever adds "soft mode" |
| 08 | **ThemeContext** | Dark/light/system + keyboard shortcut | DF dark mode toggle |

## Top Priority for DF.com

1. **ThinkingIndicator + ThinkingSteps** — Direct fit for AI therapy chat. Shows AI's reasoning process.
2. **useProximityHover** — Signature UX diff. Apply to sidebar nav items (border-left highlight moves toward cursor before click).
3. **Tab sliding indicator** — Replace current tab system with proximity-aware spring indicator.
4. **Spring token system** — Standardize all DF animations to 3-tier spring timing.

## Adaptation Pattern

Each pattern is extracted as:
```
PATTERN-NN-name.md
├── What It Does
├── Architecture (diagram + decision log)
├── How to Steal — Keep vs Drop
├── DF Adaptation (code sketch in DF's aesthetic)
└── Standalone Source (link to original)
```

## The Signature Approach

FF's core insight worth stealing: **two indicators, one cursor.** 
- One indicator shows WHERE YOU ARE (active state)
- One indicator shows WHERE YOU'RE GOING (proximity preview)
- Both animated with springs, not CSS transitions
- They coexist on screen, dimming/opacity to layer information

This works regardless of visual style — applies equally to pill shapes AND thin mono dividers.
