# a11y-architect-demo

A side-by-side demo of accessible vs. inaccessible web patterns, built to demonstrate the **a11y-architect** agent — an Accessibility Architect that enforces **WCAG 2.2 Level AA** compliance.

## What It Shows

The demo page ("AccessiBoard") presents a task board UI in two columns:

| Before (Broken) | After (Fixed) |
|---|---|
| Icon buttons with no labels | `aria-label` on all icon-only controls |
| Low-contrast text (~3:1) | 4.5:1 minimum contrast ratio |
| Keyboard traps in modals | Proper focus management and escape handling |
| Missing form labels | Explicit `<label>` elements |
| Color-only status indicators | Icons + text + color for status |
| Tiny touch targets (<24px) | 44x44px minimum target size (SC 2.5.8) |
| Auto-playing video without controls | Pause/play with visible controls |

Each fix is annotated with the specific **WCAG 2.2 Success Criterion** it addresses.

## How to Use

1. Open `index.html` in a browser
2. Compare the Before and After columns
3. Hover over annotation badges to see the WCAG criteria details
4. Use a screen reader (VoiceOver, NVDA) to experience the difference

## Files

| File | Purpose |
|---|---|
| `index.html` | Main demo page (self-contained, single file) |
| `CLAUDE.md` | Project-level guidance for Claude |
| `docs/ADR-ACC-001-demo-rationale.md` | Accessibility Decision Record |

## Agent

```
/ecc:a11y-architect — Accessibility Architect for WCAG 2.2 compliance
```
