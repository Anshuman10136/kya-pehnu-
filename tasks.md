# Implementation Plan: Kya Pehnu? Website

## Overview

Incremental implementation of the Kya Pehnu? cinematic fashion-tech website. Tasks follow a strict dependency order: scaffold → configuration → providers/hooks → shared UI primitives → layout shell → intro sequence → 3D viewer → scenes (01–05) → delivery/tracker/app/social/brand/final → page assembly → SEO → responsive/a11y/performance polish.

Each task builds directly on the previous, ensuring no orphaned code. All scenes are wired together in the final assembly task.

---

## Tasks

- [x] 1. Scaffold project and install dependencies
  - Initialise Next.js 14 App Router project with TypeScript and Tailwind CSS via `create-next-app`
  - Add ESLint config (next/core-web-vitals), Prettier, and `tsconfig.json` strict mode
  - Install all key dependencies: `three`, `@react-three/fiber`, `@react-three/drei`, `gsap`, `@studio-freight/lenis`, `framer-motion`, `clsx`
  - Create the full directory skeleton from the design: `components/`, `hooks/`, `lib/`, `data/`, `config/`, `public/models/`, `public/images/`, `public/audio/`, `public/fonts/`, `styles/`, `types/`
  - Add `globals.css` with Tailwind base directives, `cursor: none` rule for desktop, and smooth-scroll reset
  - _Requirements: 20.3 (bundle budget), 21.6 (semantic HTML scaffold)_

- [x] 2. Create configuration file
  - [x] 2.1 Write `config/site.config.ts`
    - Export `siteConfig` const with all keys: `IOS_APP_DOWNLOAD_LINK`, `ANDROID_APP_DOWNLOAD_LINK`, `PARTNER_CONTACT_URL`, `SOCIAL_INSTAGRAM`, `SOCIAL_LINKEDIN`, `SOCIAL_X`, `DELIVERY_ETA_START_MINUTES`, `STATS_DELIVERIES_COUNT`, `STATS_SATISFACTION_RATE`, `STATS_AVERAGE_RATING`, `REVIEWS` (3+ entries), `FOOTER_LINKS`
    - Add inline JSDoc comment on every key describing purpose and expected format
    - _Requirements: 22.1, 22.2, 22.3, 22.4_

  - [ ]* 2.2 Write unit tests for site.config
    - Assert all required keys are present and non-undefined
    - Assert `REVIEWS` array has at least 3 entries each with `name`, `rating`, `text`
    - _Requirements: 22.2_

- [x] 3. Define shared TypeScript types
  - [x] 3.1 Write `types/index.ts`
    - Define: `VibeCategory`, `Destination`, `StylePreference`, `OutfitKey`, `Outfit`, `Product`, `AppScreen`, `Review`, `IntroState` (`'loading' | 'intro' | 'hero'`)
    - Export all types; no runtime logic in this file
    - _Requirements: 5.1 (VibeCategory), 6.1 (Destination/StylePreference), 10.2 (AppScreen)_

- [ ] 4. Create placeholder data files
  - [x] 4.1 Write `data/vibes.ts`
    - Export `VIBES` array covering all 7 categories: DATE NIGHT, COLLEGE, PARTY, OFFICE, STREET, CASUAL, WEEKEND
    - Each entry: `{ id, label, palette: { bg, accent }, modelVariant }`
    - _Requirements: 5.1, 5.3_

  - [x] 4.2 Write `data/products.ts`
    - Export `PRODUCTS` array with at least 12 placeholder products
    - Each entry: `{ id, name, price, category, imageUrl, deliveryLabel }`; `category` maps to Scene 05 tabs (MEN, WOMEN, STREETWEAR, PARTY, CASUAL, FOOTWEAR, ACCESSORIES)
    - _Requirements: 7.1, 7.3_

  - [x] 4.3 Write `data/outfits.ts`
    - Export `outfitMap: Record<OutfitKey, Outfit>` covering representative combinations of Destination × StylePreference
    - Each `Outfit`: `{ products, totalPrice, deliveryEta }`
    - _Requirements: 6.2, 6.4_

  - [x] 4.4 Write `data/appScreens.ts`
    - Export `APP_SCREENS` array with at least 6 entries: Home, Discover, AI Stylist, Product, Cart, Order Tracking
    - Each entry: `{ id, label, imageUrl }`
    - _Requirements: 10.2_

