# Requirements Document

## Introduction

Kya Pehnu? is a fashion-tech company that enables users to discover, select, and receive a new outfit delivered to their doorstep in under 60 minutes. This document specifies the requirements for a cinematic, immersive, premium website that communicates the brand's core value proposition — speed, style, and AI-powered personalisation — through editorial-quality visual storytelling, 3D fashion presentation, scroll-triggered cinematics, and a luxury-grade user experience. The website is built on Next.js, React, TypeScript, Tailwind CSS, Three.js / React Three Fiber, and GSAP / Framer Motion.

---

## Glossary

- **Website**: The Kya Pehnu? web application served at the root domain.
- **Cinematic_Intro**: The full-screen animated opening sequence before the main experience.
- **Hero_Scene**: The full-screen 3D interactive section rendered after the user enters the experience.
- **Scroll_Story**: The sequence of themed sections revealed as the user scrolls through the page.
- **Scene**: An individual named section within the Scroll_Story.
- **3D_Viewer**: The WebGL-powered component that renders GLTF/GLB fashion model assets.
- **AI_Stylist**: The interactive recommendation interface that suggests a complete outfit based on user-selected destination and style preferences.
- **Delivery_Tracker**: The animated mock interface showing the real-time delivery journey.
- **App_Section**: The section showcasing the Kya Pehnu? mobile application with download links.
- **Config_File**: A single configuration file that holds all externally configurable values such as download links and placeholder statistics.
- **Magnetic_Button**: A button that shifts its position toward the cursor when the cursor is within a defined proximity radius.
- **Custom_Cursor**: A branded cursor element that replaces the default OS cursor on desktop.
- **Loading_Sequence**: The animated screen shown while the initial page assets are being loaded.
- **Sound_Controller**: The UI toggle that enables or disables ambient audio.
- **Nav**: The floating navigation bar persistent across all sections.
- **Footer**: The minimal luxury footer at the bottom of the page.
- **Partner_Logo**: A placeholder visual representing a brand partner.
- **Vibe_Selector**: The interactive category picker in Scene 03 (DATE NIGHT, COLLEGE, PARTY, OFFICE, STREET, CASUAL, WEEKEND).
- **Delivery_Map**: The animated map visual depicting the delivery route from store to user.
- **GLB_Model**: A 3D asset in GLTF binary format placed in `/public/models/` and consumed by the 3D_Viewer.

---

## Requirements

---

### Requirement 1: Cinematic Intro Sequence

**User Story:** As a first-time visitor, I want to see a cinematic opening sequence, so that I immediately understand the brand's premium identity and core value proposition before entering the main experience.

#### Acceptance Criteria

1. WHEN the Website is first loaded and all critical assets are ready, THE Cinematic_Intro SHALL display a sequence of full-screen text cards in this exact order: "KYA PEHNU?" → "WHAT ARE YOU WEARING TONIGHT?" → "DON'T KNOW?" → "WE'LL FIX THAT." → "NEW OUTFIT. UNDER 60 MINUTES." → a CTA button labelled "ENTER EXPERIENCE →".
2. WHILE the Cinematic_Intro is active, THE Website SHALL render the background as solid black with white typographic content only.
3. WHEN the user clicks "ENTER EXPERIENCE →", THE Cinematic_Intro SHALL transition out and THE Hero_Scene SHALL become visible using a cinematic fade or wipe animation lasting no more than 800ms.
4. IF the Cinematic_Intro assets fail to load within 5 seconds, THEN THE Website SHALL skip directly to the Hero_Scene and log the failure to the browser console.
5. THE Loading_Sequence SHALL display before the Cinematic_Intro begins, showing a progress indicator until initial assets reach 100% load completion. IF the Hero_Scene becomes visible through any other means (such as an asset-load failure) while the loading progress has not yet reached 100%, THE Website SHALL allow the Hero_Scene to display without waiting for the Loading_Sequence to complete.

---

### Requirement 2: Hero 3D Fashion Experience

**User Story:** As a visitor who has entered the experience, I want to see a high-quality 3D fashion model that responds to my mouse and scroll, so that I feel immersed in the brand's premium aesthetic.

#### Acceptance Criteria

