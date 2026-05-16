# Home Page Redesign Spec

## Overview

Full redesign of the landing/home page from a bento-grid card layout to a full-screen hero layout
with a dual-column hero, a Featured Projects carousel, and a Stats bar.
The rest of the existing card components (Skills, Experience, etc.) are preserved for future pages.

**Stack:** Astro 5 · Plain CSS custom properties · Three.js · simplex-noise · No Tailwind · TypeScript
**Theme:** Dark-first, with existing `data-theme="light"` toggle system
**Branch:** `new-template`

---

## Self-Critique & Issues Fixed (v3)

Issues found during review against actual codebase — all resolved in this version.

| # | Severity | Issue | Fix applied |
|---|---|---|---|
| 1 | **Critical** | `astro:before-swap` event never fires — project has no View Transitions | Changed cleanup to `window.addEventListener('beforeunload', cleanup)` |
| 2 | **Critical** | `SimplexNoise` is not part of Three.js — missing package | Added `simplex-noise` to package list and install step |
| 3 | **Critical** | `EffectComposer` / `UnrealBloomPass` are not in the Three.js main bundle | Documented correct `three/addons/postprocessing/` import paths |
| 4 | **Critical** | `TechStack` DOM placement contradiction — CSS cannot move a DOM node between grid containers | Resolved: TechStack stays inside HeroSection always; hero is not capped to `100vh` on mobile |
| 5 | **Significant** | CSS variable values in spec are wrong — e.g. `--border: rgba(255,255,255,0.08)` but actual is `#1e1e1e`; `--muted-text: #8888aa` but actual is `#999999` | Corrected all values to match actual `global.css` |
| 6 | **Significant** | `.eyebrow` class uses `color: var(--muted)` — but hero eyebrow "HELLO, I'M" is cyan in design | Added `.eyebrow--accent` variant to CSS plan |
| 7 | **Significant** | Component tree vs. section 2 contradiction — TechStack/TrustedBy shown as children of both HeroText and HeroSection | Resolved: they are direct children of HeroSection, NOT inside HeroText |
| 8 | **Significant** | Hero container missing `position: relative` for absolute brain positioning on mobile | Added explicitly to HeroSection CSS spec |
| 9 | **Significant** | Gradient text for "Khan" uses hardcoded hex — won't adapt to light theme | Added `[data-theme="light"]` override |
| 10 | **Significant** | Stats bar `repeat(4, 1fr)` on ~375px phone — "Stack Overflow Reps" label will overflow | Changed to `repeat(2, 1fr)` below 480px with stacked layout |
| 11 | **Significant** | Nav currently has 4 links; spec adds "About" and "Projects" which are future pages, not current anchors | Clarified: all 6 links use `href="#"` placeholder until pages exist |
| 12 | **Minor** | `prefers-reduced-motion` not addressed in `brain.ts` | Added to brain.ts spec |
| 13 | **Minor** | Footer not mentioned — spec says "rewrite index.astro" but `Footer.astro` exists | Confirmed: keep Footer, import it in index.astro |
| 14 | **Minor** | `<head>` content not addressed — Google Fonts, meta, title would be lost on rewrite | Explicitly listed what to preserve from current index.astro |
| 15 | **Minor** | TechStack tab indicator interaction detail missing — what triggers it? | Defined as CSS-only `:focus-within` / hover state, decorative on first icon by default |
| 16 | **Minor** | "View my work" specified as button — should be `<a>` since it navigates | Changed to `<a href="#projects">` |
| 17 | **Minor** | Third project card is TBD with no fallback | Defined placeholder project data |
| 18 | **Minor** | Project card absolute screenshot would break card height without min-height | Added `min-height` and `overflow: hidden` to card spec |

---

## Layout Architecture

### Desktop (`> 768px`)

