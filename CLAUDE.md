# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server (Vite)
npm run build     # Build to dist/ as a single self-contained HTML file
npm run preview   # Preview production build
```

No test runner is configured.

## Architecture

Single-file React app — all logic lives in `src/App.tsx`. There are no component subdirectories.

**Stack:** React 19 + TypeScript + Three.js + Tailwind CSS v4 + Vite. The build uses `vite-plugin-singlefile` to inline everything into one standalone `index.html` (no external assets).

**Core concept:** A 3D LED cube visualizer. A Three.js scene renders a cube where each of the 6 faces is an independent canvas-based "scene" (video feed, matrix rain, gradient, etc.). Lights (point/spot/LED) are positioned around the cube in 3D space.

**State model:**
- `faces: FaceConfig[]` — 6 cube faces, each with a `SceneType`, UV mapping rect, and per-scene params.
- `lights: LightConfig[]` — dynamic lights with position, color, intensity, and strobe settings.
- All Three.js objects are held in `useRef` (never in React state) to avoid re-render overhead during the animation loop.
- Refs mirror React state (`facesRef`, `lightsRef`, etc.) so the animation loop always reads fresh values without closure staleness.

**Scene rendering:** Each face renders to an offscreen `<canvas>` which becomes a `THREE.CanvasTexture`. The `camera` scene type uses a `THREE.VideoTexture` from `getUserMedia`. The animation loop (`requestAnimationFrame`) updates all canvases each frame based on their `SceneType`.

**Undo/Redo:** History is stored as JSON snapshots of `{ faces, lights, offFaceOpacity }`, debounced 400 ms. Max 50 entries. `Cmd+Z` / `Cmd+Shift+Z` / `Cmd+Y` are wired globally.

**Path alias:** `@/` maps to `src/`.

---

## Project Summary

### What is "Inside the Box"?

A 3D visualization tool for stage lighting design. It simulates an LED cube where each face can display different content (video, camera, gradients, etc.) while lights illuminate the scene from outside.

### Current Features (v3)

**Cube Faces:**
- 6 configurable faces with independent content
- Scene types: `camera` (webcam), `trailer` (video loop), `fileinput` (local video/image), `grid_img` (grid pattern), `gradient` (color wash), `win98` (retro glitch), `white` (solid color with RGB picker)
- Per-face resolution settings
- Preview thumbnails

**Lighting System:**
- 4 lights (expandable)
- Types: POINT, SPOT, LED
- Controls: color picker, intensity, strobe (with Hz)
- Rotation: X and Z axes (not tilt/pan)
- Position XYZ controls
- Text input for custom light names
- Visual helper meshes in 3D scene

**Rig Styles:**
- `pipe` - cylindrical pipe structure
- `truss` - truss/framework structure (horizontal + vertical bars)
- `columns` - vertical columns only (4 corners) + 4 stick-figure people (2 inside cube, 2 outside at 2m distance)

**Camera:**
- Virtual camera feed from webcam
- Auto-starts when a face is set to camera scene
- Resolution selection

**File Input:**
- Load local video (.mp4) or images (.png)
- Displays on cube faces

**UI:**
- Dark theme (Strudel aesthetic: near-black backgrounds, blue accents)
- Left panel: Face configuration + Sync all
- Right panel: Lights + Structure + Resolution
- Collapsible sections
- Color picker for white scene
- Speed slider for gradient/win98 scenes

### Recent Changes (by tomi)

1. Added color picker to "white" scene
2. Added RGB rotation controls to lights (X/Z instead of tilt/pan)
3. Added text input for light names
4. Changed active button styles to blue stroke (no white background)
5. Camera button: transparent with blue stroke when active, blinking white circle when recording
6. Added "columns" rig style with vertical bars only
7. Added 4 stick-figure people to columns rig (2 inside, 2 outside at 2m)
8. Removed fabric/tela person customization (people are automatic)
9. UI theme: dark blue-black (#0a0a0e) with blue accent (#4a8cff)

### Key Files

- `src/App.tsx` - Main application (single file, ~2200 lines)
- `src/index.css` - Global styles including animations
- `src/main.tsx` - React entry point