- [x] 5. Implement core library helpers
  - [x] 5.1 Write `lib/gsap.ts`
    - Register `ScrollTrigger` and `SplitText` GSAP plugins; export configured `gsap` instance
    - Guard registration to run client-side only
    - _Requirements: 17.1, 17.2_

  - [x] 5.2 Write `lib/lenis.ts`
    - Export `createLenis()` factory: instantiates Lenis with `lerp: 0.1, smoothWheel: true`; integrates with `gsap.ticker`; returns lenis instance
    - Accept `mobile: boolean` parameter — when true, set `smoothWheel: false`
    - _Requirements: 17.3_

  - [x] 5.3 Write `lib/three-utils.ts`
    - Export helper to create cinematic three-point lighting setup (key, fill, rim)
    - Export material helper for fallback silhouette mesh
    - _Requirements: 2.1_

- [ ] 6. Implement custom hooks
  - [x] 6.1 Write `hooks/useMobileDetect.ts`
    - Return `{ isMobile: boolean, isTouch: boolean }` based on `window.innerWidth < 768` and touch capability
    - SSR-safe (returns false during server render)
    - _Requirements: 16.5, 19.2_

  - [x] 6.2 Write `hooks/useWebGLSupport.ts`
    - Detect WebGL availability by attempting to create a WebGL context on a test canvas
    - Return `{ supported: boolean }`; SSR-safe
    - _Requirements: 2.6_

  - [x] 6.3 Write `hooks/useMousePosition.ts`
    - Track `mousemove` events; return normalised `{ x, y }` in `[-1, 1]` relative to viewport
    - Attach and clean up listener in `useEffect`
    - _Requirements: 2.2, 16.2_

  - [x] 6.4 Write `hooks/useScrollProgress.ts`
    - Accept a ref and optional ScrollTrigger config; return `progress: number` in `[0, 1]`
    - Clean up ScrollTrigger instance on unmount
    - _Requirements: 3.2, 4.2, 8.2_

  - [x] 6.5 Write `hooks/useReducedMotion.ts`
    - Read `prefers-reduced-motion` media query; return `boolean`
    - _Requirements: 23 (implicit), 17.1_

- [x] 7. Implement global providers
  - [x] 7.1 Write `components/providers/AppProvider.tsx`
    - Manage context: `introState: IntroState`, `soundEnabled: boolean`, `activeVibe: VibeCategory`, `isMobile: boolean`
    - Expose `setIntroState`, `setSoundEnabled`, `setActiveVibe` actions
    - _Requirements: 1.3, 5.3, 18.2_

  - [x] 7.2 Write `components/providers/ScrollProvider.tsx`
    - Initialise Lenis via `createLenis()` on mount; expose instance and `scrollY` via context
    - On mobile, use `smoothWheel: false`; destroy on unmount
    - _Requirements: 17.3_