```
┌──────────────────────────────────────────────────────────┐
│  NAVBAR — logo · nav links · theme toggle · Let's talk   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   HERO SECTION  (naturally tall — not capped to 100vh)   │
│   ┌──────────────────────┬───────────────────────────┐   │
│   │  "HELLO, I'M"        │                           │   │
│   │  Sameer              │   Three.js canvas         │   │
│   │  Khan  (gradient)    │   (particle brain,        │   │
│   │  MEAN Stack Dev |    │    mouse-interactive)     │   │
│   │  description         │                           │   │
│   │  [CTA] [CV]          │                           │   │
│   ├──────────────────────┤                           │   │
│   │  TECH STACK icons    │                           │   │
│   ├──────────────────────┤                           │   │
│   │  TRUSTED BY logos    │                           │   │
│   └──────────────────────┴───────────────────────────┘   │
│                                                          │
│   FEATURED PROJECTS SECTION                              │
│   ┌──────────┬──────────┬──────────┐                     │
│   │ Card 1   │ Card 2   │ Card 3   │  (grid, 3 col)      │
│   └──────────┴──────────┴──────────┘                     │
│                                                          │
│   STATS BAR                                              │
│   [ 9+ Years ] [ 15 Team ] [ 5K+ SO ] [ 60+ Projects ]  │
│                                                          │
│   FOOTER                                                 │
└──────────────────────────────────────────────────────────┘
```

### Mobile (`< 768px`)

```
┌────────────────────────────────┐
│  NAVBAR — logo · ☀ · CTA · ☰  │
├────────────────────────────────┤
│  HERO  (position: relative)    │
│  HELLO, I'M    ╔════════════╗  │
│  Sameer        ║ brain      ║  │
│  Khan          ║ (absolute, ║  │
│                ║  top-right)║  │
│  MEAN Stack|   ╚════════════╝  │
│  description                   │
│  [View my work] [Download CV]  │
│                                │
│  TECH STACK  (inside hero,     │
│  below CTAs, natural flow)     │
│  ◉ ◉ ◉ ◉ ◉                    │
│                                │
│  TRUSTED BY  (hidden on mob.)  │
├────────────────────────────────┤
│  FEATURED PROJECTS  View all → │
│  ← [ card 1 ] [ card 2… ] →   │
│       • ○ ○  (pagination dots) │
├────────────────────────────────┤
│  [ 9+ ]  [ 15 ]                │
│  Years   Team                  │
│  [ 5K+ ] [ 60+ ]               │
│  SO Reps  Projects             │
├────────────────────────────────┤
│  FOOTER                        │
└────────────────────────────────┘
```

---

## 1. Navbar (`Nav.astro` — update existing)

### Desktop

- `position: sticky` (already), full-width, `z-index: 100`
- Background: `var(--nav-bg)` (already set — `rgba(10,10,10,0.85)`) with `backdrop-filter: blur(12px)`
- **Left:** "SK" circular logo (reuse existing `/sameer-logo-xs.png`)
- **Center:** 6 nav links — Home · About · Experience · Skills · Projects · Contact
  - **Note:** "About" and "Projects" use `href="#"` — they are future pages, anchors don't exist yet
  - Active link (Home) has a small cyan dot underneath: `::after` pseudo-element, `4px` circle, `var(--accent)`
  - Links use `var(--muted-text)` default, hover → `var(--text)`
- **Right:** Gear icon (decorative, no action) · existing theme toggle button · "Let's talk" pill button

### Mobile (`< 768px`)

- Nav links already hidden at `max-width: 900px` via existing CSS — no change needed there
- Add hamburger `☰` icon button to `.nav-actions` (renders icon only; drawer deferred)
- Order in `.nav-actions`: theme toggle · "Let's talk" · hamburger

### "Let's talk" button

- Already exists as `.nav-cta` — update styles:
  - Add paper plane SVG icon before text
  - Ensure `background: var(--accent)`, `color: #000`
  - Pill shape already set (`border-radius: 100px`)

---

## 2. Hero Section (`HeroSection.astro` — new)

Layout shell only. Owns the two-column grid and `position: relative` for mobile brain overlay.

```css
.hero {
  display: grid;
  grid-template-columns: 1fr 1fr;   /* desktop */
  min-height: calc(100vh - 64px);
  padding: 4rem 2rem;
  position: relative;               /* required: mobile brain is position:absolute */
  align-items: start;
}

.hero-left {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

@media (max-width: 768px) {
  .hero {
    grid-template-columns: 1fr;
  }
  /* BrainCanvas is pulled out of grid flow and positioned absolutely — see BrainCanvas spec */
}
```

Children (in order inside `.hero-left`): `<HeroText />`, `<TechStack />`, `<TrustedBy />`
Right column: `<BrainCanvas />`

---

## 3. HeroText (`HeroText.astro` — new)

Owns: eyebrow, name, role+cursor, description, CTA buttons. Nothing else.

**Eyebrow**
- `"HELLO, I'M"` — use `.eyebrow--accent` (new variant, see CSS section)
- `letter-spacing: 0.14em`, uppercase