1. WHEN the Hero_Scene is active, THE 3D_Viewer SHALL render the GLB_Model at full viewport width and height with cinematic three-point lighting.
2. WHEN the user moves the mouse across the Hero_Scene, THE 3D_Viewer SHALL apply a subtle mouse-follow rotation to the GLB_Model within a maximum rotation offset of ±15 degrees on the X and Y axes.
3. WHEN the user scrolls down from the Hero_Scene, THE 3D_Viewer SHALL animate the camera position in response to scroll progress, transitioning smoothly into Scene 01 of the Scroll_Story.
4. THE 3D_Viewer SHALL detect and load any conformant GLB_Model file placed at `/public/models/` without requiring code changes to the viewer component; the requirement is satisfied when the file is detected and a load attempt is initiated.
5. IF a GLB_Model file is absent from `/public/models/`, THEN THE 3D_Viewer SHALL render a branded fallback visual (e.g., animated logo or silhouette) instead of an empty viewport.
6. WHERE the user's device does not support WebGL, THE 3D_Viewer SHALL display a high-resolution static image fallback and continue the rest of the page experience without interruption.
7. THE 3D_Viewer SHALL maintain a target frame rate of 60fps on desktop and gracefully reduce render quality to maintain 30fps on mobile devices.

---

### Requirement 3: Cinematic Scroll Story — Scene 01: The Problem

**User Story:** As a visitor scrolling through the site, I want to feel the brand's problem statement viscerally, so that I immediately relate to the core need Kya Pehnu? solves.

#### Acceptance Criteria

1. WHEN the user scrolls into Scene 01, THE Scroll_Story SHALL reveal the headline "PLANS CHANGED? OUTFIT DID TOO." using a scroll-triggered split-text animation.
2. THE Scroll_Story SHALL use scroll progress to control the opacity and vertical translate of Scene 01 content, producing a parallax depth effect.
3. WHILE Scene 01 is fully in the viewport, THE Website SHALL enforce 100% opacity and zero transform offset on Scene 01 content immediately upon full-viewport detection, without gradual transition.

---

### Requirement 4: Cinematic Scroll Story — Scene 02: The Solution

**User Story:** As a visitor, I want to see the brand's speed promise communicated visually, so that the "under 60 minutes" claim feels credible and impactful.

#### Acceptance Criteria

1. WHEN the user scrolls into Scene 02, THE Scroll_Story SHALL reveal the headline "SO WE MADE IT FAST. REALLY FAST. UNDER 60 MINUTES." using a scroll-triggered animation.
2. WHEN Scene 02 enters the viewport, THE Scroll_Story SHALL trigger an animated countdown or delivery-timeline graphic that progresses from 60:00 to 00:00 in synchronisation with scroll progress. WHEN the user scrolls away from Scene 02 before the countdown completes, THE Scroll_Story SHALL pause the countdown at its current position and resume from that same position when the user returns to Scene 02.
3. THE Scroll_Story SHALL complete the countdown animation before the user exits Scene 02.

---

### Requirement 5: Cinematic Scroll Story — Scene 03: Vibe Selector

**User Story:** As a visitor, I want to choose a style vibe and see the 3D model change outfit accordingly, so that I can personalise my experience and discover relevant products.

#### Acceptance Criteria

1. WHEN the user scrolls into Scene 03, THE Vibe_Selector SHALL display the following interactive categories: DATE NIGHT, COLLEGE, PARTY, OFFICE, STREET, CASUAL, WEEKEND.
2. WHEN the user selects a category in the Vibe_Selector, THE 3D_Viewer SHALL transition to a GLB_Model or material variant corresponding to that category within 600ms.
3. WHEN the user selects a category in the Vibe_Selector, THE Scroll_Story SHALL update the background colour or overlay of Scene 03 to match a predefined palette for that category.
4. WHEN the user selects a category in the Vibe_Selector, THE Scroll_Story SHALL display a curated set of recommended product cards for that category below the 3D_Viewer.
5. THE Vibe_Selector SHALL highlight the currently active category with a distinct visual treatment (e.g., underline, colour inversion, or animated indicator).

---

### Requirement 6: Cinematic Scroll Story — Scene 04: AI Stylist

**User Story:** As a visitor who doesn't know what to wear, I want an AI Stylist to generate a complete outfit recommendation based on my destination and style preference, so that I can shop a curated look instantly.

#### Acceptance Criteria