- [x] 8. Implement shared UI primitives
  - [x] 8.1 Write `components/ui/SplitText.tsx`
    - Accept `text: string` and `trigger: RefObject` props
    - Use GSAP `SplitText` to split into chars/words, animate in on ScrollTrigger entry
    - On mobile or `prefers-reduced-motion`, fall back to Framer Motion `whileInView` opacity transition
    - _Requirements: 17.1, 3.1, 13.1_

  - [x] 8.2 Write `components/ui/ImageReveal.tsx`
    - Accept `src`, `alt`, `className` props; wrap `next/image` in a clip-path container
    - Animate `clip-path: inset(100% 0 0 0)` → `inset(0% 0 0 0)` on ScrollTrigger entry
    - _Requirements: 17.2_

  - [x] 8.3 Write `components/ui/MagneticButton.tsx`
    - Forward all button props; track `mouseenter`, `mousemove`, `mouseleave`
    - Use `gsap.quickSetter` to shift position up to 12px toward cursor
    - On `mouseleave`, spring back with `elastic.out(1, 0.3)` easing
    - Disable all handlers when `isTouch` is true
    - _Requirements: 16.2, 16.3, 16.4, 16.5_

  - [x] 8.4 Write `components/ui/CustomCursor.tsx`
    - Fixed `div` with `pointer-events: none, z-index: 9999`; two concentric circles (dot + ring)
    - Track `mousemove` with `gsap.quickSetter`; ring lags behind dot for trailing effect
    - Render only when viewport ≥ 1024px and not a touch device
    - _Requirements: 16.1, 16.5_

  - [x] 8.5 Write `components/ui/SoundController.tsx`
    - Fixed-position `SOUND ON / SOUND OFF` toggle button
    - Default state: muted; reads/writes `soundEnabled` from `AppProvider`
    - Play ambient audio at `volume = 0.3`; pause on tab visibility change; resume on return if was ON
    - Render `aria-live="polite"` region announcing "Sound enabled" / "Sound disabled"
    - _Requirements: 18.1, 18.2, 18.3, 18.4, 18.5, 23.5_

  - [x] 8.6 Write `components/ui/DeliveryMap.tsx`
    - Render an abstract stylised SVG map with a store pin and destination pin connected by a `<path>`
    - Animate a circle along the path via `stroke-dashoffset` driven by a `progress` prop (0–1)
    - _Requirements: 8.3, 8.4_

- [ ] 9. Implement 3D viewer components
  - [x] 9.1 Write `components/three/SceneLighting.tsx`
    - Place key light, fill light, and rim light using `pointLight` / `directionalLight` R3F primitives
    - Export as a composable component dropped inside `Canvas`
    - _Requirements: 2.1_

  - [x] 9.2 Write `components/three/ModelFallback.tsx`
    - Render animated branded silhouette mesh or a pulsing placeholder while GLB loads
    - Also used when no GLB file is present
    - _Requirements: 2.5, 20.6_

  - [x] 9.3 Write `components/three/FashionModel.tsx`
    - Accept `modelPath: string | null` prop
    - When `modelPath` is provided: load GLB with `useGLTF`, wrap in `<Suspense fallback={<ModelFallback/>}>`
    - When `modelPath` is null: render `<ModelFallback />`
    - Apply mouse-follow rotation via `useMousePosition` and `THREE.MathUtils.lerp`, clamped to ±15°
    - Accept `vibeVariant` prop to swap material/model when vibe changes (≤ 600ms transition)
    - _Requirements: 2.2, 2.4, 2.5, 5.2_

  - [x] 9.4 Write `lib/getModelPath.ts` (server utility)
    - Read `/public/models/` directory at build time; return path of first `.glb`/`.gltf` found or `null`
    - _Requirements: 2.4_

  - [-] 9.5 Write `components/three/ThreeViewer.tsx`
    - Check WebGL support via `useWebGLSupport`; if unsupported render static image fallback
    - When supported: render R3F `<Canvas>` with `dpr` and `shadows` props driven by `useMobileDetect`
    - Use `frameloop="demand"` for performance
    - Compose `SceneLighting`, `FashionModel` inside Canvas
    - Pass `aria-label="3D interactive fashion model display"` on the canvas wrapper
    - _Requirements: 2.1, 2.6, 2.7, 23.4_

  - [ ]* 9.6 Write unit tests for 3D utilities
    - Test `getModelPath` returns null when directory is empty
    - Test `ModelFallback` renders without crashing
    - _Requirements: 2.4, 2.5_