**Name**
- `"Sameer"` — white, `font-weight: 800`, `font-size: clamp(2.5rem, 6vw, 5rem)`, Space Grotesk
- `"Khan"` — same size/weight, gradient text:
  ```css
  /* dark theme */
  background: linear-gradient(90deg, #00e5ff 0%, #7b5cff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;

  /* light theme override */
  [data-theme="light"] .name-gradient {
    background: linear-gradient(90deg, #0097b2 0%, #5b3fd4 100%);
  }
  ```

**Role + Cursor**
```css
@keyframes blink { 0%, 100% { opacity: 1 } 50% { opacity: 0 } }
.cursor { animation: blink 1s step-end infinite; }

@media (prefers-reduced-motion: reduce) {
  .cursor { animation: none; opacity: 1; }
}
```

**Description**
- `var(--muted-text)` (`#999999` dark / `#5a5a5a` light — already in global.css)
- `font-size: 1rem`, `max-width: 400px`, `line-height: 1.7`

**CTAs**
- Primary: `<a href="#projects" class="btn-primary">View my work →</a>` — it's a link, not a button
- Secondary: `<a href="/cv.pdf" class="btn-ghost" download>Download CV ↓</a>`
- `display: flex`, `gap: 1.5rem`, `align-items: center`, `flex-wrap: wrap`

---

## 4. TechStack (`TechStack.astro` — new)

Owns: icon badge row, eyebrow label, tab underline indicator.

**Eyebrow:** `"TECH STACK"` — `.eyebrow--accent`

**Icon badges** (`width/height: 44px`, `border-radius: 50%`):

| Icon | Brand color | Light-mode bg adjustment |
|---|---|---|
| Angular | `#DD0031` | same |
| Node.js | `#339933` | same |
| TypeScript | `#3178C6` | same |
| NGXS | `#1a1a2e` | `#e8e8f0` (too dark on white) |
| MongoDB | `#47A248` | same |

**Tab indicator** — decorative, CSS-only, always sits under first icon:
```css
.tech-icons { position: relative; display: flex; gap: 0.75rem; }
.tech-icons::after {
  content: '';
  position: absolute;
  bottom: -8px;
  left: 10px;          /* centered under first icon */
  width: 24px;
  height: 3px;
  background: var(--accent);
  border-radius: 2px;
}
```
No JS needed — purely decorative, mirrors the design screenshot.

**SVG icons:** Inline SVG in component. No CDN. Sources: Angular shield, Node.js hex, TS square, NGXS "N", MongoDB leaf.

---

## 5. TrustedBy (`TrustedBy.astro` — new)

Owns: company names strip. Controls its own visibility.

```css
/* hidden on mobile — component owns this rule */
@media (max-width: 768px) {
  .trusted-by { display: none; }
}
```

- Eyebrow: `"TRUSTED BY EXPERIENCE"` — `.eyebrow` (muted, not accent)
- Names: `HCL · EPAM · Nekkanti · Cypress · Angular · NGXS`
- `color: var(--muted-text)`, `gap: 1.5rem`, `font-size: 0.8rem`, `flex-wrap: wrap`

---

## 6. BrainCanvas (`BrainCanvas.astro` — new)

Owns: `<canvas>` element, glow wrapper div, ResizeObserver, cleanup. Does NOT contain Three.js logic.

```html
<div class="brain-wrap">
  <canvas id="brain-canvas"></canvas>
</div>
```

```css
.brain-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 400px;
}
.brain-wrap::before {   /* ambient glow — visible before/without bloom */
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, rgba(88,66,255,0.25) 0%, transparent 70%);
  pointer-events: none;
  z-index: 0;
}
canvas {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  display: block;
}

/* Mobile: absolute overlay top-right of hero */
@media (max-width: 768px) {
  .brain-wrap {
    position: absolute;
    top: 0;
    right: 0;
    width: 58%;
    height: 320px;
    min-height: unset;
    opacity: 0.75;
    z-index: 0;
    pointer-events: none;   /* don't block text interaction on mobile */
  }
}
```

**`<script>` (in BrainCanvas.astro):**
```ts
import { initBrainScene } from '../lib/brain';
const canvas = document.getElementById('brain-canvas') as HTMLCanvasElement;
const cleanup = initBrainScene(canvas);
window.addEventListener('beforeunload', cleanup, { once: true });
// NOTE: astro:before-swap would be correct if View Transitions is ever added
```

---

## 7. `src/lib/brain.ts` — All Three.js logic

