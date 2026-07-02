# Journal Layout Planner — project conventions

A mobile-first web tool for planning journal/scrapbook page layouts. Full spec in [PRD.md](PRD.md).

## Architecture rules (hard constraints)

- **Single self-contained `index.html`** — all HTML, CSS, and JS in one file. Repo contents stay minimal: `index.html`, `manifest.json`, 1–2 app icons.
- **No build step.** No bundler, no transpiler, no npm. JS/React (if used) loads via CDN `<script>` tags only.
- **Zero backend, zero network calls after page load.** User photos and text never leave the device (hard privacy requirement). No analytics, no external asset fetches at runtime — placeholder photos are generated locally as SVG/canvas art.

## Target platform

- **iPhone Safari is the primary target** (current iOS), installable via "Add to Home Screen" (PWA manifest). Desktop browsers secondary.
- Touch-first: handles/targets ≥ 44 px, correct `touch-action` so element gestures don't fight page scroll, pinch pan/zoom on the canvas.
- Performance: 4 layout suggestions generated in < 1 s; 60 fps editing interactions.

## Visual style

- **Warm, light palette** — owner preference, explicitly no dark mode default. Calm, uncluttered UI. English UI (v1).

## Engine conventions

- Layout engine is **algorithmic, no AI/LLM calls of any kind**.
- Keep archetype definitions as **data (parameter objects)**, not hard-coded layouts — this is the tuning surface for aesthetics.
- Internal geometry works in **centimeters** (page dimensions, margins, element sizes per PRD §5–6); convert to pixels only at render time.
- Randomization is **seeded per suggestion** so Regenerate produces genuinely new, non-repeating variants.

## Build order (PRD §13)

1. Geometry model + archetype engine rendered as static thumbnails — get generation quality right first.
2. Touch editor (largest effort — budget iteration here).
3. Export (reuses the editor render path).