1. WHEN the user scrolls into Scene 04, THE AI_Stylist SHALL display the prompt "DON'T KNOW WHAT TO WEAR?" followed by input selectors for destination and style preference.
2. WHEN the user completes both selectors and triggers the recommendation, THE AI_Stylist SHALL display a complete outfit recommendation including: a 3D model view, individual product cards, total price, and estimated delivery time within 60 minutes.
3. WHEN the AI_Stylist displays the recommendation, THE Scroll_Story SHALL animate the reveal of each product card sequentially with a stagger duration of 100ms per card.
4. THE AI_Stylist SHALL display the headline "YOUR LOOK IS READY." upon successful recommendation generation.
5. IF the user has not selected both a destination and a style preference, THEN THE AI_Stylist SHALL display an inline validation message and SHALL NOT trigger a recommendation. WHEN both selections are present and a recommendation is successfully generated, THE AI_Stylist SHALL hide any previously displayed validation messages.

---

### Requirement 7: Cinematic Scroll Story — Scene 05: Shopping Experience

**User Story:** As a visitor ready to shop, I want to browse a premium editorial product catalog, so that I can discover and select items for fast delivery.

#### Acceptance Criteria

1. WHEN the user scrolls into Scene 05, THE Scroll_Story SHALL display a horizontally scrollable editorial catalog with the following category tabs: MEN, WOMEN, STREETWEAR, PARTY, CASUAL, FOOTWEAR, ACCESSORIES.
2. WHEN the user hovers over a product card in Scene 05 on desktop, THE Scroll_Story SHALL display a product preview overlay showing product name, price, and a "ADD TO CART" action.
3. THE Scroll_Story SHALL display the label "⚡ ARRIVES IN UNDER 60 MIN" on every product card in Scene 05.
4. WHEN the user selects a category tab in Scene 05, THE Scroll_Story SHALL filter and re-render the product catalog to show only items belonging to the selected category with a smooth transition lasting no more than 400ms.
5. THE Scroll_Story SHALL support touch-based horizontal swipe gestures for the catalog on mobile devices.

---

### Requirement 8: 60-Minute Delivery Journey Animation

**User Story:** As a visitor, I want to see the delivery journey visualised step-by-step, so that I trust the 60-minute promise is real and systematic.

#### Acceptance Criteria

1. WHEN the user scrolls into the delivery journey section, THE Website SHALL animate through five sequential steps in strict scroll-based progression order — ORDER → PICK → PACK → RIDE → YOUR DOOR — where each step activates only within its designated equal scroll range and no step may activate outside its range.
2. THE Website SHALL synchronise each step transition with scroll progress so that each step activates at equal scroll increments.
3. WHEN the RIDE step is active, THE Delivery_Map SHALL display an animated delivery indicator moving along a route from a store pin to a destination pin.
4. THE Delivery_Map SHALL render as an abstract stylised map (not a third-party map embed) to avoid external API dependency.
5. WHILE a delivery journey step is active, THE Website SHALL display a descriptive label and icon for that step at full opacity.

---

### Requirement 9: Real-Time Delivery Tracker Mock

**User Story:** As a visitor, I want to see a mock real-time delivery tracker, so that I understand what the post-order experience looks like.

#### Acceptance Criteria

1. WHEN the Delivery_Tracker section is visible, THE Delivery_Tracker SHALL display five status milestones in order: "Order confirmed ✓", "Picked from store ✓", "Packed ✓", "Out for delivery →", "Arriving soon".
2. THE Delivery_Tracker SHALL animate through the milestones sequentially using a looping demo cycle with each milestone visible for no less than 1500ms.
3. THE Delivery_Tracker SHALL display a dynamic ETA countdown that decrements from a configurable starting value defined in the Config_File.
4. THE Delivery_Tracker SHALL clearly label itself as a demonstration/mock experience. THE Delivery_Tracker SHALL continue to operate and display all milestones regardless of whether the demo label renders successfully.

---

### Requirement 10: App Section

**User Story:** As a visitor interested in the mobile experience, I want to see a premium presentation of the Kya Pehnu? app, so that I feel confident downloading it.

#### Acceptance Criteria

