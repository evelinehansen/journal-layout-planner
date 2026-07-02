# PRD: Journal Layout Planner

**Version:** 1.0 (planning complete, ready for build)
**Date:** 2026-07-02
**Owner:** Eveline
**Status:** Approved for build — v1 scope

---

## 1. Summary

A mobile-first web tool for planning journal/scrapbook page layouts. The user specifies notebook size, page mode, orientation, and a set of elements (photos, sketches, washi tape, text, emojis). The tool generates four alternative layout suggestions using an algorithmic layout engine. The user picks one, refines it with full touch editing (drag, resize, rotate, z-order, add/delete), locks it, and exports a labeled PNG reference image to recreate by hand in the physical notebook.

**Output is reference-only.** The exported image is a proportional sketch with dimension labels — not a true-to-scale template. Photos are printed at home and trimmable, so all element sizes are soft constraints.

---

## 2. Goals and non-goals

### Goals
- Break "blank page paralysis" by generating varied, aesthetically intentional layout alternatives in seconds.
- Work well on iPhone (Safari), installable to home screen.
- Complete workflow in one sitting: input → generate → edit → lock → export.
- Zero backend, zero accounts, zero data leaving the device.

### Non-goals (v1)
- No saving/resuming works-in-progress.
- No true-to-scale printable templates.
- No AI/LLM API calls of any kind.
- No sharing, collaboration, or gallery features.
- No user-defined custom notebook sizes (fixed list of 4).

---

## 3. Platform, hosting, and architecture

| Decision | Choice | Rationale |
|---|---|---|
| Hosting | **GitHub Pages** (public repo, static site) | Free, already in owner's workflow, plain URL usable in Safari |
| App format | **Single self-contained HTML file** (`index.html`), JS/React via CDN, no build step | One-file maintenance, matches existing GitHub Pages usage |
| PWA | `manifest.json` + icon so "Add to Home Screen" (*Legg til på Hjem-skjerm*) gives full-screen app mode | Removes Safari chrome — important for a drag-heavy canvas |
| Backend | **None.** Fully client-side | No persistence needed; engine is algorithmic |
| Photo handling | Browser **File API** only: photos are read into memory, composited locally, exported locally | **Hard requirement: user photos never leave the device.** No upload, no analytics, no external calls |
| UI language | English (v1) | Simplicity; Norwegian localization is a possible v2 item |

Repo contents: `index.html`, `manifest.json`, 1–2 app icons. Nothing else.

---

## 4. Inputs

### 4.1 Notebook size (select one)

| Size | Page dimensions (portrait) |
|---|---|
| A5 | 14.8 × 21.0 cm |
| B6 | 12.5 × 17.6 cm |
| A6 | 10.5 × 14.8 cm |
| TN (Travelers Company regular) | 11.0 × 21.0 cm |

### 4.2 Page mode (select one)
1. **Single page**
2. **Double-page spread** — content may span both pages (see gutter rules, §5.3)
3. **Two independent pages** — e.g. front/back of a leaf. Implemented as two sequential single-page designs within one session (tabbed "Page 1 / Page 2"); each page gets its own suggestions and editing; export produces two PNGs (or one combined image with both pages side by side — build choice, either acceptable).

### 4.3 Orientation (select one)
- **Vertical** (portrait): dimensions as listed above.
- **Horizontal** (landscape): width/height swapped (e.g. horizontal TN = 21.0 × 11.0 cm).

### 4.4 Double-page spread geometry (rule)
The spine is fixed in the physical notebook. A spread doubles the page dimension **perpendicular to the spine**:

- **Vertical layout:** pages sit side by side → spread = 2 × width, same height. Gutter is a **vertical** line at center.
  - Example: TN vertical spread = 22.0 × 21.0 cm.
- **Horizontal layout:** notebook is rotated 90°, pages stack → spread = same width, 2 × height. Gutter is a **horizontal** line at center.
  - Example: TN horizontal spread = 21.0 × 22.0 cm; A5 horizontal spread = 21.0 × 29.6 cm.