- [ ] 10. Implement loading sequence and cinematic intro
  - [-] 10.1 Write `components/intro/LoadingSequence.tsx`
    - Display full-screen black overlay with a branded progress bar
    - RAF-driven fake counter capped at 90%; jumps to 100% when assets resolve
    - On completion, call `setIntroState('intro')`
    - If 5-second timeout fires before assets resolve, call `setIntroState('hero')` and log to console
    - _Requirements: 1.4, 1.5_

  - [-] 10.2 Write `components/intro/CinematicIntro.tsx`
    - Black background, white typography only
    - GSAP timeline sequences through exactly 6 cards: "KYA PEHNU?" → "WHAT ARE YOU WEARING TONIGHT?" → "DON'T KNOW?" → "WE'LL FIX THAT." → "NEW OUTFIT. UNDER 60 MINUTES." → CTA "ENTER EXPERIENCE →"
    - CTA is a `MagneticButton`; on click run 800ms cinematic wipe then call `setIntroState('hero')`
    - Mount only when `introState === 'intro'`
    - _Requirements: 1.1, 1.2, 1.3_

- [ ] 11. Implement layout shell components
  - [-] 11.1 Write `components/layout/Nav.tsx`
    - Fixed top bar; brand name top-left, links (SHOP, STYLIST, HOW IT WORKS, APP) centre/right, SEARCH / CART / ACCOUNT icons far right
    - Smooth-scroll to section on link click
    - Transition background from transparent to `bg-black/70 backdrop-blur` when `scrollY > 80` within 300ms
    - `aria-label` on all icon-only buttons
    - _Requirements: 14.1, 14.2, 14.3, 14.4, 23.1_

  - [ ] 11.2 Write `components/layout/MobileMenu.tsx`
    - Full-screen cinematic overlay triggered by hamburger icon in Nav
    - Display all nav links in large typographic style
    - Close via ✕ button or Escape key
    - Render only when viewport < 768px (or always accessible via icon)
    - _Requirements: 14.5, 14.6_

  - [~] 11.3 Write `components/layout/Footer.tsx`
    - Logo + tagline "NEW OUTFIT. UNDER 60 MINUTES." at top
    - Four link groups (Explore, Company, Help, Download) sourced from `siteConfig.FOOTER_LINKS`
    - Social links (Instagram, LinkedIn, X) with `aria-label` on each
    - Copyright notice "© 2026 Kya Pehnu?"
    - All URLs read from `siteConfig`
    - _Requirements: 15.1, 15.2, 15.3, 15.4, 15.5, 23.1_

- [ ] 12. Implement Scene 01 — The Problem
  - [~] 12.1 Write `components/scenes/Scene01Problem.tsx`
    - Render headline "PLANS CHANGED? OUTFIT DID TOO." wrapped in `<SplitText>`
    - Bind scroll progress to content `opacity` and `translateY` for parallax depth effect via ScrollTrigger
    - On full-viewport detection, snap to `opacity: 1`, `transform: none` immediately
    - _Requirements: 3.1, 3.2, 3.3_

- [ ] 13. Implement Scene 02 — The Solution
  - [~] 13.1 Write `components/scenes/Scene02Solution.tsx`
    - Render headline "SO WE MADE IT FAST. REALLY FAST. UNDER 60 MINUTES." via `<SplitText>`
    - Drive a countdown display from 60:00 to 00:00 via a GSAP timeline with `scrub: 1` linked to ScrollTrigger
    - Counter state (3600 → 0) is the source of truth — scroll position pauses/resumes automatically
    - Ensure countdown completes before section exits
    - _Requirements: 4.1, 4.2, 4.3_

- [ ] 14. Implement Scene 03 — Vibe Selector
  - [~] 14.1 Write `components/scenes/Scene03Vibe.tsx`
    - Render the 7 vibe category pills from `VIBES` data
    - On category click: update `activeVibe` in `AppProvider`, trigger `ThreeViewer` model variant swap (≤ 600ms), update background palette, render matching product cards
    - Highlight active category with visual indicator (underline / colour inversion)
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

