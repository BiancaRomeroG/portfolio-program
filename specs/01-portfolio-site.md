# Spec 01 — `portfolio` (Worker W0)

Public site for Bianca Romero. Concept: **Event Stream**. Goal: a recruiter remembers it after 10 seconds, an engineer respects it after 60.

## 1. Scope

- Single-page home with sections (below) + case-study pages per project + 404.
- Bilingual: `en` default, `es` secondary. Routes `/` (en) and `/es`. Content in MDX/JSON, never hard-coded in components.
- Hosted on Vercel at `portfolio-<hash>.vercel.app` until the owner provides a domain.
- Out of scope: CMS, blog engine, contact backend (email copy + links only), 3D/WebGL.

## 2. Stack

Next.js 16 App Router, React 19, TypeScript strict, Tailwind v4, Motion (`motion/react`), Lenis (`lenis/react`) smooth scroll, GSAP + ScrollTrigger (only for pinned/stacked sections), `next-intl` for i18n, `@next/mdx` for case studies, `next/og` for OG images, Vercel Analytics, Playwright, Lighthouse CI. Fonts via `next/font`: Geist Sans, Geist Mono, Instrument Serif. No UI kit.

Repo layout: `apps/web` only plus `packages/config`. No API. Keep Turborepo for consistency.

## 3. Visual identity

- Palette and type from Spec 00 §4. Background `#08090B` with a 3% grain and a very subtle radial lime glow following the cursor on the hero (CSS variable updated by pointer, throttled).
- Grid: 12 columns, max width 1280, generous whitespace. Section labels in mono uppercase with a lime index (`01 / Work`).
- Custom cursor: 8px dot + 32px ring with `mix-blend-mode: difference`; ring scales 2x on links, shows a label ("Open", "Drag") on project cards. Hidden on touch devices.
- Magnetic buttons (pull toward cursor within 80px, spring back).
- Text reveals: headlines split into words, each word masked and slid up (clip-path), staggered 30ms, triggered `whileInView` once.

## 4. Sections (home)

1. **Nav** (fixed, top). Left: mono wordmark `bianca.romero`. Right: `Work`, `About`, `Experience`, `Contact`, language toggle `EN / ES`, accent button `Download CV` (PDF in `/public`). Background blurs after 40px scroll. Active section underline follows scroll (IntersectionObserver).
2. **Hero** (100vh). Background: `EventCanvas` (Canvas 2D, no WebGL): 6–8 nodes placed on a loose grid, connected by bezier "pipes"; particles travel along pipes at varying speeds; colours: muted grey particles, 1 in 12 lime; particles within 120px of the cursor accelerate and brighten; nodes pulse when a particle arrives. Runs at 60fps, pauses when tab hidden or offscreen, reduces to a static SVG under `prefers-reduced-motion`. Foreground: `BIANCA ROMERO` (Geist, 9–12vw, tight tracking) with word reveal; tagline in two lines: `Full Stack Engineer building *reliable* systems` / `where money and events flow.` (`reliable` in Instrument Serif italic lime). Two CTAs: accent `See the work` (scrolls to Work), ghost `GitHub`. Bottom-left: mono status line `Santa Cruz, BO · UTC-4 · open to remote roles` with a blinking lime dot. Bottom-right: scroll cue (animated line).
3. **Ticker**. Full-width marquee, mono 13px, two rows moving opposite directions, seamless loop, pauses on hover. Content: fake-but-plausible event lines, e.g. `webhook.received zoom.meeting.ended → attendance.reconciled`, `batch.settled 1,204 tx · 0 discrepancies`, `otp.sent channel=whatsapp`, `pr.opened by agent · review=human`. Pulls from `content/ticker.json`.
4. **Work** (the core). Three project cards, **stacked scroll**: each card is `position: sticky; top: 10vh`, 80vh tall; as the next card scrolls over, the previous one scales to 0.95 and dims (GSAP ScrollTrigger scrub, or Motion `useScroll` with per-card progress). Card layout: left 55% text (index, name, one-line pitch, 3 bullet outcomes, stack chips, links `Live demo`, `Code`, `Case study`), right 45% a **live mini-visual** unique per project, built in React, no screenshots:
   - LedgerFlow: a tiny double-entry ledger that animates a payment: two rows appear (debit/credit), amounts count up, a `balanced ✓` badge flashes.
   - Incident Copilot: a chat bubble types an answer with a citation chip appearing and a tool-call pill (`searchRunbooks`) blinking.
   - Pulse: a mini event pipeline: dot enters, splits into email/push/in-app lanes, one lane fails and retries to DLQ then replays.
   Card hover: subtle tilt (max 3°) following cursor; cursor label `Open`.
5. **About / Manifesto**. Two columns. Left sticky: portrait (if provided; else a generative avatar made of the EventCanvas nodes) and 3 counters animating on view (`3+ years in production`, `5 payment / messaging integrations`, `3 stacks shipped: .NET, Java, PHP → now TypeScript`). Right: 3 short paragraphs with word-reveal (from CV "Perfil", rewritten in first person, EN and ES). Below: a horizontal "principles" row with 4 items (Reliability over novelty · Specs before code · Docs are the product · Review everything AI writes).
6. **Experience**. Vertical timeline. Left rail with a lime progress line driven by `useScroll` (`scaleY`). Entries: Crehana, Linkser (one entry, QA → Dev badge), Beepay. Each: role, company, dates, 3 bullets max, stack chips. Content from `content/experience.{en,es}.json` based on the CV v2.
7. **Stack**. Grid of tech logos as mono text chips grouped by category (Backend, Frontend, Data & Messaging, Cloud & DevOps, Quality, AI tooling). Chips repel slightly from the cursor (Motion spring). Only technologies backed by jobs or by the three repos (CV rule).
8. **Contact / Footer**. Giant `Let's talk` (Instrument Serif italic, 14vw) with a hover wave distortion per letter; below: email with copy-to-clipboard (toast `Copied`), LinkedIn, GitHub, CV. Bottom mono line: `Built with Next.js, Motion and too much coffee · 2026 · view source`. Secondary marquee with the three project names.