### Packages required
```
pnpm add three simplex-noise
pnpm add -D @types/three
```

### Imports (Three.js addons are NOT in the main bundle)
```ts
import * as THREE from 'three';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { createNoise3D } from 'simplex-noise';
```

### Scene setup
```
WebGLRenderer  antialias: true, alpha: true, canvas: passed-in element
  └── Scene
       ├── PerspectiveCamera  FOV 60, aspect from canvas, near 0.1, far 100, z=3
       ├── Points
       │    ├── SphereGeometry(1.2, 64, 64)
       │    ├── Vertices displaced by createNoise3D  amplitude 0.3
       │    └── PointsMaterial  size=0.012, color=#00e5ff, sizeAttenuation=true
       └── EffectComposer (desktop only — skip on mobile for perf)
            ├── RenderPass
            └── UnrealBloomPass  strength=1.8, radius=0.5, threshold=0
```

### Mobile performance
```ts
const isMobile = window.matchMedia('(max-width: 768px)').matches;
// isMobile → skip EffectComposer, reduce geometry: SphereGeometry(1.2, 32, 32)
// isMobile → pointer-events: none on canvas already set in CSS
```

### prefers-reduced-motion
```ts
const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
// reducedMotion → skip RAF animation loop entirely; render one static frame only
```

### Mouse / Touch interaction
```ts
let targetX = 0, targetY = 0, currentX = 0, currentY = 0;
const DAMP = 0.04;
let idleTimer: number;

window.addEventListener('mousemove', (e) => {
  targetX = (e.clientX / window.innerWidth - 0.5) * Math.PI * 0.5;
  targetY = (e.clientY / window.innerHeight - 0.5) * Math.PI * 0.3;
  clearTimeout(idleTimer);
  idleTimer = setTimeout(() => { /* resume auto-rotate */ }, 2500);
});

// RAF loop: currentX += (targetX - currentX) * DAMP — lerp to mouse position
// Auto-rotate: mesh.rotation.y += 0.003 when idle
```

### ResizeObserver
```ts
const ro = new ResizeObserver(() => {
  const { width, height } = canvas.getBoundingClientRect();
  renderer.setSize(width, height, false);
  camera.aspect = width / height;
  camera.updateProjectionMatrix();
});
ro.observe(canvas);
```

### Cleanup (returned function)
```ts
return () => {
  ro.disconnect();
  window.removeEventListener('mousemove', onMouseMove);
  cancelAnimationFrame(rafId);
  renderer.dispose();
  geometry.dispose();
  material.dispose();
};
```

---

## 8. Featured Projects (`FeaturedProjects.astro` + `ProjectCard.astro`)

### Project data (hardcoded — no CMS yet)

| # | Title | Description | Tags | Icon color |
|---|---|---|---|---|
| 1 | Doctor Booking App | Physician referral booking app rebuilt for Angular 11 | Angular · NGXS · Cypress | `#3b82f6` blue |
| 2 | Component Library | Reusable enterprise Angular component library | Angular · TypeScript | `#22c55e` green |
| 3 | Portfolio Site | This portfolio — built with Astro 5 and Three.js | Astro · TypeScript · CSS | `#a855f7` purple |

### ProjectCard layout

```
┌────────────────────────────────────────┐  min-height: 200px
│  [icon sq]  Title                      │
│             description (2 lines max)  │
│                          [screenshot]  │  absolute, right:-16px, top:1rem
│  ● Tag  ● Tag  ● Tag                   │
└────────────────────────────────────────┘
```

- `overflow: hidden` on card — clips absolute screenshot
- `min-height: 200px` — prevents height collapse with absolute image
- Screenshot: `width: 45%`, `border-radius: 8px`, `box-shadow: var(--card-shadow)`
- Screenshot source: `public/projects/project-{n}.png` — use placeholder grey rect if not available

### FeaturedProjects layout

**Desktop:** `display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem`
**Tablet (768–1024px):** `repeat(2, 1fr)`
**Mobile:** `overflow-x: scroll; scroll-snap-type: x mandatory; display: flex`
- Each card: `min-width: 80vw; scroll-snap-align: start; flex-shrink: 0`

### Dot pagination (mobile only, JS)

```ts
const cards = document.querySelectorAll('.project-card');
const dots = document.querySelectorAll('.dot');
const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const i = [...cards].indexOf(entry.target as HTMLElement);
      dots.forEach((d, j) => d.classList.toggle('active', i === j));
    }
  });
}, { threshold: 0.5 });
cards.forEach(c => observer.observe(c));
```