- [ ] 15. Implement Scene 04 — AI Stylist
  - [~] 15.1 Write `components/scenes/Scene04AIStylist.tsx`
    - Display prompt "DON'T KNOW WHAT TO WEAR?" and dropdown/selector inputs for Destination and Style Preference
    - On selection of both + trigger: look up `outfitMap[key]`, reveal product cards with GSAP 100ms stagger, display "YOUR LOOK IS READY.", total price, and ETA
    - Inline validation if either selector is empty — do not trigger recommendation; hide validation once both selected + result shown
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 16. Implement Scene 05 — Shopping Catalog
  - [~] 16.1 Write `components/scenes/Scene05Shop.tsx`
    - Category tabs: MEN, WOMEN, STREETWEAR, PARTY, CASUAL, FOOTWEAR, ACCESSORIES
    - On desktop: horizontally scrollable catalog track driven by GSAP ScrollTrigger pin + X translation
    - On mobile: native `overflow-x: scroll` with `scroll-snap-type: x mandatory` and touch swipe
    - Each product card: name, price, `⚡ ARRIVES IN UNDER 60 MIN` label
    - On desktop hover: overlay with product name, price, "ADD TO CART" action
    - Category tab switch: filter products with ≤ 400ms transition
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

- [ ] 17. Implement Delivery Journey animation
  - [~] 17.1 Write `components/scenes/DeliveryJourney.tsx`
    - Pin section for 500vh scroll travel via ScrollTrigger
    - Map `progress [0,1]` → step index 0–4 (ORDER, PICK, PACK, RIDE, YOUR DOOR) in equal 20% increments
    - Display label + icon for each active step at full opacity
    - Pass RIDE step local progress (range `[0.6, 0.8]`) to `<DeliveryMap>` for path animation
    - _Requirements: 8.1, 8.2, 8.3, 8.5_

- [ ] 18. Implement Delivery Tracker mock
  - [~] 18.1 Write `components/scenes/DeliveryTracker.tsx`
    - Display 5 milestones: "Order confirmed ✓", "Picked from store ✓", "Packed ✓", "Out for delivery →", "Arriving soon"
    - Loop through milestones sequentially, each visible ≥ 1500ms
    - ETA countdown decrementing from `siteConfig.DELIVERY_ETA_START_MINUTES`
    - Display "DEMO / MOCK EXPERIENCE" label clearly; tracker continues regardless of label render
    - _Requirements: 9.1, 9.2, 9.3, 9.4_

- [ ] 19. Implement App Section
  - [~] 19.1 Write `components/scenes/AppSection.tsx`
    - CSS 3D perspective phone mockup: `perspective(1000px) rotateY(-15deg) rotateX(5deg)`, subtle scroll parallax
    - Cycle through `APP_SCREENS` with Framer Motion `AnimatePresence` slide transition every 2500ms; pause when out of viewport
    - iOS CTA reads `IOS_APP_DOWNLOAD_LINK`; Android CTA reads `ANDROID_APP_DOWNLOAD_LINK`
    - If either link is empty/unset, render that button as disabled with label "COMING SOON"
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

- [ ] 20. Implement Social Proof section
  - [~] 20.1 Write `components/scenes/SocialProof.tsx`
    - Headline "LOOK GOOD. WITHOUT THE WAIT."
    - Display aggregate star rating (`STATS_AVERAGE_RATING`), total deliveries count (`STATS_DELIVERIES_COUNT`), satisfaction rate (`STATS_SATISFACTION_RATE`) — all from `siteConfig`
    - Render at least 3 customer review cards from `siteConfig.REVIEWS`; each card: name, star rating, review text
    - _Requirements: 11.1, 11.2, 11.3, 11.4_

- [ ] 21. Implement Brand Partners section
  - [~] 21.1 Write `components/scenes/BrandPartners.tsx`
    - Headline "THE BRANDS YOU LOVE. DELIVERED FASTER."
    - Horizontal marquee or grid of `Partner_Logo` placeholders
    - Desktop hover: scale/brightness animation on each logo
    - "BECOME A PARTNER →" CTA links to `siteConfig.PARTNER_CONTACT_URL` (wrapped in `MagneticButton`)
    - _Requirements: 12.1, 12.2, 12.3, 12.4_

- [ ] 22. Implement Final Cinematic Scene
  - [~] 22.1 Write `components/scenes/FinalScene.tsx`
    - Scroll-triggered reveal of "SO… KYA PEHNU? WE'VE GOT YOU." and "NEW OUTFIT. UNDER 60 MINUTES." via `<SplitText>`
    - Two `MagneticButton` CTAs: "SHOP NOW →" (smooth-scroll to Scene05 or navigate to shop page) and "DOWNLOAD APP →" (scroll to AppSection)
    - _Requirements: 13.1, 13.2, 13.3, 13.4_

