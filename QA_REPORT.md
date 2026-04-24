# QA Report — RepScroll Landing Page
**Reviewer**: QA Agent
**Date**: 2026-03-24
**File reviewed**: `website/index.html`
**Status**: All identified issues fixed. File is production-ready pending external assets.

---

## Issues Found and Fixed

### Accessibility (Critical)

**1. Missing skip-to-main-content link**
- **Problem**: No skip link existed, making keyboard-only navigation require tabbing through the entire navbar on every page load.
- **Fix**: Added `<a href="#main-content" class="skip-link">Skip to main content</a>` as the first focusable element in `<body>`. Added CSS that keeps it off-screen until focused, then reveals it at `top: 16px`.

**2. No `<main>` landmark element**
- **Problem**: Page content was not wrapped in a `<main>` element, meaning screen reader users had no way to jump to the primary content region via landmark navigation.
- **Fix**: Wrapped all sections (hero through download CTA) in `<main id="main-content">`. The `id` matches the skip-link target. Footer remains outside `<main>`.

**3. `aria-live` region suppressed inside `aria-hidden` parent**
- **Problem**: The `#demoCount` span had `aria-live="polite"` and `aria-atomic="true"`, but its immediate ancestor `.counter-ring-wrap` had `aria-hidden="true"`. The `aria-hidden` on the parent silences all ARIA roles and live regions within it, rendering the counter completely invisible to screen readers.
- **Fix**: Removed `aria-live` and `aria-atomic` from `#demoCount` (it stays inside `aria-hidden` for visual presentation). Added a dedicated visually-hidden `<div id="demoLiveRegion" aria-live="polite" aria-atomic="true">` placed outside the `aria-hidden` wrapper. JS now updates this element with human-readable status on each button press and on unlock/reset.

**4. Missing `:focus-visible` styles on interactive elements**
- **Problem**: No global focus ring styles existed. Keyboard users had no visible focus indicator on nav links, ghost button, App Store button, or footer links (browser defaults were the only fallback, which are often suppressed by CSS resets).
- **Fix**: Added `:focus-visible` rules for `a`, `button`, `.btn-appstore`, `.btn-ghost`, and `.nav-links a` — all with a `2px solid var(--accent-purple)` outline with `outline-offset: 3px`. Pill-shaped buttons get `border-radius: var(--radius-pill)` on the outline to match their shape.

**5. Redundant `tabindex="0"` on `<button>`**
- **Problem**: The `#demoPushBtn` button had `tabindex="0"` applied. Native `<button>` elements are already in the tab order; the attribute was redundant and potentially confusing.
- **Fix**: Removed the attribute.

---

### CSS Issues

**6. No `prefers-reduced-motion` support**
- **Problem**: Multiple animations (hero ring rotation, eyebrow dot pulse, parallax scroll-jacking, orb mouse-move animation, counter ring fill transition, progress bar grow animation, bounce-in keyframes, count-up counters) all ran unconditionally. Users with vestibular disorders, epilepsy, or motion sensitivity had no protection.
- **Fix**: Added two layers of `prefers-reduced-motion` protection:
  - **CSS**: `@media (prefers-reduced-motion: reduce)` block that sets `animation-duration: 0.01ms`, `transition-duration: 0.01ms`, disables `scroll-behavior: smooth`, forces `.animate` elements to be immediately visible, and explicitly nullifies `will-change: transform` on the parallax layer and orbs.
  - **JS**: `window.matchMedia('(prefers-reduced-motion: reduce)').matches` check gates the orb parallax loop, the hero scroll-parallax listener, and all count-up animations — instead showing final values immediately when reduced motion is preferred.

**7. `scroll-behavior: smooth` not gated by reduced-motion**
- **Problem**: `html { scroll-behavior: smooth; }` ran for all users including those who prefer reduced motion.
- **Fix**: Added `@media (prefers-reduced-motion: reduce) { html { scroll-behavior: auto; } }` immediately after the base rule.

**8. No `backdrop-filter` fallback for unsupported browsers (Firefox < 103, older Chrome)**
- **Problem**: Glass cards, the navbar, and the ghost button all used `backdrop-filter: blur()` for their frosted-glass effect with no solid-color fallback. On browsers that don't support `backdrop-filter`, these elements rendered nearly transparent against the dark background, making text hard to read.
- **Fix**: Added `@supports not (backdrop-filter: blur(1px))` block providing opaque fallback backgrounds: `.glass` → `rgba(28,28,46,0.92)`, `.glass--subtle` → `rgba(20,20,36,0.90)`, `.glass--raised` → `rgba(36,36,58,0.95)`, `nav` → `rgba(10,10,20,0.97)`, `.btn-ghost` → `rgba(20,20,36,0.90)`.

---

### JavaScript Issues

**9. All JS variables leaked to global scope**
- **Problem**: Every variable and function — `scrollProgress`, `observer`, `orbPurple`, `orbBlue`, `rafId`, `ticking`, `sections`, `navLinks`, `sectionObserver`, `counterData`, `heroObserver`, `socialObserver`, `heroTicking` — was declared at the top level of the `<script>` block, polluting the global `window` object. This could cause conflicts with any third-party scripts added later (analytics, A/B testing, etc.).
- **Fix**: Wrapped the entire script body in an IIFE `(function () { 'use strict'; ... })();`. All variables are now scoped to the IIFE closure.