1. WHEN the user scrolls into the App_Section, THE Website SHALL render a 3D or CSS-perspective smartphone mockup displaying the Kya Pehnu? mobile app UI.
2. THE App_Section SHALL cycle through at least six app screens in sequence: Home, Discover, AI Stylist, Product, Cart, Order Tracking.
3. THE App_Section SHALL display two download CTAs: one for App Store (iOS) and one for Google Play (Android).
4. THE App_Section SHALL read the iOS download URL from the Config_File key `IOS_APP_DOWNLOAD_LINK` and the Android download URL from the Config_File key `ANDROID_APP_DOWNLOAD_LINK`.
5. IF either Config_File download link is not set or is empty, THEN THE App_Section SHALL render the corresponding CTA button in a disabled state with the label "COMING SOON".

---

### Requirement 11: Social Proof Section

**User Story:** As a visitor evaluating the brand, I want to see social proof, so that I feel confident that Kya Pehnu? is trustworthy and popular.

#### Acceptance Criteria

1. THE Website SHALL display a social proof section with the headline "LOOK GOOD. WITHOUT THE WAIT."
2. THE Website SHALL display at minimum: customer review cards, an aggregate star rating, a total deliveries count, and a customer satisfaction rate percentage. THE Website SHALL display all statistics as supplied by the Config_File regardless of their values.
3. THE Website SHALL source all statistics from the Config_File so that they can be updated without code changes.
4. THE Website SHALL display at least three customer review cards with a name, rating, and review text field, each sourced from the Config_File.

---

### Requirement 12: Brand Partners Section

**User Story:** As a visitor, I want to see the brands available on Kya Pehnu?, so that I trust the product quality and variety.

#### Acceptance Criteria

1. THE Website SHALL display a brand partners section with the headline "THE BRANDS YOU LOVE. DELIVERED FASTER."
2. THE Website SHALL render Partner_Logo placeholders in a horizontal marquee or grid layout.
3. THE Website SHALL display a CTA labelled "BECOME A PARTNER →" that links to a configurable contact or partner page URL defined in the Config_File.
4. WHEN the user hovers over a Partner_Logo on desktop, THE Website SHALL apply a subtle scale or brightness animation to that logo.

---

### Requirement 13: Final Cinematic Scene

**User Story:** As a visitor who has scrolled through the full experience, I want a strong closing statement that drives conversion, so that I take action — shopping or downloading the app.

#### Acceptance Criteria

1. WHEN the user scrolls into the final cinematic scene, THE Scroll_Story SHALL reveal the headline "SO… KYA PEHNU? WE'VE GOT YOU." followed by "NEW OUTFIT. UNDER 60 MINUTES." using a scroll-triggered animation.
2. THE Scroll_Story SHALL display two CTAs in the final scene: "SHOP NOW →" and "DOWNLOAD APP →".
3. WHEN the user clicks "SHOP NOW →", THE Website SHALL either smooth-scroll to Scene 05 or navigate to the shop page; the choice between these two behaviours is left to the implementation.
4. WHEN the user clicks "DOWNLOAD APP →", THE Website SHALL scroll to the App_Section.

---

### Requirement 14: Navigation

**User Story:** As a visitor at any point in the page, I want a persistent, minimal navigation bar, so that I can access key sections without losing context.

#### Acceptance Criteria

1. THE Nav SHALL be permanently visible and floating above all page content at the top of the viewport.
2. THE Nav SHALL display the brand name "Kya Pehnu?" at the top-left, navigation links (SHOP, STYLIST, HOW IT WORKS, APP) in the centre or right, and icons for SEARCH, CART, and ACCOUNT at the far right.
3. WHEN the user clicks a Nav link, THE Website SHALL smooth-scroll to the corresponding section.
4. WHEN the user scrolls more than 80px from the top, THE Nav SHALL transition its background from transparent to a semi-transparent dark overlay within 300ms to maintain legibility.
5. WHEN the viewport width is below 768px, THE Nav SHALL hide the centre navigation links and display a menu icon that opens a full-screen cinematic menu overlay. On desktop viewports the menu icon may also be present alongside centre navigation links.
6. WHEN the full-screen menu is open on mobile, THE Nav SHALL display all navigation links in a large typographic style with a close action. On mobile viewports both the centre links (when visible in the overlay) and the menu icon may be simultaneously visible.

---

### Requirement 15: Footer

**User Story:** As a visitor at the bottom of the page, I want a comprehensive yet minimal footer, so that I can find important links and brand information.

#### Acceptance Criteria

