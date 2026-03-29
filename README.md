# Breath + Meditation Web App (1bref)

This repo folder contains two related single-file web experiences:

1) A breathing technique visualizer (multi-ring hold-to-breathe + focus list).
2) A meditation timer/guide page that matches the same visual language and is linked from the breathing page.

The key constraint: `1bref_v18.html` is treated as the original source and is generally not edited; most changes are done via the wrapper `index.html` which loads and patches it at runtime.

## Run Locally

From this folder:

```bash
python -m http.server 8000
```

Open:

- `http://localhost:8000/` (breathing app via wrapper `index.html`)
- `http://localhost:8000/meditation_mobile_10_minimal.html` (meditation page)

## Files You Actually Work In

Breathing:

- `1bref_v18.html`
  - The original breathing app HTML/CSS/JS in one file.
  - Contains `TECHNIQUES` array, ring animation (`updateRing`), draw loop, focus list UI.
  - Prefer NOT to edit unless explicitly asked.

- `index.html`
  - Wrapper entrypoint used for local/dev and GitHub Pages.
  - Fetches `1bref_v18.html`, injects its `<style>` + `<body>` + `<script>`, then applies runtime patches.
  - This is where most breathing-side changes currently live.

Meditation:

- `meditation_mobile_10_minimal.html`
  - Standalone meditation page.
  - Matches the breathing look (dark background, thin UI, dock).
  - Single white breathing circle (4s inhale / 6s exhale internally).
  - Has a bottom dock to navigate meditations + a picker overlay.
  - Full instructions are always visible in the page (and color coded).

Assets:

- `latty.png`
  - Bottom-right “portal” image on the breathing page.
  - Clicking it navigates to `meditation_mobile_10_minimal.html`.

Deployment:

- `.github/workflows/deploy-pages.yml` (GitHub Pages deploy)
- `.nojekyll` (disable Jekyll)

## How The Breathing Wrapper Works (`index.html`)

`index.html`:

- Loads `1bref_v18.html` via `fetch()`.
- Extracts and injects:
  - `<style>` into `<head>`
  - HTML body into `<body>`
  - `<script>` back into the page (optionally after string patches)

### Current Breathing-Side Patches (String / Runtime)

Most of these are implemented by string-replacing content inside the injected `1bref_v18.html` script.

- Technique list edits
  - Removes some techniques and merges names (historically used to reduce duplicates).
  - Adds additional techniques by inserting new objects into `TECHNIQUES`.
  - Restores expanded text blocks for certain techniques (notably `Nadi Shodhana`).

- Color scheme
  - Replaces `techColor()` with an HSL-based spectrum so similar inhale lengths do not all look the same.
  - Keeps long-inhale end anchored toward purple.

- Focus list layout
  - Replaces the short/medium/long bucket append logic.
  - Builds a single grid sorted by inhale length (then cycle/name).

- Home screen curation
  - Alters ring `visible` defaults so the home view is intentionally calm.
  - Currently allowlist-based: only specific techniques show in the default multi-ring home view.
  - Focus mode still shows any technique you select.

- Mobile sizing
  - Injects `viewport-fit=cover`, safe-area paddings, `--app-vh` logic.

- Release ring effect
  - Adds an overlay white “outbreath” ring on release.
  - Supports multiple contracting rings simultaneously.
  - Does not modify the main ring simulation.

- Fonts
  - Loads Google Fonts and overrides typography:
    - UI: IBM Plex Sans
    - Display: Fraunces

If you change any string-replace patch, be careful: these patches are brittle (they depend on exact substrings from `1bref_v18.html`).

## Meditation Page Notes (`meditation_mobile_10_minimal.html`)

Key pieces:

- `MEDITATIONS` array
  - `stage` and `name`: used by the dock title + picker.
  - `accent`: drives the dock dot and overall accent variables.
  - Other fields (`subtitle`, `howto`, etc.) still exist but the main visible instructions are sourced from `RAW_TEXT` and normalized.

- Instructions rendering
  - `RAW_TEXT` holds the verbatim meditation instruction blocks.
  - `buildInstructionsHtml()` renders:
    - Aim (colored with meditation accent)
    - Detailed instructions
    - Reflection
      - Good practice: dark green label
      - Bad practice: dark red label
      - Advance when: purple label

- Audio cues
  - Start tone and finish tone are synthetic WebAudio clicks.

## Next Work: Meditation Accent Colors (Progression)

To give each meditation a unique color with a sense of progression (e.g. red -> purple -> white “chakra-like”):

- Edit the `accent` values in `MEDITATIONS` in `meditation_mobile_10_minimal.html`.
- Keep contrast against `#07070e` (avoid very dark accents).
- Suggestion: assign a monotonic hue progression by index and gently increase lightness for later stages.

If you want a strict progression, implement a small helper like `accentForIndex(i)` and generate `accent` dynamically, but note this file is intentionally standalone (no build step).

## Quick Debug Checklist

- If the breathing page shows only “LOADING”, the wrapper script likely has a syntax error.
  - Check `index.html` first (it injects patched JS as a string).
  - A single stray newline inside a JS string can break the whole wrapper.

- If the focus list/grid looks empty or broken:
  - The build-list patch in `index.html` might not match the original `1bref_v18.html` snippet anymore.

- If audio seems missing:
  - Browser may require a user gesture; in focus mode it should work once you interact.