### 4.5 Density (select one)
Replaces "utilize as much space as possible." Controls target coverage of usable area:

| Density | Approx. target coverage |
|---|---|
| Airy | ≤ ~45% |
| **Balanced (default)** | ≤ ~65% |
| Packed | ≤ ~85% |

### 4.6 Elements (type + quantity)
User picks element types **with counts** (stepper controls, e.g. "Landscape photo × 2"). See §6 for the element model.

### 4.7 Text input
For each text element the user either:
- Types/pastes their own text, or
- Taps "Placeholder text" → app inserts a random snippet from the bundled placeholder library (§6.6).

### 4.8 Photo input
For each photo element the user either:
- Uploads a photo from the phone (File API / photo picker), or
- Selects one of three bundled standard placeholders: **Landscape**, **City**, **Abstract** (generated locally as SVG/canvas art — no external assets, keeps the app single-file).

---

## 5. Layout rules (geometry)

### 5.1 Margins
Minimum **0.5 cm margin on all four outer edges** of the page/spread. No element may cross the outer margin.

### 5.2 Usable area
Page dimensions minus margins. Example: A6 portrait usable area = 9.5 × 13.8 cm.

### 5.3 Gutter dead zone (spreads only)
- **0.8 cm on each side of the spine.**
- The generator does **not place** text, small elements, emojis, or photo focal areas in the dead zone.
- **Washi tape strips and large photos may deliberately cross the spine** (this is an intentional design move, used by some archetypes).
- Manual editing may override gutter rules — the user is in charge; the zone is rendered as a subtle visual hint on the canvas.

### 5.4 Element scaling (soft constraints)
All default element sizes may be scaled by the engine within **60–130%** of default to make a composition work. Actual resulting dimensions are shown in export labels (§9). If even minimum scaling cannot fit the requested elements at the chosen density, show a **feasibility warning** before generation ("These elements won't fit comfortably on A6 at Airy density — remove elements, increase density, or choose a larger page") with the option to proceed anyway (Packed-style best effort).

---

## 6. Element model

| # | Element | Default size | Notes |
|---|---|---|---|
| 1 | Landscape photo | 10.0 × 7.5 cm | User photo or standard placeholder |
| 2 | Portrait photo | 7.5 × 10.0 cm | User photo or standard placeholder |
| 3 | Sketch placeholder, small | 4.0 × 4.0 cm | Rendered as a framed doodle-style placeholder |
| 4 | Sketch placeholder, large | 10.0 × 6.0 cm | Same style, larger |
| 5 | Washi tape | 1.5 cm wide, variable length | See §6.5 — placed by *role*, not as a bare rectangle |
| 6 | Headline text | 1.5 cm tall, width from content | Hand-lettering-style display font |
| 7 | Paragraph text | User picks size 8 / 10 / 12 (≈ line heights 3.5 / 4.2 / 5.0 mm) | Block height computed from text length at chosen size and assigned width |
| 8 | Emoji / simple symbol | ~1–2 cm | Native emoji characters; user picks which |
| 9 | Keyword highlights | Auto-derived | See §6.4 |

### 6.1 Photos
- Uploaded photos are cropped-to-fit their assigned frame (center crop by default; crop position adjustable in editing is a nice-to-have, not required for v1).
- Aspect ratio of the *frame* follows the element type; the engine may scale the frame per §5.4.

### 6.2 Sketch placeholders
Render as visually pleasant "sketch goes here" blocks (e.g. light border, faint doodle icon, label). They mark reserved space for hand drawing.

### 6.3 Text
- Headline: single line (wraps to two if needed), rendered in a display font suggesting hand lettering.
- Paragraph: rendered in a handwriting-adjacent font at the chosen size. If user text is long, block grows; engine accounts for computed height during layout.