1. THE Footer SHALL display the Kya Pehnu? logo and tagline "NEW OUTFIT. UNDER 60 MINUTES." at the top.
2. THE Footer SHALL display four link groups: Explore (Shop, Trending, AI Stylist, How It Works), Company (About, Careers, Partners, Contact), Help (FAQs, Delivery, Returns, Support), and Download (App Store, Google Play).
3. THE Footer SHALL display social media links for Instagram, LinkedIn, and X (formerly Twitter).
4. THE Footer SHALL display the copyright notice "© 2026 Kya Pehnu?" at the bottom.
5. THE Footer SHALL source all external link URLs from the Config_File so they can be updated without code changes.

---

### Requirement 16: Interaction Design — Magnetic Buttons and Custom Cursor

**User Story:** As a desktop visitor, I want premium micro-interactions on buttons and cursor, so that the experience feels high-end and intentional.

#### Acceptance Criteria

1. THE Website SHALL replace the default OS cursor with a Custom_Cursor on desktop viewports (width ≥ 1024px).
2. WHEN the user hovers over a Magnetic_Button, THE Magnetic_Button SHALL shift its position toward the cursor by a maximum of 12px in any direction.
3. WHEN the user's cursor leaves the Magnetic_Button proximity radius, THE Magnetic_Button SHALL return to its original position using a spring-easing animation.
4. THE Website SHALL designate all primary CTA buttons as Magnetic_Button instances.
5. WHERE the device is a touch device, THE Website SHALL disable Magnetic_Button behaviour and Custom_Cursor to preserve mobile usability. On desktop devices, THE Website SHALL keep Magnetic_Button behaviour active even when the Custom_Cursor is unavailable. WHEN a device switches between mouse and touch input (e.g., a convertible laptop), THE Website SHALL maintain the current Magnetic_Button state until the next full page reload.

---

### Requirement 17: Interaction Design — Scroll and Text Animations

**User Story:** As a visitor scrolling through the site, I want smooth, cinematic text and image reveals, so that every section feels intentional and editorial.

#### Acceptance Criteria

1. THE Website SHALL implement split-text animation on all headline elements within the Scroll_Story, revealing characters or words as they enter the viewport.
2. THE Website SHALL implement scroll-triggered image reveal animations using a clip-path or mask wipe technique on all editorial imagery.
3. THE Website SHALL implement smooth page-level scrolling using a scroll library (e.g., Lenis) rather than native browser scroll where supported.
4. THE Website SHALL implement horizontal scroll for catalog sections using scroll-linked animation rather than native overflow scroll on desktop.
5. WHEN a section transitions out of the viewport during forward scrolling, THE Website SHALL apply an exit animation that does not cause layout shift or content flash. THE Website SHALL apply exit animations only during forward scroll; sections re-entering the viewport from below during upward scroll SHALL use their entrance animation only.

---

### Requirement 18: Sound Design

**User Story:** As a visitor, I want optional ambient audio that enhances the cinematic experience without being intrusive, so that I can choose my preferred experience.

#### Acceptance Criteria

1. THE Sound_Controller SHALL display a "SOUND ON / SOUND OFF" toggle in a fixed position on the viewport.
2. WHEN the Website first loads, THE Sound_Controller SHALL default to the muted (SOUND OFF) state; THE Website SHALL NOT autoplay any audio.
3. WHEN the user toggles the Sound_Controller to SOUND ON, THE Website SHALL begin playing ambient audio at a maximum volume of 30% of the system volume. IF the current system volume level would result in audio output exceeding 30% of the system volume ceiling, THE Website SHALL refuse to play audio until the system volume is reduced to a level where 30% is achievable.
4. WHEN the user navigates away from the browser tab, THE Sound_Controller SHALL automatically pause audio playback.
5. WHEN the user returns to the browser tab, THE Sound_Controller SHALL resume audio playback only if the user had previously set the state to SOUND ON.

---

### Requirement 19: Responsive Design

**User Story:** As a visitor on any device, I want a premium experience suited to my screen size, so that the website feels intentional regardless of device.

#### Acceptance Criteria

