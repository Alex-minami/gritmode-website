# Designer Handoff — RepScroll Landing Page

## What was built

A single-file `index.html` landing page for the RepScroll iOS app, fully self-contained with embedded CSS and vanilla JS.

### Sections delivered
1. **Navbar** — fixed, glassmorphic, hides links on mobile, scroll-progress bar across the top
2. **Hero** — full-viewport dark section, animated headline, App Store CTA, live stat strip with counter animation on scroll-in
3. **How It Works** — 3-step glass card layout with connector arrows (desktop only), staggered scale-in animations
4. **Features** — 12-column CSS Grid layout with 5 feature cards: AI Detection (wide), Screen Time API, Stats, Achievements, Widgets — each with micro UI previews
5. **Social Proof** — stat numbers strip + 3 testimonial glass cards
6. **Download CTA** — centered section with ambient orb background, headline + App Store button
7. **Footer** — brand blurb + 3 link columns + legal row

### Design system applied
- Colors: exact token values from `docs/DESIGN.md` (`#0A0A14`, `#0E0E1A`, `#1C1C2E`, glass white opacities)
- Fonts: DM Serif Display (headlines) + DM Sans (UI), loaded via Google Fonts
- Glass cards: 3 intensity levels (subtle / default / raised), `backdrop-filter: blur(16–28px)`
- Ambient orbs: `position:fixed`, radial gradients — purple top-right (`#6430FF` 28%), blue bottom-left (`#1490FF` 18%), with mouse-parallax
- Corner radii: 22px cards, 14px rows, 50px pills
- Animations: `easeOut` 0.6s, staggered delays via `.delay-1..5` classes

### JavaScript implemented
- `IntersectionObserver` triggers `.in-view` class on scroll — drives all entrance animations
- Animated counter for hero stats (easeOutExpo interpolation)
- Mouse-move parallax on ambient orbs (lerped via `requestAnimationFrame`)
- Scroll progress bar (throttled via `rAF`)
- Active nav link highlight via section intersection observer

---

## What the Developer agent should add next

### High priority
1. **Real App Store link** — replace `href="#"` on the Download buttons with the live App Store URL
2. **Phone mockup / screenshot** — insert an actual app screenshot or device frame in the Hero section (currently uses animated ring decoration as placeholder)
3. **Form / waitlist** — if not yet live, add an email capture form in the Download CTA with backend integration (e.g. Mailchimp, Supabase)
4. **OG / Social meta tags** — `og:image`, `og:title`, `og:description`, Twitter card tags
5. **Favicon** — link to the app icon asset already in `PushUp/Assets.xcassets/AppIcon.appiconset/`

### Enhancements
6. **Video demo** — replace or augment the Features AI card with a short autoplay muted `.mp4` showing rep detection in action
7. **Animated rep counter demo** — live incrementing number in the AI card to better demonstrate the feature
8. **Scroll-linked orb breathing** — shift orb position subtly based on `window.scrollY` for depth
9. **Dark/light mode toggle** — currently dark-only; a toggle could be wired via `prefers-color-scheme`
10. **Cookie consent banner** — required for GDPR if targeting European users

### Performance
11. **Font `font-display: swap`** is already set via Google Fonts URL; verify LCP score in Lighthouse
12. **Image optimisation** — once screenshots are added, use `loading="lazy"` and serve WebP
13. **Critical CSS inlining** — the embedded `<style>` block already handles this for above-the-fold content

### SEO
14. Add `<link rel="canonical">` once the domain is live
15. Add structured data (`@type: MobileApplication`) for App Store rich results

---

## File location
`/Users/mr.president/Downloads/Apps/PushUp/website/index.html`

No build step required — open directly in a browser or deploy the `website/` directory to any static host (Vercel, Netlify, GitHub Pages).
