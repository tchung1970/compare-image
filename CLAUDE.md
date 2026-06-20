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
- **Split view**: Both images overlaid with a draggable vertical slider controlling a CSS `clip-path` to reveal left/right. Image A is clipped on top of Image B.
- **Side-by-side view** (default): Two independent panels (`flex: 1`) separated by a divider. `normalizeSideBySide()` computes the contain-rendered height for each image and constrains both to the smaller height, so images with different resolutions appear at the same visual size.
- **Four `<img>` elements**: `imgA`/`imgB` for split mode, `imgA2`/`imgB2` for side-by-side mode. All four share the same object URLs.
- **Zoom/Pan**: Scroll wheel for zoom (0.5x–10x), click-drag to pan when zoomed >1x. The zoom badge is a `contenteditable` `<span>` (not a button): single-click cycles presets 100→200→300→400→500→100 (debounced 220ms via `cycleZoom()`), double-click enters edit mode (`startZoomEdit()`) to type a custom percentage (clamped 0.5x–10x on commit). `editingZoom` guards `updateTransform()` so it won't overwrite the field mid-edit. Transform applied to all four image elements via `updateTransform()`. All images use `translate(-50%, -50%)` centering.
- **Info bar**: Shows file name, extension, and native resolution as `filename (EXT WxH)` for each image. Each tag takes 50% width, centered within its half to align with the split/side-by-side panels below.

## Key Globals

All state is in module-level variables: `fileAObj`/`fileBObj` (selected files), `sliderPos` (0–1), `mode` ('split'|'side'), `zoom`, `panX`/`panY`. No classes or modules — everything is functions + event listeners.
