# CLAUDE.md — The Wellness Garden · Interactive Project Site

This file gives you (Claude Code) standing context for this project. Read it before each task.

## What we're building
A single, polished, mobile-responsive **web app** that presents an ultra-luxury wellness + grocer destination at **4425 N Central Expressway, Knox District, Dallas**. Three views, switchable via top tabs:

1. **Site Map** — an interactive top-down plan of the lot. Clickable zones open a panel with the zone's specs, photos/renders, and notes.
2. **3D Walkthrough** — a real-time 3D model of the building (loaded from a GLB the owner provides) with orbit/pan controls and clickable hotspots that surface renders + room info.
3. **Economics** — a live pro-forma calculator (inputs → revenue/cost breakdown → EBITDA, margin, payback). Formulas and defaults are in `SPEC.md`.

## Tech stack (locked — don't substitute without asking)
- **Vite + React + TypeScript**
- **Tailwind CSS** for styling
- **three.js + @react-three/fiber + @react-three/drei** for 3D
- Deploy target: **Vercel** (or Netlify)

## Design system (match this — it's the established brand look)
- Display/headings font: **Fraunces** (serif). Body font: **Hanken Grotesk**. Load from Google Fonts.
- Color tokens:
  - `--bone #F4EFE6` (bg), `--bone2 #EAE2D3`, `--ink #241F18` (text), `--ink2 #4A4338`
  - `--clay #B4583C` (accent), `--teal #3E6B66`, `--garden #9FB089`, `--sand #CBB893`
- Voice: calm, editorial, generous whitespace. Minimal hard borders, soft rounded cards (radius ~16px), subtle shadows. Avoid generic "AI dashboard" aesthetics.

## Assets (owner will place these)
- 3D model: `/public/models/site.glb` (the architect's model exported to glTF/GLB).
- Renders & photos: `/public/renders/*.jpg`. Reference them from hotspots and the site-map panels.
- If `site.glb` is missing, the 3D view must render a graceful placeholder (a labeled box + "Drop your model at /public/models/site.glb"), never a crash.

## Conventions
- Keep components small and in `src/components/`. Data lives in `src/data/` (zones, hotspots, econ defaults) so it's easy to edit.
- Everything mobile-first and responsive; the 3D canvas must work on touch.
- No `localStorage`/`sessionStorage` dependence for core function (fine to use for remembering the last tab).
- Currency formatting helper; tabular-nums for all figures.
- Accessible: keyboard-focusable zones/hotspots, alt text on images.

## Commands
- `npm install` · `npm run dev` (local) · `npm run build` · `npm run preview`

## Definition of done
All three views work, the GLB loads with at least 3 working hotspots, the economics calculator recalculates live and matches the formulas in `SPEC.md`, the site map has clickable zones, and the whole thing is deployed to a shareable URL.