**10. `scrollProgress` used without null check**
- **Problem**: `scrollProgress.style.width = ...` was called directly without first checking that `document.getElementById('scrollProgress')` returned a non-null value. A typo in the HTML or a CSP-stripped element would cause an uncaught TypeError.
- **Fix**: Renamed to `scrollProgressEl`, added `if (!scrollProgressEl) return;` guard at the top of `updateScrollProgress()`.

**11. `orbPurple` / `orbBlue` used without null checks**
- **Problem**: `animateOrbs()` called `orbPurple.style.transform` and `orbBlue.style.transform` unconditionally. If either element is missing, the function throws on every rAF tick (hundreds of times per second).
- **Fix**: The entire orb animation block is now gated with `if (!prefersReducedMotion && orbPurple && orbBlue)`.

**12. `counting` class never removed for integer counters in social stat section**
- **Problem**: The social stat count-up set `el.classList.add('counting')` before animating, and the float path removed it on completion. However, the integer path called `animateCounter(el, ...)` which had no callback — so `counting` (which sets `will-change: contents`) was never removed for integer entries (362K and 28M+), leaving those elements permanently in a promoted layer state.
- **Fix**: Added an `onComplete` callback parameter to `animateCounter()`. The social observer now passes `() => el.classList.remove('counting')` as the callback for integer entries. Float entries already handled it inline.

**13. Parallax direction inverted**
- **Problem**: `heroParallax.style.transform = \`translateY(${scrollY * 0.22}px)\`` translated the hero content **downward** (positive Y) as the user scrolled down, which is the opposite of the intended parallax depth effect (content should drift upward relative to the viewport to create perceived depth).
- **Fix**: Changed to `translateY(${scrollY * -0.22}px)` (negative multiplier).

**14. Counter demo unlock timer could theoretically stack**
- **Problem**: The `showUnlock()` function used `setTimeout` for auto-reset but stored the timeout IDs only in variables without clearing them on re-entry. While the `unlocked` flag prevents double-triggering via the click handler, having unnamed timeouts could theoretically cause issues if the code were ever modified.
- **Fix**: Added `resetTimeout1` and `resetTimeout2` variables. `showUnlock()` now calls `clearTimeout()` on both before setting new timers, making the reset sequence idempotent.

**15. `repsLeftEl` not null-checked in counter IIFE**
- **Problem**: The early-return guard was `if (!countEl || !ringEl || !btnEl) return` — but `repsLeftEl` was used in `repsLeftEl.textContent = GOAL` without being included in the guard.
- **Fix**: All accesses to `repsLeftEl` are now guarded with `if (repsLeftEl)` inline checks, and `unlockEl` was added to the early-return guard since it is also required.

---

### Performance

**16. `prefersReducedMotion` check skips count-up animations**
- When reduced motion is preferred, hero stat numbers and social stat numbers are now set to their final values immediately without running rAF animation loops. This is both an accessibility fix and a performance improvement (no unnecessary rAF scheduling for users who don't want animation).

---

### Content

**17. Twitter handle confirmed as `@repscrollapp`**
- The `twitter:site` meta tag reads `@repscrollapp`. Per the Developer's own note, this is a placeholder. The value is consistent across the page. **Action required before launch**: Update to the real account handle once the account is created.

**18. App Store CTA uses `href="#"` as placeholder**
- The primary download button in the `#download` section uses `href="#"`. This is intentional and acceptable as a pre-launch placeholder. **Action required before launch**: Replace with the real App Store URL (format: `https://apps.apple.com/app/id<YOUR_APP_ID>`).

---

## Remaining Known Limitations (Require External Assets or Server)

| Item | Detail | Owner |
|------|--------|-------|
| `og:image` / `twitter:image` | Both point to `https://repscroll.app/og-image.png` which does not exist. Social shares will show no preview image until this 1200×630px asset is created and deployed. | Design / Marketing |
| `apple-touch-icon` | Points to `https://repscroll.app/apple-touch-icon.png`. Must be exported from the Xcode asset catalog (180×180px PNG) and deployed at that URL. | Dev / Infra |
| App Store URL | The `href="#"` on all download buttons needs a real App Store link once the app is submitted. | PM |
| Twitter handle | `@repscrollapp` is a placeholder; update `twitter:site` once the account is live. | Marketing |
| Changelog / About / Blog / Press / Contact / Legal pages | All footer links use `href="#"`. These need real destinations. | Content / Dev |

---

## Overall Quality Assessment

**Pre-fix grade**: B− (visually polished, but accessibility and JS quality had serious gaps)
**Post-fix grade**: A−

The Designer and Developer produced a visually excellent, well-structured landing page with strong semantic HTML, good ARIA labeling on most elements, and a thoughtfully implemented interactive demo. The main issues were:

1. Accessibility gaps that would fail WCAG 2.1 AA — now resolved (skip link, `<main>` landmark, `aria-live` fix, focus rings).
2. No `prefers-reduced-motion` support — now fully implemented in both CSS and JS.
3. No glass card fallback for older Firefox — now implemented with `@supports`.
4. JS global scope pollution — now fully contained in an IIFE.
5. Several null-check omissions that could cause runtime errors — all guarded.

The page is now production-ready from a code quality standpoint. The only remaining blockers before public launch are the external asset requirements listed in the table above.