### 6.4 Keyword highlights (no AI)
Naive client-side extraction from the user's paragraph text:
- Strip a small stop-word list (English + Norwegian).
- Prefer longer words and capitalized words; sample 3–5 candidates with randomness.
- Render as marker-highlight chips or underlined callouts placed as small decorative elements.
- Purpose is **visualizing where highlights would sit**, not smart summarization. (AI-quality extraction = v2 candidate.)
- Only available when at least one paragraph element with real user text exists; otherwise greyed out.

### 6.5 Washi tape roles
Washi is placed according to one of four roles, chosen by the archetype/engine (or by the user when adding manually):

1. **Border strip** — full-width or full-height run along a page edge (inside margin).
2. **Photo anchor** — short strips across photo corners or edges, as if taping the photo down (rendered overlapping the photo, above it in z-order).
3. **Divider** — strip separating two content zones.
4. **Diagonal accent** — angled strip (±5–30°) as a decorative element; allowed to cross the spine on spreads.

Washi renders semi-transparent with a simple repeating pattern (a few built-in pattern styles, randomized).

### 6.6 Placeholder text library
Bundled snippets, two sources — **no copyrighted text**:
1. Public-domain sci-fi classics: H.G. Wells (*The War of the Worlds*, *The Time Machine*), Jules Verne (*Twenty Thousand Leagues Under the Seas*), and similar pre-1928 works.
2. Original absurdist lines written for this app in the spirit of comic sci-fi (towels, improbability, bureaucratic aliens, poetry-writing robots) — original prose, quoting no one.

Random selection per insertion; tapping again reshuffles.

---

## 7. Layout engine (algorithmic, no AI)

### 7.1 Approach
A library of **layout archetypes**, each a parameterized placement strategy. One generation cycle = pick 4 distinct archetypes (weighted by suitability for the element mix and page shape), apply randomized variation within each, validate, render as thumbnails.

### 7.2 Archetypes (v1 set, minimum 6)
1. **Grid** — aligned rows/columns, even gaps.
2. **Focal hero** — one dominant element (largest photo or sketch) at 1/3 or golden-ratio position; supporting elements orbit it.
3. **Diagonal flow** — elements step diagonally across the page/spread; washi diagonal accents fit naturally.
4. **Columns (magazine)** — 2–3 vertical columns; text fills one, visuals the others.
5. **Bands** — horizontal zones (e.g. photo band / text band / decoration band); suits horizontal layouts and TN especially.
6. **Scatter collage** — slightly rotated (±2–8°) overlapping-adjacent placement, washi photo-anchors, organic feel.

### 7.3 Generation principles
- **One focal element per page**; others support it.
- Respect margins (§5.1), gutter rules (§5.3), density target (§4.5), scale bounds (§5.4).
- No element fully hidden behind another; deliberate partial overlap allowed only where the archetype calls for it (scatter, photo anchors).
- Alignment: non-scatter archetypes snap edges/centers to a light internal grid so results look intentional.
- Randomization uses a seed per suggestion, so **Regenerate** produces 4 genuinely new variants (track recent seeds/archetype-variant combos to avoid near-repeats within a session).

### 7.4 Flow
1. User confirms inputs → feasibility check (§5.4).
2. Engine renders **4 suggestion thumbnails**.
3. User taps **Regenerate** for 4 new ones (unlimited), or taps a suggestion to enter editing.

---

## 8. Editing (the largest build effort — budget iterations here)

Touch-first canvas editor, tested against iPhone Safari pointer events.

| Capability | Detail |
|---|---|
| Select | Tap an element → selection outline + handles. Tap empty canvas → deselect |
| Move | Drag selected element; light snap-to-alignment guides (toggleable) |
| Resize | Corner handle drag; aspect ratio locked for photos/sketches, free for washi length and text blocks |
| Rotate | Dedicated rotate handle above selection; free rotation, soft snap at 0°/±5°/±10°/90° |
| Z-order | "Forward / Back" buttons on the selection toolbar (photo tucked under washi etc.) |
| Add | "+" menu to add any element type post-generation (including choosing washi role) |
| Delete | Delete button on selection toolbar |
| Duplicate | Nice-to-have, not required v1 |
| Lock | Lock toggle freezes the entire canvas (no selection/edit possible) and reveals the Export button. Unlock returns to editing |

