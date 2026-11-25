# UI/UX & Animation

## 1. Design direction

- **Feel:** premium industrial — engineered, trustworthy, clean. Big product imagery, confident type, generous space, purposeful motion. Must not look like a template.
- **Brand:** keep the Vectus logo and colours, refined into a design-token palette (primary, secondary, neutrals, success/warning, dark sections). Category accents (tanks, bathware, garden, road safety) as secondary tokens.
- **Type:** one display face + one text face, self-hosted via `next/font`, fluid sizes with `clamp()`.
- **Grid:** 12 columns, max width 1440 px, 16 px gutters on mobile.
- **Components:** product card, buy box, variant pills, compare tray, cart drawer, filter drawer, mega menu, forms, accordions, tabs, toasts, modals, breadcrumbs, pagination, skeletons.
- **Accessibility:** WCAG 2.1 AA — contrast, visible focus states, keyboard navigation for menus/drawers/modals, labelled forms, `prefers-reduced-motion`.
- **Mobile first:** most Indian traffic is mobile (confirm with the GA4 baseline). Bottom-sheet filters, sticky buy bar on product pages, thumb-reachable CTAs.

## 2. UX improvements over the current site

- Mega menu for Products with category images and featured series
- Collection pages with real filters (capacity, layers, brand, colour, and price for buyable items) and sorting
- Product pages with spec table, layer explainer, assembly animation, FAQs, downloads, warranty badge and related products
- Instant search with suggestions
- Clear Buy vs Enquire actions; the enquiry form opens in a drawer without leaving the page
- Compare tray
- Dealer locator with a map and "use my location"
- Warranty registration easy to find from the header ("Register warranty") and the homepage
- Fixes for current issues: duplicated brands section, placeholder FAQ text, "Rs. 1.00" prices, dead "#" links

## 3. Animation stack

| Library | Use |
|---|---|
| **Lenis** (`lenis`, `lenis/react`) | Smooth scrolling site-wide |
| **GSAP core** | All tweens and timelines |
| **ScrollTrigger** | Scroll-driven reveals, pinning, scrubbing (assembly sequences) |
| **SplitText** | Headline line/word reveals |
| **Flip** | Layout transitions: filter results reordering, fly-to-compare/cart, grid ↔ list |
| **`@gsap/react` (`useGSAP`)** | Scoped animations with automatic cleanup in React |
| React Three Fiber + drei (optional) | Interactive 3D/360° product viewer |

GSAP, including all its plugins, is free for commercial use since v3.13.

### Integration rules (Next.js App Router)

1. A single client `SmoothScrollProvider` in the root layout creates Lenis with `autoRaf: false`, drives it from `gsap.ticker`, and updates ScrollTrigger on every Lenis scroll:
   ```ts
   lenis.on('scroll', ScrollTrigger.update);
   gsap.ticker.add((time) => lenis.raf(time * 1000));
   gsap.ticker.lagSmoothing(0);
   ```
2. GSAP plugins are registered once, in a client module (`components/motion/gsap.ts`).
3. Every animated component uses `useGSAP(() => { … }, { scope: ref })`, so its animations are killed on unmount and route change.
4. On route change: scroll to top through Lenis, then call `ScrollTrigger.refresh()` once images and fonts have loaded.
5. Modals, drawers, the cart and the mega menu call `lenis.stop()` while open; scrollable inner areas get `data-lenis-prevent`.
6. Anchor links (e.g. `#board-of-directors`) use `lenis.scrollTo()` with a header offset.
7. `prefers-reduced-motion: reduce` → Lenis off (native scroll), reveals become instant fades, no pinning or scrubbing, and assembly sections show the final assembled image plus a step list.
8. Touch devices use native touch scrolling (Lenis `syncTouch` off) and lighter effects.
9. Admin preview pages run the same animations.

### SEO-safe animation rules

- All text and images are in the server-rendered HTML. Animations only change `transform`/`opacity` after hydration.
- Never ship content with `opacity: 0` in CSS. Hide-before-reveal styles apply only under a `.js-motion` class that the client adds, so crawlers and no-JS users see everything.
- The LCP element (hero heading/poster) is never delayed by an entrance animation.
- Animate only `transform` and `opacity` (GPU-friendly); no animations that shift layout (CLS).
- GSAP/Three code is split per page and loaded only where it's used; 3D loads on interaction or when scrolled into view.
- SplitText keeps an `aria-label` with the full sentence on the original element, so screen readers read it whole.

## 4. Animation presets (selectable per CMS block)

| Preset | Effect |
|---|---|
| `none` | Static |
| `fade-up` | Opacity + 30 px rise on enter |
| `stagger-children` | Children reveal in sequence (cards, logos) |
| `split-text-reveal` | Headline lines slide up from behind a mask |
| `parallax-media` | Background image/video moves slower than the scroll |
| `scale-in-media` | Image scales from 1.15 → 1 inside its frame |
| `count-up` | Numbers count up (stats block) |
| `marquee` | Endless horizontal scroll (logo wall), pauses on hover |
| `pin-steps` | Section pins while steps change (process, layer explainer) |
| `horizontal-scroll` | Pinned horizontal track (timeline, series) |
| `assembly-sequence` | Scroll-scrubbed product assembly (§5) |

