# Portfolio Website Spec

## Overview
Personal developer portfolio for Sameer Khan — Senior MEAN Stack Developer (8 years).
Built with Astro, hosted on GitHub Pages at sameerthekhans.github.io.

## Goals
- Present professional identity and skills clearly
- Mobile-first experience (primary target: mobile visitors)
- Visually engaging with smooth animations
- Fast load time and good Lighthouse score

## Tech Stack
- **Framework:** Astro
- **Animations:** GSAP + ScrollTrigger (staggered card entrance, stats count-up)
- **Styling:** Plain CSS (CSS Grid bento layout)
- **Package manager:** pnpm
- **Deployment:** GitHub Actions → GitHub Pages

## Design — Bento Grid
Dark, minimal, card-based layout inspired by GridX but with richer content.

### Tokens
- `--bg: #0a0a0a` — page background
- `--surface: #111111` — card background
- `--surface-2: #161616` — elevated card (CTA)
- `--text: #f0f0f0`
- `--muted: #666666` — eyebrows, labels
- `--muted-text: #999999` — body text
- `--border: #1e1e1e` — default card border
- `--border-hover: #2e2e2e` — hover border
- `--accent: #00e5ff` — cyan, used sparingly
- `--radius: 16px` — card border-radius
- Typography: Space Grotesk (headings, labels) + Inter (body)

### Nav (sticky)
- Logo left: "SK" in accent cyan
- Nav links center (hidden on mobile): Home · Experience · Skills · Contact
- CTA right: "Let's talk" pill button

### Ticker (below nav)
- Full-width marquee: MEAN STACK DEVELOPER ★ 8 YEARS EXPERIENCE ★ ANGULAR SPECIALIST ★ ...
- Infinite CSS animation, 28s loop

## Grid Layout (3-column desktop → 2-col tablet → 1-col mobile)

| Card            | Desktop placement       | Content |
|-----------------|------------------------|---------|
| **Hero**        | col 1–2, row 1–2        | Photo placeholder (SK initials + "Available" badge), name, tagline, top-3 skill pills |
| **Current Role**| col 3, row 1            | HCL badge, Senior Consultant, company, period |
| **About**       | col 3, row 2            | Profile summary paragraph |
| **Skills**      | col 1, row 3            | Tag cloud — primary: Angular, Node.js, TypeScript, AWS (accent); secondary: Express, MongoDB, RxJS, NGXS, Web Components, Cypress, Playwright, Jest, React, Firebase, SQL, Git, Figma |
| **Experience**  | col 2–3, row 3          | Timeline: HCL (Oct 2021–Present) + Nekkanti Systems (Aug 2016–Oct 2021) with dot + description |
| **Achievements**| col 1, row 4            | 4 bullets: SAFe® 5, 5K SO, led 15 members, 60 defects |
| **Education**   | col 2, row 4            | MBA Finance 2019 + B.Com 2016; "Self-taught developer ✦" label |
| **Profiles**    | col 3, row 4            | GitHub, LinkedIn, Email as bordered link rows |
| **Stats**       | col 1–2, row 5          | Count-up: 8+ Years · 15 Team Members · 5K+ SO Reps |
| **CTA**         | col 3, row 5            | "Let's work together." (together in accent), snowflake decoration |

### Card anatomy
- `.eyebrow` — 0.6rem uppercase label top-left
- `h3` — card title
- `.card-arrow` — 38px circle bottom-right, arrow SVG; glows accent on card hover
- Hover: `translateY(-3px)` + `box-shadow` + `border-color` transition

## Animations (GSAP)
- **Entrance:** all `.card` elements fade+slide up, stagger 0.07s, on page load
- **Nav + ticker:** fade down, 0.5s, on load
- **Stats count-up:** ScrollTrigger `once: true`, triggers at `top 85%`
- **Ticker:** CSS `animation: marquee` infinite (no GSAP needed)
- Respect `prefers-reduced-motion`

## Content (real data from resume)
- **Name:** Sameer Khan
- **Title:** Senior MEAN Stack Developer
- **Email:** sameerthekhans@gmail.com
- **GitHub:** github.com/sameerthekhans
- **LinkedIn:** linkedin.com/in/sameerthekhans
- **Current:** Lead Software Engineer @ EPAM Systems (Oct 2024–Present)
- **Previous:** Senior Consultant @ HCL Technologies (Oct 2021–Oct 2024)
- **Earlier:** Senior Front End Developer @ Nekkanti Systems (Aug 2016–Oct 2021)
- **Certifications:** SAFe® 5 Practitioner (2023)
- **Education:** MBA Finance — Azad College (2019); B.Com Computers — St. Pauls (2016)
- **Photo:** placeholder (SK initials, gradient background) — to be replaced with real photo

## Pending / Future
- Replace SK initials placeholder with actual profile photo
- Add real project screenshots to a Projects card
- Add LinkedIn/GitHub real URLs once confirmed
- Consider adding a "Works / Projects" card in a future grid row

## Constraints
- No backend — fully static site
- All content hardcoded (no CMS)
- Must deploy cleanly via existing GitHub Actions workflow
- Single-page (`index.astro`) — no routing needed yet
