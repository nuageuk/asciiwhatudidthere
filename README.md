# asciiwhatudidthere

A single-file three.js proof of concept that renders a hidden 3D scene (a rotating cube) as live ASCII text in the browser.

The scene is rendered off-screen to a small WebGL canvas, sampled pixel-by-pixel each frame, and mapped to a density ramp of ASCII characters to produce a text-mode render of the 3D scene.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static file server:

```
npx serve .
```

## Controls

- **Resolution** — slider (40–200 columns) that controls how many ASCII columns/rows the scene is rendered at, keeping a ~4:3 aspect ratio. Lower values render faster and look chunkier; higher values are sharper but slower.
- **Color** — select between rendering modes:
  - `mono` — plain white-on-black text (fastest).
  - `grey` — each character is tinted by its sampled luminance.
  - `color` — each character is tinted with the actual sampled pixel color.

## Tech

Everything lives in `index.html`: three.js (loaded from a CDN) renders the scene to a WebGL canvas, a 2D canvas samples its pixels, and the result is written out as a `<pre>` of plain ASCII text in mono mode, or drawn directly onto a visible `<canvas>` with `fillText()` per character in grey/color mode (avoids creating thousands of DOM nodes per frame).
