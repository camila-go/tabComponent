# Accessibility Audit: Format Comparison Tabs
**Standard:** WCAG 2.1 AA (2.2 noted where relevant) | **Date:** 2026-06-12
**Component:** [`index.html`](https://github.com/camila-go/tabComponent/blob/main/index.html) — audited live in Chromium (computed styles, programmatic focus, keyboard dispatch, contrast math)

## Summary
**Issues found this pass:** 4 | **Critical:** 0 | **Major:** 2 (both fixed) | **Minor:** 2 (1 fixed, 1 open for design)

The component follows the ARIA APG Tabs pattern with automatic activation. The two
major findings (focus reaching invisible content; decorative icons exposed to AT)
were fixed during the audit and verified.

## Findings

### Perceivable
| # | Issue | WCAG | Severity | Status |
|---|-------|------|----------|--------|
| 1 | Decorative SVG icons (6 tab icons + shield) had no `aria-hidden`, so screen readers could announce them as unlabeled images | 1.1.1 | 🟡 Major | ✅ Fixed: `aria-hidden="true" focusable="false"` on all decorative SVGs |
| 2 | Inactive pill border (`#ffffff33` on `#212322`) measures **1.92:1**, below the 3:1 non-text contrast minimum for component boundaries | 1.4.11 | 🟢 Minor | ⚠️ Open — values come from design tokens (`gl-neutral-0-opacity-2`); pills remain identifiable via 15.8:1 labels, and selected/focus states far exceed 3:1. **Recommend design review:** raising border to ~40% white (`#ffffff66`) meets 3:1 |

### Operable
| # | Issue | WCAG | Severity | Status |
|---|-------|------|----------|--------|
| 3 | CTA buttons inside visually-hidden panels were reachable by keyboard/programmatic focus (`opacity: 0` keeps elements in the tab order) | 2.4.3 | 🟡 Major | ✅ Fixed: inactive panels get `inert` (instant, timing-independent) plus `aria-hidden` and delayed `visibility: hidden` |

### Robust
| # | Issue | WCAG | Severity | Status |
|---|-------|------|----------|--------|
| 4 | `role="tablist"` element had no accessible name (label sat on a wrapping `<nav>`) | 4.1.2 | 🟢 Minor | ✅ Fixed: `aria-label="Learning format comparison topics"` moved onto the tablist; non-navigation `<nav>` wrapper removed |

## Color Contrast Check (computed, not eyeballed)
| Element | Foreground | Background | Ratio | Required | Pass? |
|---------|-----------|------------|-------|----------|-------|
| Body / checklist text | #ffffff | #212322 | 15.8:1 | 4.5:1 | ✅ |
| Subtitle (72% white) | ≈#c7c8c7 | #212322 | 9.4:1 | 4.5:1 | ✅ |
| Active tab label | #111111 | #ffffff | 18.9:1 | 4.5:1 | ✅ |
| Card text over photo (worst case: photo highlight under 70% black overlay) | #ffffff | ≈#4d4d4d | 8.5:1 | 4.5:1 | ✅ |
| Selected-state indicator (white glider) | #ffffff | #212322 | 15.8:1 | 3:1 | ✅ |
| Focus outline (2px white, 3px offset) | #ffffff | #212322 | 15.8:1 | 3:1 | ✅ |
| Inactive pill border | #ffffff33 blend | #212322 | 1.92:1 | 3:1 | ❌ see finding #2 |

## Keyboard Navigation (all verified by dispatched events)
| Element | Tab Order | Enter/Space | Arrow Keys | Home/End |
|---------|-----------|-------------|-----------|----------|
| Tab pills | One stop (roving tabindex; exactly one `tabindex="0"` at all times) | Activates (native button) | Left/Right move + select, wrapping at both ends | First / last tab |
| Active panel CTAs | After the tablist, in reading order | Activates | — | — |
| Hidden panel CTAs | **Not reachable** (`inert`), verified programmatically | — | — | — |

Selection follows focus (automatic activation) per ARIA APG; no unexpected context
changes on focus (3.2.1 ✅).

## Screen Reader Expectations
| Element | Announced As |
|---------|-------------|
| Tab pill | "Course schedule, tab, selected, 1 of 6" (role, state, position from native semantics) |
| Panel | "tab panel, Course schedule" (`aria-labelledby` → tab) |
| Checklist | "list, 4 items" → each item text; check glyphs are CSS, not announced |
| Icons | Nothing (decorative, hidden) |
| Hidden panels | Nothing (`inert` + `aria-hidden`) |

## Other Checks
- **Touch targets:** pills 48px desktop / 40px mobile, CTAs 48px — meets WCAG 2.2 AA (2.5.8, 24px min); mobile pills are 4px short of the 44px AAA/2.5.5 target (minor; padding could absorb it).
- **Zoom/reflow (1.4.10):** verified down to 320px viewport (≈400% reflow basis) — no horizontal scroll, no clipped content; container height animation always releases to `auto`.
- **Reduced motion (2.3.3):** all transitions disabled via media query; JS skips height pinning.
- **Structure (1.3.1):** `lang="en"`, single `h1`, card brands as `h2` — hidden duplicates are out of the a11y tree via `inert`.
- **No time limits, no auto-playing media, no focus traps.**

## Priority Fixes
1. ~~**Focus reaching invisible CTAs**~~ — fixed with `inert`; blocked keyboard/SR users from a coherent focus order.
2. ~~**Decorative icons exposed to AT**~~ — fixed with `aria-hidden`.
3. **Pill border contrast (open):** needs a design-token decision (`#ffffff33` → `#ffffff66` meets 3:1) — flagged for the design team rather than silently diverging from the published tokens.
4. **Production must-do:** server-render the content (currently JS-injected; empty page without JS affects robust AT setups and SEO alike).
