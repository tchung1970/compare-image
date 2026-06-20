# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file client-side image comparison tool. The entire app lives in `index.html` — HTML, CSS, and JavaScript are all inline. No build step, no dependencies, no framework.

## Development

Open `index.html` directly in a browser. No server required (uses `URL.createObjectURL` for local file handling).

## Architecture

- **Single-file app**: All HTML, CSS (`<style>` block), and JS (`<script>` block) live in `index.html`. Keep it that way — do not extract into separate files.
- **Two screens**: Upload screen (`#uploadScreen`) and compare mode (header + compare view). Toggled by showing/hiding DOM elements in `startCompare()` and `backBtn` click handler.
- **Upload screen**: Drag-and-drop or file picker for two images (A = left, B = right). Drop on a specific box or anywhere on the screen (auto-fills A then B). Upload boxes default to 250×260px and dynamically resize via `normalizeUploadBoxes()` to match image aspect ratio — landscape (16:9) expands to maxW 480, portrait (9:16) expands to maxH 400. Both boxes always share the same dimensions (widest aspect ratio wins); all boxes resize together even if only one has an image.
- **Split view** (default): Both images overlaid with a draggable vertical slider controlling a CSS `clip-path` to reveal left/right. Image A is clipped on top of Image B.
- **Side-by-side view**: Two independent panels (`flex: 1`) separated by a divider. `normalizeSideBySide()` computes the contain-rendered height for each image and constrains both to the smaller height, so images with different resolutions appear at the same visual size.
- **Four `<img>` elements**: `imgA`/`imgB` for split mode, `imgA2`/`imgB2` for side-by-side mode. All four share the same object URLs.
- **Header layout**: `.header-left` groups the logo + view-toggle (Split View / Side by Side) on the left. `.header-right` holds the zoom controls, Capture, Reset, and Close on the right.
- **Zoom/Pan**: Zoom is set via the header controls only (no scroll-wheel zoom); click-drag to pan when zoomed >1x. Zoom controls (`.zoom-controls`) are five preset buttons (`.zoom-preset` with `data-zoom` of 1–5 = 100%–500%) plus a custom `.zoom-input` (`#zoomInput`) that takes a typed percentage (Enter/blur applies, clamped 0.5x–10x; Esc reverts). `updateZoomUI()` highlights the matching preset and syncs the input (unless it's focused). Transform applied to all four image elements via `updateTransform()`, which calls `updateZoomUI()`. All images use `translate(-50%, -50%)` centering.
- **Auto-align** (split view): `computeAutoAlign()` estimates a global similarity transform (scale + translation) that registers image B onto image A, applied as an extra CSS transform on `imgB` only (composed before zoom so it scales with zoom on screen; side-by-side B is left to `normalizeSideBySide()`). Method: render both images to equal-height grayscale buffers (`grayBuffer`, H=900), measure a displacement field over a 7×4 grid via local block matching (`bestShift`, coarse-to-fine NCC, confidence-thresholded at 0.6), then least-squares fit `dx=(s-1)(x-cx)+Tx` to recover scale+translation. Runs once after load (deferred via setTimeout so the view paints first) and is re-applied on view switch/resize. Faithful upscales resolve to scale 1.0 (no-op); creative re-generations get a small scale/offset correction. State: `autoAlign` (toggle, default on — `alignBtn`), `alignS`, `alignTxFrac`/`alignTyFrac` (translation as fraction of displayed height). It cannot fix non-rigid content differences (re-drawn detail) — only the global similarity. No dependencies. The Capture export reflects alignment automatically (it reads `imgB`'s live rect).
- **Capture**: `captureComparison()` (Capture button in header) renders the current comparison to a PNG via a `<canvas>` (no libraries). Reads each image's `getBoundingClientRect()` so the on-screen zoom/pan are reproduced; in split mode it draws `imgB` then `imgA` clipped to the left `sliderPos` fraction. Downloads as `compare-<mode>.png` (DPR-scaled). Object URLs only (same-origin blobs), so the canvas is never tainted.
- **Info bar**: Shows file name, extension, and native resolution as `filename (EXT WxH)` for each image. Each tag takes 50% width, centered within its half to align with the split/side-by-side panels below.

## Key Globals

All state is in module-level variables: `fileAObj`/`fileBObj` (selected files), `sliderPos` (0–1), `mode` ('split'|'side'), `zoom`, `panX`/`panY`. No classes or modules — everything is functions + event listeners.