Active dot: `width: 20px`, filled `var(--accent)`. Inactive: `width: 8px`, gray.
Dots only rendered via `class="dots-wrap"` — CSS hides on desktop: `@media (min-width: 768px) { .dots-wrap { display: none } }`.

---

## 9. Stats Bar (`StatsBar.astro` — new)

| Stat | Icon | Number | Label | Icon bg |
|---|---|---|---|---|
| 1 | Person outline SVG | 9+ | Years Experience | `#3b82f6` |
| 2 | Group/team SVG | 15 | Team Members Led | `#22c55e` |
| 3 | `</>` code SVG | 5K+ | Stack Overflow Reps | `#a855f7` |
| 4 | Paper plane SVG | 60+ | Projects Delivered | `#14b8a6` |

```css
.stats { display: grid; grid-template-columns: repeat(4, 1fr); }

@media (max-width: 480px) {
  .stats { grid-template-columns: repeat(2, 1fr); }
}
```

---

## 10. Component Architecture & SOLID Principles

### Component tree

```
index.astro                        ← page shell: <head>, section order, Footer
├── Nav.astro                      ← navigation only
├── HeroSection.astro              ← two-column grid layout + position:relative. No content.
│   ├── (left col) HeroText.astro  ← eyebrow, name, role, description, CTAs
│   ├── (left col) TechStack.astro ← icon badges + decorative tab indicator
│   ├── (left col) TrustedBy.astro ← company strip, hides itself on mobile
│   └── (right col) BrainCanvas.astro  ← <canvas> + ResizeObserver + cleanup
│                    └── imports src/lib/brain.ts  ← ALL Three.js logic
├── FeaturedProjects.astro         ← grid/carousel layout + dot pagination
│   └── ProjectCard.astro          ← single card
├── StatsBar.astro                 ← 4-stat grid
└── Footer.astro                   ← unchanged, keep as-is
```

### SRP boundary table

| File | Owns | Does NOT own |
|---|---|---|
| `index.astro` | `<head>`, section order, body | Any component content |
| `Nav.astro` | Nav bar layout, hamburger icon | Page content |
| `HeroSection.astro` | Grid columns, `position: relative` | Any text, icons, canvas |
| `HeroText.astro` | Name, role, description, CTAs | Layout, icons |
| `TechStack.astro` | Icon data, badges, indicator | Hero layout |
| `TrustedBy.astro` | Company list, its own mobile visibility | Anything else |
| `BrainCanvas.astro` | `<canvas>` element, ResizeObserver | Three.js internals |
| `brain.ts` | Entire Three.js scene | DOM structure |
| `ProjectCard.astro` | Single card markup + styles | Grid/carousel |
| `FeaturedProjects.astro` | Layout, scroll, dot pagination | Card internals |
| `StatsBar.astro` | 4 stats, icons, numbers | Other sections |

---

## 11. CSS Architecture

### New utility classes in `global.css`

```css
/* Variant of existing .eyebrow — accent colour instead of muted */
.eyebrow--accent {
  font-size: 0.6rem;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--accent);
}

.btn-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  border-radius: 999px;
  background: var(--accent);
  color: #000;
  font-weight: 600;
  font-size: 0.9rem;
  text-decoration: none;
  transition: opacity 0.2s;
}
.btn-primary:hover { opacity: 0.85; }

.btn-ghost {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--text);
  font-size: 0.9rem;
  text-decoration: none;
  transition: color 0.2s;
}
.btn-ghost:hover { color: var(--accent); }

.gradient-text {
  background: linear-gradient(90deg, #00e5ff 0%, #7b5cff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
[data-theme="light"] .gradient-text {
  background: linear-gradient(90deg, #0097b2 0%, #5b3fd4 100%);
  -webkit-background-clip: text;
  background-clip: text;
}
```

### Verified CSS custom properties (from actual global.css)
```css
/* dark (root) */          /* light ([data-theme="light"]) */
--bg: #0a0a0a              --bg: #f5f5f5
--surface: #111111         --surface: #ffffff
--surface-2: #161616       --surface-2: #efefef
--text: #f0f0f0            --text: #0d0d0d
--muted: #666666           --muted: #888888
--muted-text: #999999      --muted-text: #5a5a5a
--border: #1e1e1e          --border: #e4e4e4
--border-hover: #2e2e2e    --border-hover: #d0d0d0
--accent: #00e5ff          --accent: #0097b2
--nav-bg: rgba(10,10,10,.85) --nav-bg: rgba(245,245,245,.9)
--card-shadow: ...         --card-shadow: ...
--radius: 16px             (not redeclared)
```

