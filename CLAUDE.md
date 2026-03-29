# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file client-side image comparison tool. The entire app lives in `index.html` — HTML, CSS, and JavaScript are all inline. No build step, no dependencies, no framework.

## Development

Open `index.html` directly in a browser. No server required (uses `URL.createObjectURL` for local file handling).

## Architecture

- **Single-file app**: All HTML, CSS (`<style>` block), and JS (`<script>` block) live in `index.html`. Keep it that way — do not extract into separate files.
- **Two screens**: Upload screen (`#uploadScreen`) and compare mode (header + compare view). Toggled by showing/hiding DOM elements in `startCompare()` and `backBtn` click handler.
- **Upload screen**: Drag-and-drop or file picker for two images (A = left, B = right). Drop on a specific box or anywhere on the screen (auto-fills A then B).
- **Split view** (default): Both images overlaid with a draggable vertical slider controlling a CSS `clip-path` to reveal left/right. Image A is clipped on top of Image B.
- **Side-by-side view**: Two independent panels (`flex: 1`) separated by a divider. Images use `object-fit: contain` with absolute positioning so different resolutions are normalized to the same visual size.
- **Four `<img>` elements**: `imgA`/`imgB` for split mode, `imgA2`/`imgB2` for side-by-side mode. All four share the same object URLs.
- **Zoom/Pan**: Scroll wheel for zoom (0.5x–10x), click-drag to pan when zoomed >1x. Transform applied to all four image elements via `updateTransform()`. Split-mode images use `translate(-50%, -50%)` centering; side-by-side images don't.
- **Info bar**: Shows file name, extension, and native resolution as `filename (EXT WxH)` for each image. Each tag takes 50% width, centered within its half to align with the split/side-by-side panels below.

## Key Globals

All state is in module-level variables: `fileAObj`/`fileBObj` (selected files), `sliderPos` (0–1), `mode` ('split'|'side'), `zoom`, `panX`/`panY`. No classes or modules — everything is functions + event listeners.
