# Technical Design Document — Kya Pehnu? Website

## Overview

This document describes the technical architecture for the Kya Pehnu? immersive fashion-tech website. The website is a cinematic, scroll-driven single-page experience built on Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, Three.js / React Three Fiber, GSAP + ScrollTrigger, Framer Motion, and Lenis smooth scroll.

---

## 1. Project Structure

```
kya-pehnu/
├── app/
│   ├── layout.tsx              # Root layout: metadata, fonts, providers
│   ├── page.tsx                # Home page — assembles all scenes
│   ├── sitemap.ts              # Next.js sitemap generator
│   └── robots.ts               # Next.js robots.txt generator
├── components/
│   ├── layout/
│   │   ├── Nav.tsx             # Floating navigation bar
│   │   ├── MobileMenu.tsx      # Full-screen cinematic mobile menu
│   │   └── Footer.tsx          # Minimal luxury footer
│   ├── intro/
│   │   ├── LoadingSequence.tsx # Asset loading progress screen
│   │   └── CinematicIntro.tsx  # Animated opening text sequence
│   ├── hero/
│   │   └── HeroScene.tsx       # 3D viewer + hero copy overlay
│   ├── three/
│   │   ├── ThreeViewer.tsx     # Core React Three Fiber canvas
│   │   ├── FashionModel.tsx    # GLB model loader with fallback
│   │   ├── SceneLighting.tsx   # Cinematic three-point lighting rig
│   │   └── ModelFallback.tsx   # Animated silhouette / static fallback
│   ├── scenes/
│   │   ├── Scene01Problem.tsx
│   │   ├── Scene02Solution.tsx
│   │   ├── Scene03Vibe.tsx
│   │   ├── Scene04AIStylist.tsx
│   │   ├── Scene05Shop.tsx
│   │   ├── DeliveryJourney.tsx
│   │   ├── DeliveryTracker.tsx
│   │   ├── AppSection.tsx
│   │   ├── SocialProof.tsx
│   │   ├── BrandPartners.tsx
│   │   └── FinalScene.tsx
│   ├── ui/
│   │   ├── MagneticButton.tsx  # Cursor-attracted CTA button
│   │   ├── CustomCursor.tsx    # Branded desktop cursor
│   │   ├── SoundController.tsx # SOUND ON/OFF toggle
│   │   ├── SplitText.tsx       # GSAP-powered split text animation
│   │   ├── ImageReveal.tsx     # Clip-path scroll reveal for images
│   │   └── DeliveryMap.tsx     # Abstract SVG animated delivery map
│   └── providers/
│       ├── ScrollProvider.tsx  # Lenis smooth scroll context
│       └── AppProvider.tsx     # Global state (sound, cursor, intro)
├── hooks/
│   ├── useScrollProgress.ts    # ScrollTrigger scroll progress hook
│   ├── useMousePosition.ts     # Normalised mouse position hook
│   ├── useMobileDetect.ts      # Touch/mobile detection hook
│   ├── useWebGLSupport.ts      # WebGL capability detection
│   └── useReducedMotion.ts     # prefers-reduced-motion hook
├── lib/
│   ├── gsap.ts                 # GSAP + ScrollTrigger registration
│   ├── lenis.ts                # Lenis initialisation helper
│   └── three-utils.ts          # Three.js helpers (lighting, materials)
├── data/
│   ├── products.ts             # Placeholder product catalog data
│   ├── vibes.ts                # Vibe category definitions + palettes
│   ├── outfits.ts              # AI Stylist outfit recommendation map
│   └── appScreens.ts           # App section screen definitions
├── config/
│   └── site.config.ts          # Single configuration file (all external values)
├── public/
│   ├── models/                 # Drop .glb/.gltf files here
│   ├── images/                 # WebP fashion editorial images
│   ├── audio/                  # Ambient audio file
│   └── fonts/                  # Custom typeface files
├── styles/
│   └── globals.css             # Tailwind base + custom CSS (cursor hide, scroll)
└── types/
    └── index.ts                # Shared TypeScript type definitions
```

---

## 2. Configuration File Schema

**`config/site.config.ts`**

