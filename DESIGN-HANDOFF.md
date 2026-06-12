# Handoff Spec: Format Comparison Tabs ("Two Online Formats Fit For Your Life")

**Prototype repo:** https://github.com/camila-go/tabComponent
**Live code:** [`index.html`](https://github.com/camila-go/tabComponent/blob/main/index.html) — single self-contained file (vanilla HTML/CSS/JS, no build step, no dependencies)
**Figma sources:**
- Desktop tab states: [`node 1-7263` (DT Tabs Area)](https://www.figma.com/design/EUSZZtDMNxsLrzJc2ufZCe/prototype?node-id=1-7263)
- Mobile tab states: [`node 1-6962` (mobile Tabs Area)](https://www.figma.com/design/EUSZZtDMNxsLrzJc2ufZCe/prototype?node-id=1-6962)
- Token/typography source: [`node 1-3939` (prototypes)](https://www.figma.com/design/EUSZZtDMNxsLrzJc2ufZCe/prototype?node-id=1-3939)

---

## Overview

A six-tab comparison module letting prospective students compare the FlexPath and
GuidedPath learning formats across Course schedule, How you learn, Tuition, Grading,
Programs, and Ideal for. Tab content swaps with a crossfade + slide; a "glider"
highlight slides between pills. Interaction model is inspired by masterclass.com's
category tabs.

All copy lives in one data array (`TABS` in
[`index.html` script section](https://github.com/camila-go/tabComponent/blob/main/index.html))
— content edits don't require touching markup.

## Layout

- Section max-width **1240px**, centered, `64px` top / `96px` bottom padding.
- **≥1024px:** tab pills centered in a single row; panels are a 2-column grid (`gap: 24px`), FlexPath left / GuidedPath right.
- **<1024px:** pills become a horizontally scrollable row (scrollbar hidden, edge-fade mask hints at off-screen tabs, active pill auto-centers via `scrollIntoView`); cards stack vertically.
- Container height animates between tab states (JS measures target panel, transitions `height`, releases to `auto`).

## Design Tokens Used

| Token (Figma) | Value | Usage |
|-------|-------|-------|
| `color/university/uni-black/gl-uni-black-500` | `#212322` | Page/section background |
| `color/usability/neutral/gl-neutral-0` | `#ffffff` | All text, active pill fill, CTA border |
| `color/usability/neutral/gl-neutral-0-opacity-1` | `#ffffff1a` | Inactive pill background |
| `color/usability/neutral/gl-neutral-0-opacity-2` | `#ffffff33` | Inactive pill border |
| `headings/h2 - desktop` | Acumin VF ExtraCondensed 600 · 48px · lh 0.9 · ls −1 | Section title ≥1024px |
| `headings/h2 - mobile` | Acumin VF ExtraCondensed 600 · 36px · lh 0.9 · ls −1 | Section title <1024px |
| `body/body - med` | Inter 400 · 16px · lh 1.5 | Subtitle, card checklist items |
| `headings/h4 - mobile / desktop` | Inter 700 · 22px / 28px · lh 1.2 | Card brand heading (see logo note) |
| `UI/ec-ui-med-default` | Inter 700 · 16px · lh 1.5 | CTA button label |
| `gl-number/border/gl-border-sm` | 2px | CTA button border |
| `gl-number/border-radius/gl-border-radius-full` | 360px (full) | Pills + CTA buttons |
| Card radius (from node `1:7146`) | 16px | Comparison cards |
| Card padding (desktop, node `1:7146`) | 32px / 48px | Comparison cards ≥1024px (28/24 mobile) |

## Components

| Component | Variant | Notes |
|-----------|---------|-------|
| Tab pill | inactive | 48px tall desktop / 40px mobile, icon (16px) + label, `#ffffff1a` fill, `#ffffff33` 1px border |
| Tab pill | active | White fill provided by the shared `.tabs__glider` layer (not per-pill styles); label flips to `#111` |
| Comparison card | `card--flex` / `card--guided` | Photo background per brand at ~30% over black (see Assets), 16px radius, checklist + optional CTA |
| CTA button | outline | 48px tall, 2px white border, full radius, arrow nudges +4px on hover, inverts to white/dark on hover |
| Grading tab cards | no CTA | The Grading panel has single-item cards and **no buttons** (per Figma) |

## States and Interactions

| Element | State | Behavior |
|---------|-------|----------|
| Tab pill | hover | Border brightens to `rgba(255,255,255,0.5)` |
| Tab pill | selected | Glider slides + resizes to pill (`transform`/`width`, 480ms), label color crossfades |
| Tab pill | focus-visible | 2px white outline, 3px offset |
| Panel | enter | Fade in + 14px rise; checklist items stagger in (90–270ms delays) |
| Panel | exit | Fade out; panel becomes `inert` immediately and `visibility` flips after the fade |
| Panels container | tab change | Height animates old→new, then releases to `auto` |
| CTA | hover | Fills white, text inverts, arrow translates 4px right |

## Responsive Behavior

| Breakpoint | Changes |
|------------|---------|
| ≥1024px | Tabs centered single row; cards side-by-side; title 48px; card padding 32/48; brand 28px |
| <1024px | Tabs scrollable with edge fade + auto-centering; cards stacked; title 36px; pills 40px tall |
| Verified at | 320 / 375 / 768 / 1024 / 1280 / 1680 — no horizontal overflow at any width |

## Edge Cases

- **No CTA** (Grading tab): card bottom padding holds; button block simply omitted.
- **Long labels:** pills use `white-space: nowrap` and the row scrolls; never wraps or truncates.
- **Tall→short content:** height animation prevents the footer jump; inactive panels are `position: absolute` so the active one defines height.
- **Reduced motion:** `prefers-reduced-motion` disables all transitions; JS skips the height pin entirely (no stuck fixed heights).
- **Slow font load:** glider re-anchors on `document.fonts.ready` so pill widths stay aligned after the Inter/Oswald swap.

## Animation / Motion

| Element | Trigger | Animation | Duration | Easing |
|---------|---------|-----------|----------|--------|
| Glider | tab select | `translateX` + `width` | 480ms | `cubic-bezier(0.22,1,0.36,1)` |
| Panel | tab select | opacity + `translateY(14px→0)` | 480ms | same |
| Checklist items | panel enter | staggered fade/rise | 420ms, +60ms/item | same |
| Panels container | tab select | height old→new→auto | 480ms | same |
| CTA arrow | hover | `translateX(4px)` | 240ms | same |

Architecture intent: no `display:none` (kills transitions); glider is a separate
transformed layer (no layout thrash); active panel `position: relative` / inactive
`absolute` so container height tracks content naturally.

## Accessibility Notes

- Proper tablist: `role="tablist"` / `role="tab"` / `role="tabpanel"`, `aria-selected`, `aria-controls`, `aria-labelledby`, `aria-hidden` on inactive panels.
- Roving tabindex; **ArrowLeft/ArrowRight** cycle (wrapping), **Home/End** jump; selection follows focus.
- Inactive panels are `inert` (instantly out of the focus order and accessibility tree) with `aria-hidden` and delayed `visibility: hidden` as backup, so transitions stay intact while hidden buttons stay unreachable.
- Focus ring: 2px outline offset 3px on `:focus-visible` only.

## Assets & Production TODOs

1. **Card background photos** — exported from Figma and committed:
   [`assets/card-bg-flex.jpg`](https://github.com/camila-go/tabComponent/blob/main/assets/card-bg-flex.jpg) (node `1:7146` fill) and
   [`assets/card-bg-guided.jpg`](https://github.com/camila-go/tabComponent/blob/main/assets/card-bg-guided.jpg) (node `1:7182` fill),
   1200×800 JPEG ~140KB each. Figma layers them at 30% opacity over black; CSS reproduces this with a `rgba(10,9,8,0.7)` gradient overlay so text contrast is preserved. *Production: confirm photo licensing and consider WebP/AVIF + `image-set()`.*
2. **Headings font** — design specifies **Acumin VF ExtraCondensed** (Adobe Font). The prototype loads **Oswald** from Google Fonts as a visual stand-in; the CSS stack already prefers `"Acumin VF", "acumin-pro-extra-condensed"` so dropping in the Adobe Fonts `<link>` kit makes it correct with no CSS change. **Dev needs:** the org's Adobe Fonts kit ID.
3. **Brand lockups** — Figma cards use logo images (`CU_FlexPath_level4p_White`, `CU_GuidedPath_level4p_White`, 30px tall); the prototype substitutes a shield icon + text heading. **Dev needs:** official SVG exports of both lockups.
4. **Tab icons** — Figma uses **Font Awesome 6 Pro** (`v6-icon` component); prototype uses hand-drawn inline SVGs sized 16×16. Swap to FA Pro classes if the production site already loads it.
5. **CTA destinations** — buttons are `type="button"` placeholders; URLs for Explore FlexPath / GuidedPath, trial courses, Learn more, and Find your program are needed.
