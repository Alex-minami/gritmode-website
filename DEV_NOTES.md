# Developer Notes — RepScroll Landing Page Enhancement

**Author**: Developer Agent
**Date**: 2026-03-24
**File modified**: `website/index.html` (no new files created)

---

## Changes Made

### 1. Meta Tags
- Added `<link rel="canonical" href="https://repscroll.app">` in `<head>`
- Added `<link rel="apple-touch-icon" sizes="180x180">` pointing to `https://repscroll.app/apple-touch-icon.png`
- Added `<meta name="apple-mobile-web-app-title">`
- Added full Open Graph tag set: `og:type`, `og:url`, `og:title`, `og:description`, `og:image` (1200×630), `og:site_name`
- Added Twitter Card tags: `summary_large_image` card type, `twitter:site`, `twitter:title`, `twitter:description`, `twitter:image`

### 2. Hero Scroll-Jacking Parallax
- Wrapped all hero content in `<div id="heroParallax" class="hero-parallax-content">` with `will-change: transform`
- Added a second passive scroll listener that applies `translateY(scrollY * 0.22)` — hero text drifts upward at ~22% of scroll rate, creating a subtle depth layer effect relative to the fixed orbs
- Uses the existing `rAF` ticking pattern to prevent layout thrash

### 3. Interactive Exercise Counter Demo
- Inserted after the 3-step grid in `#how` section
- CSS-only SVG progress ring (r=60, circumference≈377px), filled via `stroke-dashoffset` transition
- GOAL = 5 push-ups → reveals "5 minutes earned" green flash overlay
- Green overlay auto-resets after 2.8 s, counter resets so users can replay
- All DOM writes batched inside a single `requestAnimationFrame` callback to avoid layout thrash
- Keyboard accessible: `tabindex="0"`, `Enter`/`Space` handlers, `aria-label` on button, `aria-live="polite"` on count display, `aria-live="assertive"` on unlock overlay
- `role="group"` + `aria-label` on the widget container

### 4. iPhone Mockup Carousel
- Three CSS-only device frames placed after the features grid (before Social Proof)
- Pure CSS `border-radius: 42px`, dynamic island notch via child `div`, side buttons via `::before`/`::after` pseudo-elements — zero images
- Three simulated screens: Rep Tracker, Blocked Apps, Achievements
- Center frame is scaled up (`scale(1.04) translateY(24px)`) to create depth; side frames are `scale(0.96) opacity(0.75)`
- `will-change: transform` on `.iphone-frame`; hover lifts all frames uniformly
- `role="list"` / `role="listitem"` on the carousel; `aria-label` on each device frame

### 5. Card Glow Micro-Interactions
- Added `border-color` + `box-shadow` transition to base `.glass` class (does not override per-component `transform` transitions to avoid cascade conflicts)
- On `:hover` / `:focus-within`: border brightens to `rgba(255,255,255,0.20)`, adds a soft outer glow (`0 0 0 1px rgba(255,255,255,0.08)`)

### 6. Button Spring Feel
- Overrode `.btn-appstore` and `.btn-ghost` transition to use `cubic-bezier(0.34, 1.56, 0.64, 1)` — this is a spring easing (slight overshoot then settle)
- `:active` state scales down to `0.97` on both

### 7. Stat Count-Up on Social Proof Numbers
- Added `IntersectionObserver` on `.social-numbers` element (same easeOutExpo interpolation as the hero stat counters already present)
- Targets: 4.8M, 4.9, 362K, 28M+
- Adds `will-change: contents` (class `counting`) during animation, removes after

### 8. Section Accessibility
- Added `role="region"` + `aria-labelledby` to all five `<section>` elements (`#hero`, `#how`, `#features`, `#social`, `#download`)
- Added matching `id` attributes to all section headings
- Added `aria-hidden="true"` to the down-arrow SVG in the "See how it works" button, and `aria-label` on the `<a>` element

### 9. Scroll Progress Bar
- Already present from the Designer's build; confirmed `passive: true` on the scroll listener — no changes needed

---

## Known Issues / QA Checklist

1. **OG image placeholder** — `og:image` and `twitter:image` both point to `https://repscroll.app/og-image.png` which does not yet exist. QA should verify this asset is created and uploaded before launch.

2. **apple-touch-icon** — points to `https://repscroll.app/apple-touch-icon.png` (not a local path). If the domain isn't live, the icon won't resolve. The actual `.png` asset in `PushUp/Assets.xcassets/AppIcon.appiconset/` should be exported and placed at this URL.

3. **Hero parallax on mobile** — `will-change: transform` on `#heroParallax` promotes a new compositor layer. On very low-end devices this could increase memory pressure. Consider a `prefers-reduced-motion` media query to disable the parallax (currently not implemented).

4. **Counter demo ring radius** — the SVG circle `r="60"` with viewBox `0 0 140 140` means the track is inside the SVG bounds. If the container renders narrower than 140px (unlikely but possible on very small phones), the ring may be cropped. Visual QA on iPhone SE (375px) recommended.

5. **`glass` transition override specificity** — the base `.glass` now has `transition: box-shadow, border-color`. Components like `.step-card` and `.feat-card` declare `transition: transform 0.3s` which will cascade alongside the base rule. Test in Safari and Firefox to confirm no stutter on cards.

6. **Social stat counter data order** — the `socialCounterData` array assumes the DOM order of `.social-stat-num` elements matches [Users, Rating, Workouts, Total Reps]. If the HTML order ever changes, the counts will mis-match.

7. **Twitter `@repscrollapp` handle** — placeholder handle used. Update `twitter:site` once the real account is live.

8. **Prefers-reduced-motion** — no `@media (prefers-reduced-motion: reduce)` guards have been added for the hero parallax or counter ring animation. Recommend adding before public launch for accessibility compliance.

9. **iPhone mockup `margin-top: auto`** — the `mock-row` "Total reps" line in the Achievements mockup uses `margin-top: auto` to push to the bottom. On smaller frame widths this may overlap badge rows. Visual check at `width: 180px` (mobile breakpoint) recommended.
