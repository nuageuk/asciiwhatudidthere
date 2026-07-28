# asciiwhatudidthere

A single file, from scratch project that renders live video and 3D as ASCII text in the browser. Text, a spinning cube, or your own webcam, all pushed through the same pixel sampling pipeline.

A source (a hidden three.js scene or your webcam feed) is drawn to an off screen canvas each frame, sampled pixel by pixel, and mapped to a density ramp of ASCII characters (the ramp is ` .:-=+*#%@`, darkest to densest) to produce a live text mode render.

## Live demo

[nuageuk.github.io/asciiwhatudidthere](https://nuageuk.github.io/asciiwhatudidthere/)

## Running it

Open `index.html` directly in a browser, or serve the directory with any static file server:

```
npx serve .
```

Webcam mode requires HTTPS or `localhost`. Browsers block camera access over plain `file://`, so use the serve command above (or any local dev server) to test it.

## Hosting it

This is a fully static site. Everything, including the ML background removal, runs client side in the visitor's browser; nothing here needs a backend or database. It can be deployed to Render, GitHub Pages, Netlify, or any static host with no build step. three.js and MediaPipe both load from a CDN at runtime, so serving the file is all a host needs to do.

## Privacy

Everything happens locally in your browser. The webcam feed, the segmentation mask, and every rendered frame stay on your device and are never uploaded, streamed, or logged anywhere; there's no backend to send them to. The only network requests are the initial load of three.js and (only if you turn on background removal) the MediaPipe runtime and model, both from their respective CDNs. Screenshots and copied text are generated and saved directly in-browser too.

## Controls

**Resolution.** Slider (40 to 200 columns) controlling how many ASCII columns and rows the scene is rendered at, keeping a stable on screen footprint regardless of density. Lower values render faster and look chunkier; higher values are sharper but slower.

**Color.** Select between rendering modes. `mono` is plain white on black text and is the fastest, using native text layout. `grey` tints each character by its sampled luminance. `color` tints each character with the actual sampled pixel color.

**Chars.** The density ramp used to map brightness to characters, darkest to densest, left to right (default: `` .:-=+*#%@``). Pick a built-in preset (Default, Blocks, Circles, Braille) or just type into the field — your name or anything else — to remap the whole render onto it. Picking a preset fills the field with it, and editing the field away from a known preset flips the dropdown to "Custom". A space is always the darkest character, prepended automatically if you don't type one, so black areas stay blank instead of filling in with a visible glyph. No emoji preset is bundled: every character shares one fixed monospace cell, and color/proportional emoji glyphs don't downscale cleanly to that — they render wider than their cell and collide with neighbors, and are markedly slower to draw. The bundled presets use plain text-style Unicode symbols instead (not emoji-presentation characters), which don't have that problem. You can still type actual emoji into the field yourself if you don't mind the tradeoff.

**Webcam.** Toggle to switch the render source from the spinning cube to your live camera feed.

**Mirror** (icon, webcam mode only). Flips the feed horizontally for a selfie-style view. On by default. Mirroring happens at the pixel-sampling stage, not with a CSS flip, so ASCII glyphs stay upright and readable instead of rendering backwards.

**Remove Background.** Webcam mode only. Real time person segmentation via MediaPipe, so only you render; the background is treated as empty space.

**Pause** (icon). Freezes the render loop (cube rotation and ascii output) on the current frame. Everything else stays interactive while paused.

**Copy Text.** Copies the current ascii grid to the clipboard as plain text, in any color mode.

**Screenshot** (icon). Downloads the current frame as a PNG.

## Tech

Everything lives in `index.html`. three.js (loaded from a CDN) renders the 3D scene; a 2D canvas samples pixels from either that render or the live `<video>` feed. Mono mode writes the result as a `<pre>` of plain text when the active character ramp is plain ASCII, the cheapest option since it creates no per character DOM nodes and relies on the font's own monospace metrics. Grey and color modes always draw directly onto a visible `<canvas>` with `fillText()` per character, sized for the display's actual pixel density to stay crisp — and mono falls back to the same canvas path whenever the ramp contains non-ASCII characters (as with the Circles or Braille presets), since those aren't guaranteed to share Courier New's advance width and a native `<pre>` would drift out of alignment line by line once the browser substitutes a fallback font per glyph.

Background removal uses [MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe/solutions/vision/image_segmenter), a real ML dependency and not hand rolled like the rest of this, for per frame person segmentation. The raw mask is noisy frame to frame, so it's stabilized with temporal smoothing (an exponential moving average), hysteresis (separate thresholds for a cell turning "on" versus staying "on"), and a spatial despeckle pass (a 3x3 majority filter) before being used.

Webcam luminance is also smoothed per cell, a separate exponential moving average from the mask smoothing above, to cancel out ordinary camera sensor noise. Without it, individual characters flicker between adjacent ramp values every frame even when the camera is pointed at something completely static.

The sample grid is fixed 4:3, but most webcams are 16:9. Rather than stretch the raw frame to fit, the webcam feed is center-cropped to 4:3 before sampling (the same idea as CSS `object-fit: cover`). The segmentation mask is computed from the uncropped raw frame, so that same crop window is reused when sampling the mask, keeping it in registration with the cropped color image instead of drifting out of alignment.

## Limitations

No `.obj` or 3D model support yet.

Color mode is capped by real per character canvas draw cost. Very high resolution plus color or grey mode plus a busy, non black source will drop below 60fps; mono mode has no such ceiling.

## Roadmap

Things planned but not yet built:

* `.obj` model upload, parsed from scratch, rendered through the same ASCII pipeline as text and webcam.
* Revisiting color mode's performance ceiling with a real shared glyph atlas (one packed canvas, drawn via source rectangles) rather than one `fillText()` call per character; an earlier attempt at a naive per glyph cache made things slower, not faster, so this needs to be done properly or not at all.
* General mobile and small screen responsiveness, including making the control panel and text bigger on small screens.
* Further performance fixes and smoothing.