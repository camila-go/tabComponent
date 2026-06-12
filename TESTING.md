# Testing Strategy & Edge-Case Validation: Format Comparison Tabs

**Component:** [`index.html`](https://github.com/camila-go/tabComponent/blob/main/index.html)
**Status:** All executed checks pass as of 2026-06-12. Issues found during this pass were fixed in the same commit (see Findings).

This is a dependency-free static prototype, so the strategy favors a small
automatable core plus a browser-executed edge-case matrix over a heavy harness.

## Testing Pyramid for This Component

| Layer | What | How |
|-------|------|-----|
| Unit (most) | `setActive` state machine, glider math, edge-fade thresholds, `TABS` data shape | Extract the IIFE into a module when productionized; pure-function tests for index math/wrapping |
| Integration | Tab click → ARIA state → panel swap → height settle | Playwright/Cypress driving a real browser (transitions and `inert` need one) |
| E2E / visual (few) | Breakpoint screenshots, reduced-motion variant, font-swap layout | Playwright screenshot diffs at 320/375/768/1024/1280/1680 |
| Accessibility | Axe-core scan + manual SR pass | See [ACCESSIBILITY.md](ACCESSIBILITY.md) |

## Executed Edge-Case Matrix

All run against the live component in Chromium. ✅ = verified passing.

### Interaction edge cases
| # | Case | Result |
|---|------|--------|
| 1 | Rapid-fire clicking 5 tabs inside one frame (interrupted/retargeted transitions) | ✅ exactly one selected tab, one active panel, ARIA consistent, height settles to `auto` |
| 2 | Switching between two equal-height panels (no height transition → no `transitionend`) | ✅ height releases via equal-height guard |
| 3 | Re-clicking the already-active tab | ✅ no-op (guarded by index check) |
| 4 | Click while page backgrounded/occluded (rAF + transitions + timers suspended) | ✅ synchronous safety-net timer releases height; `inert` applies immediately |
| 5 | Tab-row scrolling: active pill auto-centers; edge fades only where content hides | ✅ verified at scroll start/middle/end |

### Keyboard edge cases
| # | Case | Result |
|---|------|--------|
| 6 | `ArrowRight` on last tab wraps to first; `ArrowLeft` on first wraps to last | ✅ |
| 7 | `Home` / `End` jump to first / last | ✅ |
| 8 | Roving tabindex: exactly one tab in the tab order at all times | ✅ |
| 9 | Hidden panels' CTA buttons unreachable by Tab key or programmatic `.focus()` | ✅ via `inert` (timing-independent) |
| 10 | Focus order after switch: active tab → its panel's CTAs only | ✅ (e.g. Tuition → 2× "Learn more") |

### Environment edge cases
| # | Case | Result |
|---|------|--------|
| 11 | Viewports 320–1680px | ✅ no horizontal overflow at any width |
| 12 | `prefers-reduced-motion` | ✅ all transitions disabled; JS skips height pin entirely (code path; emulator can't toggle the media query) |
| 13 | Slow web-font load changing pill widths | ✅ glider re-anchors on `document.fonts.ready` |
| 14 | Window resize | ✅ glider + edge fades recompute on `resize` |
| 15 | Image assets fail to load | ✅ cards keep `background-color: #0c0a08` + overlay; contrast unaffected |

## Findings From This Pass (all fixed)

1. **Stuck container height on equal-height tab switches** — when old/new panels
   measured the same, no `height` transition ran, `transitionend` never fired, and
   the container kept a pinned pixel height (would clip content on later resize).
   *Fix:* equal-height guard releases immediately.
2. **Stuck height when rAF is suspended** (backgrounded/occluded page) — the
   release logic lived inside `requestAnimationFrame`, which doesn't run for
   hidden pages. *Fix:* the safety-net timer is scheduled synchronously in
   `setActive`; `transitionend` still clears it in the normal path.
3. **Hidden-panel focus gate depended on CSS transition timing** — the delayed
   `visibility: hidden` flip is suspended in occluded pages, leaving invisible
   buttons focusable. *Fix:* panels now also get `inert`, which removes them from
   focus order and the accessibility tree instantly with no timing dependence.

## Known Limitations (accepted for a prototype)

- **No-JS:** all content is injected by JS; with scripts disabled the section is
  empty. Production should server-render the markup and hydrate behavior.
- **`inert` support:** Chrome/Edge 102+, Safari 15.5+, Firefox 112+. Older
  browsers degrade to the previous behavior (`aria-hidden` + `pointer-events:
  none` + delayed `visibility`), losing only the focus-exclusion hard guarantee.
- **CTA buttons are placeholders** — no navigation to test yet.

## Recommended Next Steps for Production

- Port the matrix above into Playwright (each numbered case is one spec).
- Add an axe-core CI step (zero violations currently — see ACCESSIBILITY.md).
- Visual regression snapshots at the six verified widths × light/dark × reduced motion.
- Coverage target: the `setActive`/glider/fade logic is ~80 lines — aim for 100%
  branch coverage at unit level; interaction specs cover the rest.