1. THE Website SHALL render a fully functional and visually complete layout at viewport widths of 320px, 768px, 1024px, and 1440px. Rendering behaviour at other viewport widths is not guaranteed by this requirement.
2. WHERE the viewport width is below 768px, THE Website SHALL reduce WebGL render resolution and particle count in the 3D_Viewer to maintain performance.
3. WHERE the viewport width is below 768px, THE Website SHALL replace complex GSAP scroll-scrub animations with simplified fade-in transitions to reduce CPU usage.
4. THE Website SHALL support touch swipe gestures for horizontal scroll sections on mobile.
5. THE Website SHALL maintain minimum touch target sizes of 44×44px for all interactive elements on mobile, in compliance with WCAG 2.1 AA guidelines.
6. THE Website SHALL render all text at accessible contrast ratios meeting WCAG 2.1 AA (minimum 4.5:1 for body text, 3:1 for large text) across all themes and backgrounds.

---

### Requirement 20: Performance

**User Story:** As a visitor on any connection speed, I want the website to load and respond quickly, so that I don't abandon the experience before it starts.

#### Acceptance Criteria

1. THE Website SHALL achieve a Lighthouse Performance score of 70 or above on mobile and 85 or above on desktop when measured on a standard production build.
2. THE Website SHALL implement lazy loading for all images and 3D assets below the fold using Intersection Observer or Next.js built-in lazy loading.
3. THE Website SHALL implement route-level and component-level code splitting via Next.js dynamic imports so that the initial JavaScript bundle does not exceed 200KB gzipped.
4. THE Website SHALL serve all images in WebP format with appropriate responsive `srcset` attributes.
5. THE Website SHALL compress all GLB_Model assets to under 5MB each before serving.
6. WHEN a GLB_Model is loading, THE 3D_Viewer SHALL display a skeleton or branded progress animation to prevent a blank viewport.
7. THE Website SHALL implement HTTP cache headers for all static assets with a max-age of at least 1 year for fingerprinted assets.

---

### Requirement 21: SEO and Metadata

**User Story:** As a potential customer searching for fashion delivery services, I want the website to appear in search results with accurate previews, so that I can discover and trust the brand.

#### Acceptance Criteria

1. THE Website SHALL set the page `<title>` to "Kya Pehnu? — New Outfit Under 60 Minutes".
2. THE Website SHALL set the meta description to "Discover your next look and get your new outfit delivered in under 60 minutes with Kya Pehnu?".
3. THE Website SHALL include Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`) on all publicly accessible pages.
4. THE Website SHALL include structured data (JSON-LD) of type `Organization` on the homepage with the brand name, URL, and logo.
5. THE Website SHALL generate and serve a `sitemap.xml` covering all public routes.
6. THE Website SHALL use semantic HTML5 landmark elements (`<header>`, `<main>`, `<section>`, `<footer>`, `<nav>`) throughout all pages.
7. THE Website SHALL include a `robots.txt` file that permits crawling of all public routes.

---

### Requirement 22: Configuration File

**User Story:** As a developer or content manager, I want all externally configurable values in a single file, so that I can update links, statistics, and copy without touching component code.

#### Acceptance Criteria

1. THE Website SHALL maintain a single Config_File (e.g., `config/site.config.ts`) that exports all configurable values.
2. THE Config_File SHALL include at minimum: `IOS_APP_DOWNLOAD_LINK`, `ANDROID_APP_DOWNLOAD_LINK`, partner contact URL, social media URLs, delivery tracker ETA start value, statistics (deliveries count, satisfaction rate), and customer review content.
3. WHEN any Config_File value is changed, THE Website SHALL reflect the updated value in the production deployment resulting from the next successful build. The requirement is satisfied only when the updated value is observable in production, regardless of intermediate build or deployment failures.
4. THE Config_File SHALL include inline comments describing the purpose and expected format of each configurable key.

---

### Requirement 23: Accessibility

**User Story:** As a visitor using assistive technologies, I want the website to be navigable and understandable without relying solely on visual presentation, so that I can access the full experience.

#### Acceptance Criteria

1. THE Website SHALL provide `aria-label` or `aria-labelledby` attributes on all icon-only interactive elements (e.g., SEARCH, CART, ACCOUNT, social media icons).
2. THE Website SHALL ensure all interactive elements are reachable and operable via keyboard navigation in a logical tab order.
3. THE Website SHALL provide descriptive `alt` text for all non-decorative images.
4. THE Website SHALL provide a text alternative or `aria-label` for the 3D_Viewer canvas element describing its content.
5. WHEN the Sound_Controller state changes, THE Website SHALL announce the new state ("Sound enabled" or "Sound disabled") via an `aria-live` region.
6. THE Website SHALL not rely on colour alone to convey meaning; supplementary icons or text SHALL accompany all colour-coded states.