```typescript
export const siteConfig = {
  // App download links — set to empty string to show "COMING SOON" button
  IOS_APP_DOWNLOAD_LINK: '',       // e.g. 'https://apps.apple.com/...'
  ANDROID_APP_DOWNLOAD_LINK: '',   // e.g. 'https://play.google.com/...'

  // Partner / contact URLs
  PARTNER_CONTACT_URL: '/contact', // URL for "BECOME A PARTNER →" CTA

  // Social media
  SOCIAL_INSTAGRAM: 'https://instagram.com/kyapehnu',
  SOCIAL_LINKEDIN: 'https://linkedin.com/company/kyapehnu',
  SOCIAL_X: 'https://x.com/kyapehnu',

  // Delivery tracker
  DELIVERY_ETA_START_MINUTES: 45, // Starting ETA value in minutes

  // Social proof statistics
  STATS_DELIVERIES_COUNT: '50,000+',
  STATS_SATISFACTION_RATE: '98%',
  STATS_AVERAGE_RATING: 4.9,

  // Customer reviews (array of 3+)
  REVIEWS: [
    { name: 'Priya S.', rating: 5, text: 'Outfit arrived in 38 minutes. Wore it to the party the same night.' },
    { name: 'Rohan M.', rating: 5, text: 'The AI Stylist nailed my vibe. Zero effort, full look.' },
    { name: 'Anika T.', rating: 5, text: 'Finally, fashion that keeps up with my last-minute plans.' },
  ],

  // Footer links
  FOOTER_LINKS: {
    explore: {
      shop: '/#shop',
      trending: '/#shop',
      aiStylist: '/#stylist',
      howItWorks: '/#how-it-works',
    },
    company: {
      about: '/about',
      careers: '/careers',
      partners: '/partners',
      contact: '/contact',
    },
    help: {
      faqs: '/faqs',
      delivery: '/delivery',
      returns: '/returns',
      support: '/support',
    },
  },
} as const;
```

---

## 3. State Management

Global UI state is managed via React Context (no external state library needed given the limited scope).

**`AppProvider`** holds:
- `introComplete: boolean` — whether the cinematic intro has been dismissed
- `soundEnabled: boolean` — SOUND ON/OFF toggle state
- `activeVibe: VibeCategory` — currently selected vibe in Scene 03
- `isMobile: boolean` — derived from `useMobileDetect`

**`ScrollProvider`** holds:
- Lenis instance reference — exposed to all components via context
- `scrollY: number` — current scroll position (updated via RAF)

Scene-local state (AI Stylist selections, Shop active category) is managed with `useState` inside each scene component.

---

## 4. Scroll Architecture

### 4.1 Lenis Smooth Scroll

Lenis is initialised once in `ScrollProvider` and integrated with GSAP's ticker:

```typescript
// lib/lenis.ts
import Lenis from '@studio-freight/lenis';
import { gsap } from 'gsap';

export function createLenis() {
  const lenis = new Lenis({ lerp: 0.1, smoothWheel: true });
  gsap.ticker.add((time) => lenis.raf(time * 1000));
  gsap.ticker.lagSmoothing(0);
  return lenis;
}
```

On mobile (viewport < 768px), Lenis is instantiated with `smoothWheel: false` to use native scroll for performance, while GSAP ScrollTrigger uses `scroller: window`.

### 4.2 GSAP ScrollTrigger

All scroll-triggered animations use `ScrollTrigger.create()` with `scrub: true` for scroll-linked animations and `scrub: false` for one-shot entry animations.

Each Scene component registers its ScrollTrigger instances in a `useLayoutEffect` and cleans them up on unmount via `ScrollTrigger.getAll().forEach(t => t.kill())` scoped to a React ref context using `gsap.context()`.

### 4.3 Scene 02 Countdown Scroll Sync

The 60-minute countdown in Scene 02 uses a ScrollTrigger with `scrub: 1` mapped to a GSAP timeline driving a counter from 3600 (60:00) to 0 (00:00). Pausing/resuming is handled automatically by ScrollTrigger's scrub behaviour — scroll position is the source of truth.

---

## 5. 3D Architecture

### 5.1 ThreeViewer Component

```
ThreeViewer
├── Canvas (React Three Fiber)        # WebGL context
│   ├── SceneLighting                 # Key, fill, rim lights
│   ├── FashionModel                  # Suspense-wrapped GLB loader
│   │   └── ModelFallback             # Shown while loading
│   └── OrbitControls (disabled)      # Mouse rotation via custom hook
└── WebGLFallback (static image)      # Shown when WebGL unavailable
```

### 5.2 GLB Model Auto-Detection

The `FashionModel` component uses Next.js `getStaticProps` (or a server component API route) to list files in `/public/models/` at build time and picks the first `.glb` file found. The path is passed as a prop. If no file is found, `ModelFallback` renders instead.

```typescript
// Server-side model detection (Next.js)
export async function getModelPath(): Promise<string | null> {
  const fs = await import('fs/promises');
  const path = await import('path');
  const dir = path.join(process.cwd(), 'public', 'models');
  try {
    const files = await fs.readdir(dir);
    const glb = files.find(f => f.endsWith('.glb') || f.endsWith('.gltf'));
    return glb ? `/models/${glb}` : null;
  } catch {
    return null;
  }
}
```