## 5. Case-study pages (`/work/[slug]`, en and es)

MDX from `content/work/<slug>.{en,es}.mdx`, generated from each repo's `docs/case-study.md` (W0 adapts text, adds images). Layout: hero with project name, pitch, meta table (role, timeline, stack, links); sticky table of contents; sections Problem, Constraints, Architecture (Mermaid diagram rendered at build with `rehype-mermaid` or pre-rendered SVG), Key decisions (ADR-style callouts), Results (numbers), What I would do differently; prev/next project footer. Page transitions: shared element on the project name (Motion `layoutId`) when coming from the Work section.

## 6. Technical requirements

- Performance: LCP < 2.0 s on Vercel; canvas starts after first paint; fonts preloaded; images `next/image`; no layout shift from animations.
- Lighthouse CI thresholds per Spec 00 §6.
- SEO: metadata per locale, canonical + hreflang, `sitemap.xml`, `robots.txt`, JSON-LD `Person`, dynamic OG image with name and tagline on the Event Stream background.
- Accessibility: all animations respect reduced motion; focus states; semantic landmarks; marquee pausable; canvas `aria-hidden`.
- i18n: all strings in `messages/{en,es}.json`; language toggle preserves the section.
- Analytics: Vercel Analytics; custom events for CTA clicks and project card opens.
- Content model in `content/`: `profile.json`, `experience.*.json`, `projects.json` (slug, names, links, status: `in progress` until the repo hits Gate 4), `ticker.json`, `work/*.mdx`.
- Tests: Playwright e2e: home renders, nav scroll, language toggle, cards open case study, reduced-motion snapshot. Vitest for utilities (split text, i18n helpers).

## 7. Tasks

### Phase 1 — skeleton
- [W0-01] Scaffold Turborepo with `apps/web` (Next 16, TS, Tailwind v4, src dir, App Router), `packages/config`. pnpm, husky, commitlint, release-please, CI, Lighthouse CI config.
- [W0-02] Design tokens in `globals.css` (`@theme`), fonts via `next/font`, grain overlay, dark/light tokens, base typography scale.
- [W0-03] `next-intl` setup with `en`/`es`, `messages/*.json`, locale routing, language toggle.
- [W0-04] Layout: `SmoothScroll` (Lenis root, GSAP ticker sync), `Cursor`, `Nav` with scroll state, footer placeholder, `prefers-reduced-motion` hook.
- [W0-05] Animated primitives in `src/components/motion`: `RevealText`, `Magnetic`, `Marquee`, `Counter`, `SectionLabel`.
- [W0-06] Docs skeleton from templates; ADR-0001 stack versions; architecture.md (simple: Vercel, Next, content pipeline).

### Phase 2 — hero and work section
- [W0-07] `EventCanvas` with nodes, bezier pipes, particles, cursor influence, pause offscreen, reduced-motion fallback. Unit test for the particle scheduler (pure function).
- [W0-08] Hero composition, status line, CTAs, scroll cue, cursor glow.
- [W0-09] Ticker (two rows, seamless, pause on hover, data-driven).
- [W0-10] Work section: stacked sticky cards with scroll-linked scale/dim; card tilt; cursor label.
- [W0-11] Three live mini-visuals (Ledger, Copilot, Pulse) as self-contained components with their own timers; stop when offscreen.

### Phase 3 — rest of home, case studies
- [W0-12] About/Manifesto with counters and principles row; avatar fallback.
- [W0-13] Experience timeline with scroll progress.
- [W0-14] Stack grid with cursor repulsion.
- [W0-15] Contact/Footer with letter wave, copy email, links, secondary marquee.
- [W0-16] Case-study route with MDX pipeline, TOC, Mermaid rendering, prev/next, shared-element transition. Placeholder MDX for the three projects marked `in progress`.
- [W0-17] Playwright suite + reduced-motion snapshots; Lighthouse CI in workflow.

### Phase 4 — ship
- [W0-18] SEO: metadata, hreflang, sitemap, robots, JSON-LD, OG image route.
- [W0-19] Vercel project (orchestrator provides), env, Analytics, preview deployments on PR.
- [W0-20] Content pass: fill `profile`, `experience`, `ticker` from CV v2 (EN + ES); put `BiancaRomeroG_CV.pdf` in `/public` (owner provides final PDF; placeholder until then).
- [W0-21] README with GIF of the hero, docs complete, case-study integration guide for W1–W3 (`docs/case-study-format.md`).

### Phase 5 — polish
- [W0-22] Ingest the three real case studies when W1–W3 reach Gate 4; replace `in progress` badges with live links.
- [W0-23] Micro-interactions audit; 404 page with the canvas; `/es` copy review; keyboard-only run-through; Lighthouse >= 95 everywhere.
- [W0-24] Optional: `/now` page, `/uses` page, print stylesheet that renders the home as a one-page CV.

## 8. Definition of done (Gate 4 for W0)
Public URL loads in < 2 s, both locales, all sections animated and accessible, three project cards (two may still say `in progress`), CV download works, Lighthouse thresholds met, README with GIF, docs complete.