- [~] 23. Checkpoint — Verify all components render
  - Ensure all components mount without TypeScript errors or runtime exceptions
  - Ensure `siteConfig` values flow correctly through all components
  - Ask the user if questions arise before proceeding.

- [ ] 24. Implement Hero Scene
  - [~] 24.1 Write `components/hero/HeroScene.tsx`
    - Full-viewport layout composing `<ThreeViewer>` with hero copy overlay
    - Hero copy: brand tagline and "ENTER EXPERIENCE" CTA (handled by `CinematicIntro` transition)
    - Scroll-down animation: GSAP ScrollTrigger moves camera/canvas on scroll progress transitioning into Scene 01
    - Only mounts when `introState === 'hero'`
    - _Requirements: 2.1, 2.3_

- [ ] 25. Implement SEO and metadata
  - [~] 25.1 Write `app/layout.tsx` metadata export
    - Set `title`, `description`, Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`)
    - Include font loading and root `<html>` / `<body>` structure with providers
    - _Requirements: 21.1, 21.2, 21.3_

  - [~] 25.2 Inject JSON-LD structured data in `app/page.tsx`
    - `<script type="application/ld+json">` with `@type: Organization`, name, url, logo
    - _Requirements: 21.4_

  - [~] 25.3 Write `app/sitemap.ts` and `app/robots.ts`
    - `sitemap.ts`: Next.js `MetadataRoute.Sitemap` generator covering all public routes
    - `robots.ts`: permit crawling of all public routes
    - _Requirements: 21.5, 21.7_

  - [~] 25.4 Add semantic HTML5 landmark elements throughout all page components
    - Wrap scenes in `<section>`, nav in `<nav>`, footer in `<footer>`, page in `<main>`
    - _Requirements: 21.6_

- [ ] 26. Implement performance optimisations
  - [~] 26.1 Apply `next/dynamic` code splitting to heavy components
    - Dynamically import `ThreeViewer`, `DeliveryJourney`, `DeliveryTracker`, `Scene03Vibe`, `Scene04AIStylist`, `Scene05Shop`, `AppSection` with `ssr: false` where appropriate
    - _Requirements: 20.3_

  - [~] 26.2 Optimise all images with `next/image`
    - Replace all `<img>` tags with `<Image>` from `next/image`; add `sizes` prop and WebP sources
    - _Requirements: 20.2, 20.4_

  - [~] 26.3 Configure Next.js cache headers in `next.config.js`
    - Add `Cache-Control: public, max-age=31536000, immutable` headers for `/models/`, `/images/`, `/audio/`
    - _Requirements: 20.7_

  - [~] 26.4 Add `useGLTF.preload()` and Suspense boundaries for 3D assets
    - Call `useGLTF.preload(modelPath)` in `HeroScene` when `modelPath` is available
    - Ensure all GLB loads are guarded by `<Suspense>` with skeleton fallback
    - _Requirements: 20.2, 20.6_

- [ ] 27. Implement responsive and mobile optimisations
  - [~] 27.1 Apply mobile-specific overrides across all scene components
    - Reduce `Canvas` DPR and disable shadows on mobile via `useMobileDetect`
    - Replace complex GSAP scroll-scrub with Framer Motion `whileInView` fade-ins on mobile
    - Ensure all touch targets ≥ 44×44px
    - _Requirements: 19.2, 19.3, 19.4, 19.5_

  - [~] 27.2 Verify layout at all required breakpoints
    - Manually verify (or write layout snapshot tests) at 320px, 768px, 1024px, 1440px
    - Ensure horizontal scroll in Scene 05 falls back to native touch swipe on mobile
    - _Requirements: 19.1, 19.4_

- [ ] 28. Implement accessibility hardening
  - [~] 28.1 Audit and add ARIA attributes across all interactive components
    - Add `aria-label` to all icon-only buttons (SEARCH, CART, ACCOUNT, social icons, sound toggle)
    - Add `aria-label="3D interactive fashion model display"` to `ThreeViewer` canvas wrapper
    - Ensure all non-decorative images have descriptive `alt` text
    - Verify `focus-visible` ring styles are applied consistently via Tailwind
    - _Requirements: 23.1, 23.2, 23.3, 23.4_

  - [~] 28.2 Implement reduced-motion support across all animated components
    - Use `useReducedMotion` hook to disable GSAP animations and replace with instant opacity transitions
    - _Requirements: 17.1 (implied), 23 (general)_

  - [~] 28.3 Verify colour contrast compliance
    - Audit primary, accent, and any coloured text against WCAG 2.1 AA ratios (4.5:1 body, 3:1 large)
    - Fix any failing combinations in Tailwind config or component styles
    - _Requirements: 19.6, 23.6_

- [ ] 29. Assemble home page
  - [~] 29.1 Write `app/page.tsx` — compose all scenes
    - Wrap in `<AppProvider>` and `<ScrollProvider>`
    - Render in order: `LoadingSequence` → `CinematicIntro` → `Nav` → `HeroScene` → `Scene01Problem` → `Scene02Solution` → `Scene03Vibe` → `Scene04AIStylist` → `Scene05Shop` → `DeliveryJourney` → `DeliveryTracker` → `AppSection` → `SocialProof` → `BrandPartners` → `FinalScene` → `Footer`
    - Render `CustomCursor` and `SoundController` as global overlays
    - Inject JSON-LD `<script>` block
    - Gate `HeroScene` and scroll story on `introState === 'hero'`
    - _Requirements: 1.1–1.5, 2.1–2.7, 3.1–3.3, 4.1–4.3, 5.1–5.5, 6.1–6.5, 7.1–7.5, 8.1–8.5, 9.1–9.4, 10.1–10.5, 11.1–11.4, 12.1–12.4, 13.1–13.4, 14.1–14.6, 15.1–15.5_

- [~] 30. Final checkpoint — Ensure all tests pass
  - Run `next build` and verify zero TypeScript errors and zero ESLint errors
  - Run the test suite; confirm all unit tests pass
  - Verify `next build` output reports initial JS bundle within the 200KB gzipped budget
  - Ask the user if questions arise before considering the implementation complete.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Every task references specific requirement IDs for traceability
- The design has no Correctness Properties section, so property-based tests are not applicable — unit tests cover edge cases instead
- Checkpoints (tasks 23 and 30) ensure incremental validation at natural breaks
- All configurable values (links, stats, reviews) must flow through `siteConfig` — never hardcoded in components
- GLB model files should be placed in `public/models/` before running the dev server; the viewer handles absence gracefully via `ModelFallback`
- Three.js components use `next/dynamic` with `ssr: false` to prevent server-side rendering errors

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1", "3.1"] },
    { "id": 1, "tasks": ["2.2", "4.1", "4.2", "4.3", "4.4", "5.1", "5.2", "5.3"] },
    { "id": 2, "tasks": ["6.1", "6.2", "6.3", "6.4", "6.5"] },
    { "id": 3, "tasks": ["7.1", "7.2"] },
    { "id": 4, "tasks": ["8.1", "8.2", "8.3", "8.4", "8.5", "8.6"] },
    { "id": 5, "tasks": ["9.1", "9.2", "9.3", "9.4"] },
    { "id": 6, "tasks": ["9.5", "10.1", "10.2"] },
    { "id": 7, "tasks": ["9.6", "11.1", "11.2", "11.3"] },
    { "id": 8, "tasks": ["12.1", "13.1", "14.1", "15.1", "16.1", "17.1", "18.1", "19.1", "20.1", "21.1"] },
    { "id": 9, "tasks": ["24.1"] },
    { "id": 10, "tasks": ["25.1", "25.2", "25.3", "25.4"] },
    { "id": 11, "tasks": ["26.1", "26.2", "26.3", "26.4"] },
    { "id": 12, "tasks": ["27.1", "27.2", "28.1", "28.2", "28.3"] },
    { "id": 13, "tasks": ["29.1"] }
  ]
}
```