### 5.3 Mouse-Follow Rotation

`useMousePosition` returns normalised `{ x, y }` values in `[-1, 1]`. In the Three.js render loop, the model's rotation is interpolated toward `targetRotation` using `THREE.MathUtils.lerp` with a factor of 0.05 per frame, clamped to ±15° (0.26 radians).

### 5.4 Mobile Performance Degradation

`useMobileDetect` drives two flags:
- `pixelRatio: Math.min(window.devicePixelRatio, 1.5)` — capped on mobile
- `shadowMapEnabled: false` — disabled on mobile

These are passed as props to `Canvas`:
```tsx
<Canvas dpr={pixelRatio} shadows={shadowMapEnabled} />
```

---

## 6. Cinematic Intro State Machine

States: `loading → intro → hero`

```
loading:  LoadingSequence visible, assets fetching
          → auto-transition to `intro` when assets ready OR 5s timeout
intro:    CinematicIntro visible, text cards sequencing via GSAP timeline
          → transition to `hero` on "ENTER EXPERIENCE →" click (800ms wipe)
hero:     CinematicIntro unmounted, HeroScene mounted, Nav visible
```

State is held in `AppProvider.introComplete`. The `LoadingSequence` uses a fake progress counter (RAF-driven, capped at 90%) that jumps to 100% once assets actually resolve.

---

## 7. AI Stylist — Client-Side Logic

No backend is required. Outfit recommendations are driven by a static lookup table in `data/outfits.ts`:

```typescript
type OutfitKey = `${Destination}_${StylePreference}`;

export const outfitMap: Record<OutfitKey, Outfit> = {
  'date_minimal': { products: [...], totalPrice: 3499, deliveryEta: '38 min' },
  'party_bold':   { products: [...], totalPrice: 4999, deliveryEta: '42 min' },
  // ... all combinations
};
```

`Scene04AIStylist` derives the key from user selections and reads from `outfitMap`. Product cards are revealed with a GSAP stagger of 100ms using `gsap.from('.product-card', { opacity: 0, y: 30, stagger: 0.1 })`.

---

## 8. Delivery Journey Animation

`DeliveryJourney` uses a single ScrollTrigger pinning the section for `500vh` of scroll travel. The pinned section contains five steps, each occupying 20% of the total scroll range (100vh equivalent).

ScrollTrigger progress `[0, 1]` maps to step index `0–4` via `Math.floor(progress * 5)`.

The `DeliveryMap` SVG animates a circle element along a `<path>` using `stroke-dashoffset` driven by the RIDE step's local progress (steps 3/4 range = `[0.6, 0.8]` of total).

---

## 9. Magnetic Button and Custom Cursor

### MagneticButton

```typescript
// Uses useRef for the button element and GSAP quickSetter for performance
const setX = gsap.quickSetter(ref.current, 'x', 'px');
const setY = gsap.quickSetter(ref.current, 'y', 'px');

onMouseMove: compute offset from button centre, clamp to 12px, call setX/setY
onMouseLeave: gsap.to(ref.current, { x: 0, y: 0, ease: 'elastic.out(1, 0.3)', duration: 0.7 })
```

Touch devices: `useMobileDetect` disables all handlers, no DOM event listeners attached.

### CustomCursor

A `div` with `position: fixed`, `pointer-events: none`, `z-index: 9999`, tracking `mousemove` via `gsap.quickSetter`. Two concentric circles (dot + ring) with different lerp speeds for a trailing effect. Hidden on `touch` devices and when viewport < 1024px.

---

## 10. Sound Controller

```typescript
// useSound hook
const audioRef = useRef<HTMLAudioElement | null>(null);

useEffect(() => {
  audioRef.current = new Audio('/audio/ambient.mp3');
  audioRef.current.loop = true;
  audioRef.current.volume = 0.3; // 30% of system volume
}, []);

// Page visibility API for auto-pause/resume
document.addEventListener('visibilitychange', () => {
  if (document.hidden) audioRef.current?.pause();
  else if (soundEnabled) audioRef.current?.play();
});
```

An `aria-live="polite"` region announces state changes: "Sound enabled" / "Sound disabled".

---

## 11. Scroll and Text Animations

### SplitText

Uses GSAP's `SplitText` plugin to split headlines into individual characters or words. Each character is wrapped in a `span` and animated with `gsap.from` on ScrollTrigger entry.