Ergonomics requirements:
- All handles/targets ≥ 44 px effective touch size.
- Canvas pan/zoom (pinch) so small elements are editable on a phone screen; element gestures must not fight page-scroll (use `touch-action` correctly).
- Undo (single-level minimum; multi-level preferred).

---

## 9. Export

- **Trigger:** "Export PNG" button, visible when layout is locked.
- **Rendering:** canvas rendered to PNG at **2× resolution**.
- **Contents:**
  - Thin page outline; **spine line** on spreads; margins/gutter *not* drawn (clean output), unless "show guides" toggle is on.
  - **Dimension labels** on each element, e.g. "8.5 × 6.4 cm" — **toggleable**, default on (this is the recreate-by-hand aid).
  - Optional **title and date stamp** in a corner (user-editable text field, default = today's date; toggleable).
- **Delivery:** standard browser download / iOS share sheet (user chooses destination there). The app does not and cannot pick a save location itself.
- Two-independent-pages mode exports both pages (two files or one combined — build choice).

---

## 10. Non-functional requirements

- **Target device:** iPhone Safari (current iOS). Must also be usable on desktop browsers, but phone is primary.
- **Privacy (hard requirement):** no network calls after page load; user photos and text never leave the device. No analytics.
- **Performance:** generation of 4 suggestions < 1 s; editing interactions at 60 fps on a modern iPhone.
- **Visual style of the app itself:** warm, light palette (owner preference — explicitly **no dark mode default**); calm, uncluttered UI.
- **Offline:** should work offline once loaded (bonus: simple service worker for full offline PWA; not required v1).
- **Dependencies:** CDN-loaded only; app must remain a single HTML file plus manifest/icons.

---

## 11. Acceptance criteria (v1)

1. On iPhone Safari, user can select A5/B6/A6/TN, single/double/two-independent, vertical/horizontal, density, and elements with counts.
2. TN vertical spread canvas measures 22 × 21 cm proportionally; TN horizontal spread 21 × 22 cm; gutter orientation matches (§4.4).
3. Requesting a 10 cm landscape photo on A6 produces a scaled-down photo with correct labeled dimensions — never an element crossing the outer margin.
4. Four visibly distinct suggestions per generation; Regenerate yields four new ones not repeating the previous set.
5. All six editing capabilities (move, resize, rotate, z-order, add, delete) work by touch; lock disables editing and enables export.
6. Exported PNG shows page outline, spine line (spreads), and correct per-element dimension labels; labels and title/date toggles work.
7. Placeholder text never includes copyrighted passages; photo placeholders render without any network fetch.
8. With network disconnected after load, the full flow (generate → edit → export) still works with placeholder assets.
9. Feasibility warning appears for impossible element/page/density combinations and allows best-effort proceed.

---

## 12. Out of scope / v2 candidates

- AI-quality keyword/highlight extraction (would require an API — revisit only if genuinely wanted).
- Saving/resuming projects; a personal archetype/preset library ("my usual weekly spread").
- Crop-position adjustment inside photo frames; sticker/stamp element library; custom washi widths.
- True-to-scale PDF export for printing cut guides.
- Norwegian UI localization.
- Sharing layouts with other journaling folks (would change hosting/privacy posture — decide deliberately if ever).

---

## 13. Build notes for the next session

- Start with the **geometry model + archetype engine** rendered as static thumbnails; get generation quality right before touching editing.
- Then build the editor (expect most iteration here — pointer events, pinch zoom, handle ergonomics).
- Export last; it reuses the editor's render path.
- Keep archetype definitions as data (parameter objects), not hard-coded layouts — easiest place to tune aesthetics later.