---

## 12. What to Preserve from current `index.astro`

These must be kept in the rewritten file:

- `<meta charset>`, `<meta viewport>`
- `<title>Sameer Khan | Senior MEAN Stack Developer</title>`
- `<meta name="description">` value
- Google Fonts preconnect + `<link>` for Space Grotesk + Inter
- Inline `is:inline` FOUC prevention script (reads localStorage + prefers-color-scheme)
- `<Footer />` import and render

Remove: `epamTenure` calculation, all bento card imports, `<style is:global>` bento CSS block.

---

## 13. Responsive Breakpoints

| Breakpoint | Navbar | Hero columns | Brain | Projects | Stats |
|---|---|---|---|---|---|
| `> 1024px` | Full 6 links | `1fr 1fr` | Right col, full height | 3-col grid | 4-col row |
| `768–1024px` | Full 6 links | `1fr 1fr` (brain smaller) | Right col | 2-col grid | 4-col row |
| `< 768px` | Links hidden, hamburger icon | Single col, brain `position: absolute` top-right | 58% width, 320px, opacity 0.75, pointer-events none | Horizontal carousel + dots | 2×2 grid |
| `< 480px` | Same | Same | Same | Same | 2-col (already covered above) |

---

## 14. Assets / Packages Required

| Asset | Action |
|---|---|
| `pnpm add three` | Three.js renderer |
| `pnpm add simplex-noise` | Noise for sphere displacement |
| `pnpm add -D @types/three` | TypeScript types |
| Tech stack SVG icons | Inline in TechStack.astro |
| Stats icons | Inline in StatsBar.astro |
| `public/projects/project-1.png` | Placeholder or real screenshot |
| `public/projects/project-2.png` | Placeholder or real screenshot |
| `public/projects/project-3.png` | Placeholder or real screenshot |
| `public/cv.pdf` | User provides — linked from Download CV button |

---

## 15. Implementation Order

1. `pnpm add three simplex-noise && pnpm add -D @types/three`
2. Update `global.css` — add `.eyebrow--accent`, `.btn-primary`, `.btn-ghost`, `.gradient-text`
3. Update `Nav.astro` — 6 links, active dot, hamburger icon, updated CTA button
4. Create `src/lib/brain.ts` — full Three.js scene, exports `initBrainScene`
5. Create `BrainCanvas.astro` — canvas + glow wrap + calls `initBrainScene`
6. Create `HeroText.astro` — eyebrow, name, role+cursor, description, CTAs
7. Create `TechStack.astro` — icon badges + decorative tab underline
8. Create `TrustedBy.astro` — company strip, hides on mobile via own style
9. Create `HeroSection.astro` — two-column grid, composes steps 5–8
10. Create `ProjectCard.astro` — single card
11. Create `FeaturedProjects.astro` — grid/carousel + dot pagination JS
12. Create `StatsBar.astro` — 4-stat responsive grid
13. Rewrite `index.astro` — preserve `<head>`, import all new components, add Footer
14. Test: brain mouse/touch, carousel swipe, dots, responsive breakpoints, dark/light theme, reduced-motion

---

## 16. Decisions Log

| Question | Decision |
|---|---|
| **Brain geometry** | ✅ Procedural sphere + simplex-noise |
| **3D tech** | ✅ Three.js |
| **Sidebar** | ✅ Removed |
| **Tech stack icons** | ✅ Inline SVG |
| **Typing animation** | CSS blink cursor; JS typewriter optional later |
| **Gear icon in nav** | Decorative only |
| **"Download CV"** | `href="/cv.pdf" download` — user provides PDF |
| **Mobile brain** | `position: absolute` top-right overlay, `pointer-events: none` |
| **"Khan" gradient** | `#00e5ff → #7b5cff` dark / `#0097b2 → #5b3fd4` light |
| **Mobile nav** | Hamburger icon only; drawer deferred |
| **"View my work" element** | `<a href="#projects">` — link not button |
| **TechStack on mobile** | Stays inside HeroSection in normal flow — hero is not height-capped |
| **Nav links for future pages** | `href="#"` placeholder until pages are built |
| **Stats on small mobile** | 2×2 grid below 480px |
| **Footer** | Kept — import unchanged Footer.astro in index.astro |
