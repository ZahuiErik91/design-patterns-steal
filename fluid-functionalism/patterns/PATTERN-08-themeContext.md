# Pattern: Theme Context (System/Light/Dark)

**Source:** fluid-functionalism (`registry/default/lib/theme-context.tsx`)
**Steal:** Architecture — dark mode with smooth transition and keyboard shortcut

## What It Does

Theme switcher (system/light/dark) with:
- Cycles on keyboard `T`
- Adds `transitioning` class before theme change for smooth CSS transitions
- Saves to state, toggles `.dark`/`.light` on `<html>`

## Architecture

```
ThemeProvider
  └── theme: "system" | "light" | "dark"
  └── setTheme(newTheme)
  └── keyboard: T key cycles themeOrder
        └── skips if focused on input/textarea/contentEditable
```

## Key Architecture Decisions

1. **transitioning class + forced reflow:** Same as ShapeContext. Before theme change, add class to `<html>`, force reflow with `void root.offsetHeight`, remove after 200ms. This animates ALL color/border changes smoothly.

2. **Browser input guard:** Keyboard shortcut ignores if user is typing in an input field — prevents accidental theme toggle while typing.

3. **CSS variable system:** Theme sets `.light`/`.dark` classes, CSS vars respond via class selector. No runtime CSS-in-JS — pure CSS custom properties driven by class toggle.

## DF.com Adaptation

DF.com already uses dark mode via CSS variables. Key takeaways:
- The `transitioning` class + forced reflow technique for smooth theme switching
- Browser input guard pattern for keyboard shortcuts
- The system/light/dark triad pattern

## Minimal DF Adaptation

```tsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState<"system" | "dark" | "light">("system");

  useEffect(() => {
    const root = document.documentElement;
    root.classList.remove("light", "dark");
    if (theme !== "system") root.classList.add(theme);
  }, [theme]);

  // Keyboard shortcut: T cycles system → light → dark
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.key !== "t" || e.metaKey || e.ctrlKey) return;
      const tag = (e.target as HTMLElement)?.tagName;
      if (tag === "INPUT" || tag === "TEXTAREA") return;
      e.preventDefault();
      // Add transitioning class for smooth animation
      const root = document.documentElement;
      root.classList.add("transitioning");
      void root.offsetHeight;
      setTheme(t => t === "system" ? "dark" : t === "dark" ? "light" : "system");
      setTimeout(() => root.classList.remove("transitioning"), 200);
    };
    document.addEventListener("keydown", handler);
    return () => document.removeEventListener("keydown", handler);
  }, []);

  return children;
}
```