For mobile (< 768px), `SplitText` is replaced with a simple `opacity: 0 → 1` Framer Motion `whileInView` transition.

### ImageReveal

Uses CSS `clip-path: inset(100% 0 0 0)` → `inset(0% 0 0 0)` driven by `gsap.to` on ScrollTrigger entry with `ease: 'power4.inOut'`.

### Horizontal Scroll (Scene 05)

The catalog track is translated on the X axis via ScrollTrigger with `pin: true` on the section and `end: () => '+=' + trackWidth`:

```typescript
gsap.to(trackRef.current, {
  x: () => -(trackRef.current.scrollWidth - window.innerWidth),
  ease: 'none',
  scrollTrigger: { trigger: sectionRef.current, pin: true, scrub: 1, end: '+=3000' }
});
```

On mobile, the track uses native `overflow-x: scroll` with `scroll-snap-type: x mandatory`.

---

## 12. App Section — Phone Mockup

The smartphone is rendered as a CSS 3D perspective card (no Three.js needed here) with `transform: perspective(1000px) rotateY(-15deg) rotateX(5deg)`. It subtly parallaxes on scroll via GSAP ScrollTrigger.

App screens cycle using a Framer Motion `AnimatePresence` with `variants` for slide-in/slide-out. Auto-cycling interval: 2500ms, paused when section is out of viewport using `useInView`.

---

## 13. SEO and Metadata

**`app/layout.tsx`** exports Next.js `Metadata`:

```typescript
export const metadata: Metadata = {
  title: 'Kya Pehnu? — New Outfit Under 60 Minutes',
  description: 'Discover your next look and get your new outfit delivered in under 60 minutes with Kya Pehnu?',
  openGraph: {
    title: 'Kya Pehnu? — New Outfit Under 60 Minutes',
    description: '...',
    url: 'https://kyapehnu.com',
    images: [{ url: '/og-image.jpg', width: 1200, height: 630 }],
  },
};
```

**JSON-LD** is injected via a `<script type="application/ld+json">` in `app/page.tsx`:

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Kya Pehnu?",
  "url": "https://kyapehnu.com",
  "logo": "https://kyapehnu.com/logo.png"
}
```

**`app/sitemap.ts`** and **`app/robots.ts`** use Next.js built-in generators.

---

## 14. Performance Strategy

| Strategy | Implementation |
|---|---|
| Code splitting | `next/dynamic` for Three.js viewer, scene components |
| Image optimisation | `next/image` with WebP, `sizes` prop, lazy loading |
| 3D asset loading | `useGLTF.preload()` from Drei, Suspense boundary |
| Bundle size | Dynamic imports keep initial bundle < 200KB gzipped |
| Caching | `next.config.js` headers: `Cache-Control: public, max-age=31536000, immutable` for `/models/`, `/images/`, `/audio/` |
| WebGL optimisation | `frameloop="demand"` on Canvas (renders only on change), reduced DPR on mobile |

Three.js canvas uses `frameloop="demand"` from React Three Fiber — the render loop only fires when scene state changes (mouse move, vibe selection, scroll), not at 60fps continuously.

---

## 15. Accessibility Patterns

- All icon-only buttons: `aria-label` attribute
- Canvas: `aria-label="3D interactive fashion model display"`
- Sound toggle: `aria-live="polite"` region for state announcements
- Keyboard nav: `focus-visible` styles via Tailwind `focus-visible:ring-2`
- Tab order: logical DOM order, no `tabindex` > 0 used
- Reduced motion: `useReducedMotion` hook disables GSAP animations and replaces with instant `opacity` transitions when `prefers-reduced-motion: reduce` is detected
- Colour contrast: black/white primary palette inherently meets WCAG AA; accent colours validated in Tailwind config

---

## 16. Responsive Breakpoints

| Breakpoint | Behaviour |
|---|---|
| < 768px (mobile) | Native scroll, simplified animations, reduced WebGL DPR, touch swipe |
| 768px–1023px (tablet) | Lenis active, partial animations, hidden desktop cursor |
| ≥ 1024px (desktop) | Full experience: custom cursor, magnetic buttons, full GSAP |
| ≥ 1440px (large desktop) | Maximum typography scale, full parallax depth |

---

## 17. Key Dependencies

```json
{
  "next": "14.x",
  "react": "18.x",
  "typescript": "5.x",
  "tailwindcss": "3.x",
  "three": "^0.165.0",
  "@react-three/fiber": "^8.x",
  "@react-three/drei": "^9.x",
  "gsap": "^3.12.x",
  "@studio-freight/lenis": "^1.x",
  "framer-motion": "^11.x",
  "clsx": "^2.x"
}
```