Editors choose the preset; developers own the timings and easing curves, so motion stays consistent across the site.

## 5. Product assembly animations

Goal: show how a product is built, controlled by scroll — e.g. the **Ten-X tank's 10 layers** wrapping together, a tap's parts assembling, a toilet seat cover's hinge mechanism.

| Approach | How | Pros | Cons | Use for |
|---|---|---|---|---|
| **A. Scroll-scrubbed image sequence** (recommended) | A 3D artist renders the assembly as 90–150 frames (Blender, from CAD or modelled). Frames → WebP/AVIF in two sizes (mobile/desktop). A canvas draws the frame matching scroll progress (ScrollTrigger scrub); text callouts animate per step | Photoreal, smooth, works on every phone, predictable performance | Fixed camera angle; ~3–6 MB of frames per product (loaded progressively) | Flagship tanks (Ten-X, Granito, Cool, Safe), hero sections |
| **B. Real-time 3D** (React Three Fiber) | GLB model (Draco-compressed) with parts as separate meshes; a GSAP timeline moves the parts; the user can rotate | Interactive, 360°, small download if the model is simple | Needs clean, optimised models, more GPU and more dev time | Optional 360° viewer on product pages, 1–3 hero products |
| **C. 2D layered exploded view** | Layer PNGs/SVGs (cut-away tank wall); GSAP moves them apart and together | Cheap, light, quick to produce | Less impressive | Remaining products, layer explainer block |

**Recommendation:** A for 4–6 flagship products, C for the layer explainer on every tank page, and B only if the client wants an interactive viewer and can supply CAD files.

**Pipeline**

1. The client supplies CAD/3D files (STEP/OBJ/FBX), or reference photos + drawings.
2. The 3D artist models/cleans the product, matches materials to real product colours, animates the assembly (fixed camera), and renders the frames + a final still.
3. Export frames at 1920×1080 (desktop) and 1080×1350 (mobile), WebP quality ~75, named `0001.webp…`.
4. Upload to the product's `assembly_frames` media collection in admin and set step labels and frame ranges.
5. The frontend `AssemblySequence` component preloads the first frame (which is also the poster), loads the rest progressively as the section nears the viewport, draws to a `<canvas>` with a capped `devicePixelRatio`, and scrubs with a pinned ScrollTrigger.
6. Reduced motion or Save-Data (`navigator.connection.saveData`) → show the final still + numbered steps.

Budget per sequence: ≤ 150 frames, ≤ 6 MB desktop / ≤ 3 MB mobile, first frame ≤ 120 KB.

## 6. Performance budget

| Metric | Target |
|---|---|
| LCP (mobile, p75 field) | ≤ 2.5 s |
| INP (mobile, p75 field) | ≤ 200 ms |
| CLS (mobile, p75 field) | ≤ 0.1 |
| JS on initial load (home) | ≤ 200 KB gzipped incl. React (GSAP core + ScrollTrigger is roughly 40–45 KB) |
| Hero video | ≤ 2.5 MB on mobile, poster image first, no autoplay under Save-Data |
| Images | next/image, AVIF/WebP, correct `sizes`, lazy below the fold |
| Fonts | ≤ 2 families, subset, `font-display: swap`, display face preloaded |

Test on a mid-range Android phone (around 4 GB RAM) over 4G, not just on desktop.

## 7. Motion plan by page

| Page | Motion |
|---|---|
| Header | Hides on scroll down, shows on scroll up; mega menu stagger |
| Home | Hero headline split-reveal over video; category tiles stagger + hover tilt; pinned Best of Vectus showcase with the Ten-X assembly sequence; stats count-up; client logo marquee; brands strip; FAQ accordion height animation |
| Collection | Filter changes animate with Flip; cards stagger in; sticky filter bar |
| Product | Gallery crossfade; sticky buy box; layer explainer (pin-steps); assembly sequence; spec rows reveal; fly-to-cart / fly-to-compare |
| Compare | Columns Flip on add/remove; "differences only" collapse animation |
| About | Horizontal-scroll timeline (1989 → today); plant map pins drop in; team grid stagger |
| Blog | Reading progress bar; image scale-in |
| Careers | Job cards stagger; apply drawer |
| Warranty scan | Light "verified ✔" micro-animation only |
| Page transitions | Short fade/clip between routes (View Transitions API where supported, GSAP fallback) |

## 8. Design deliverables

1. Moodboard + 2 style directions → client picks one
2. Design system in Figma (tokens, type, components, states)
3. Hi-fi desktop + mobile screens: Home, Collection, Product (Buy and Enquiry versions), Compare, Cart, Checkout, Account, Warranty scan & registration, Blog list/post, Careers list/detail/apply, About, Contact, Dealer locator, 404
4. Motion prototype (video or coded) of the hero, the assembly sequence and the compare tray
5. Handoff: Figma dev mode + motion spec (durations, easings, triggers)
