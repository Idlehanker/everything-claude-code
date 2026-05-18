# ADR-ACC-001: AccessiBoard Demo Design Rationale

**Status:** Accepted
**Platform:** Web
**WCAG 2.2 Criteria:** 1.1.1, 1.4.1, 1.4.3, 1.4.11, 2.1.1, 2.1.2, 2.2.2, 2.4.3, 2.4.11, 2.5.8, 3.3.2, 4.1.2

## Context

We needed a demonstration project for the `/ecc:a11y-architect` agent that clearly shows the difference between inaccessible and WCAG 2.2 Level AA compliant UI patterns. The demo targets developers learning accessibility, so it must be self-contained, visually clear, and annotated with specific success criteria references.

**Constraints:**
- Single HTML file (no build step, no dependencies)
- Side-by-side before/after comparison
- Each fix annotated with the relevant WCAG 2.2 SC number
- Covers Perceivable, Operable, Understandable, and Robust (POUR)

## Decision

We built "AccessiBoard" — a task board UI with 5 demo sections, each demonstrating a distinct accessibility category:

| Section | POUR Principle | WCAG SCs |
|---|---|---|
| Task Board | Perceivable + Robust | 1.1.1, 1.4.1, 2.5.8, 4.1.2 |
| Typography & Contrast | Perceivable | 1.4.3, 1.4.11 |
| Search Form | Understandable + Robust | 3.3.2, 4.1.2 |
| Detail View (Modal) | Operable | 2.1.2, 2.4.3, 2.4.11 |
| Media | Operable | 2.2.2, 2.1.1 |

### Implementation Details

**Before column** deliberately introduces these anti-patterns:
- Icon-only buttons without `aria-label` (invisible to screen readers)
- Status conveyed solely through color dots (fails SC 1.4.1)
- 24px touch targets (meets old minimum, but many PDAs require 44px)
- #999 text on white background (2.8:1 contrast — fails SC 1.4.3)
- Placeholder-only form fields with no `<label>` (SC 3.3.2 failure)
- Modal dialog with no close button or Escape handler (keyboard trap — SC 2.1.2)
- Autoplaying video without controls (SC 2.2.2)

**After column** fixes each issue:
- `aria-label` on all icon buttons + `aria-hidden="true"` on decorative SVGs
- Color + icon + text badges for status (redundant coding)
- 44x44px hit areas on interactive elements
- #1a1a2e body text (10:1 contrast) and #555 secondary text (7:1)
- Explicit `<label for="...">` elements on all form controls
- Modal with Escape key handler, focus trap, and visible close button
- Video with `controls` attribute, no `autoplay`

### What We Did Not Do

- No `aria-live` region demo (requires async content — too complex for a single file)
- No custom focus trap library (kept it vanilla for clarity)
- No screen reader-only announcements beyond `aria-label` and `role="status"`

## Consequences

**Positive:**
- Developers can open a single file and immediately see the difference
- Each fix maps to a specific WCAG 2.2 success criterion — no guesswork
- Works with actual screen readers for live testing
- Annotations explain the "why" not just the "what"

**Trade-offs:**
- Single-file constraint limits complexity of interactions
- Side-by-side layout breaks to stacked on mobile (acceptable trade-off)
- Some ARIA patterns (combobox, treeview) are omitted for brevity

## Compliance Mapping

| Component | Before (Broken) | After (Fixed) | WCAG SC |
|---|---|---|---|
| Icon button | `<button><svg>...</svg></button>` | `<button aria-label="..."><svg aria-hidden="true">...</svg></button>` | 4.1.2, 1.1.1 |
| Status indicator | Color dot only | Color + icon + text badge with `role="status"` | 1.4.1 |
| Touch target | 24x24px | 44x44px | 2.5.8 |
| Body text color | #999 on #fff (2.8:1) | #1a1a2e on #fff (10:1) | 1.4.3 |
| Form label | `<input placeholder="...">` | `<label for="...">` + `<input id="...">` | 3.3.2 |
| Modal close | Hidden button with no label | Visible close button + Escape handler + focus return | 2.1.2, 2.4.3 |
| Video | `autoplay loop` no controls | `controls preload="metadata"` | 2.2.2 |
