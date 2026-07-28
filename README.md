# asciiwhatudidthere

A single file, from scratch project that renders live video and 3D as ASCII text in the browser. Text, a spinning cube, or your own webcam, all pushed through the same pixel sampling pipeline.

A source (a hidden three.js scene or your webcam feed) is drawn to an off screen canvas each frame, sampled pixel by pixel, and mapped to a density ramp of ASCII characters (the ramp is ` .:-=+*#%@`, darkest to densest) to produce a live text mode render.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static file server:

```
npx serve .
```

Webcam mode requires HTTPS or `localhost`. Browsers block camera access over plain `file://`, so use the serve command above (or any local dev server) to test it.

## Hosting it

This is a fully static site. Everything, including the ML background removal, runs client side in the visitor's browser; nothing here needs a backend or database. It can be deployed to Render, GitHub Pages, Netlify, or any static host with no build step. three.js and MediaPipe both load from a CDN at runtime, so serving the file is all a host needs to do.

## Controls

**Resolution.** Slider (40 to 200 columns) controlling how many ASCII columns and rows the scene is rendered at, keeping a stable on screen footprint regardless of density. Lower values render faster and look chunkier; higher values are sharper but slower.

**Color.** Select between rendering modes. `mono` is plain white on black text and is the fastest, using native text layout. `grey` tints each character by its sampled luminance. `color` tints each character with the actual sampled pixel color.

**Webcam.** Toggle to switch the render source from the spinning cube to your live camera feed.

**Remove Background.** Webcam mode only. Real time person segmentation via MediaPipe, so only you render; the background is treated as empty space.

## Tech

Everything lives in `index.html`. three.js (loaded from a CDN) renders the 3D scene; a 2D canvas samples pixels from either that render or the live `<video>` feed. Mono mode writes the result as a `<pre>` of plain text, the cheapest option since it creates no per character DOM nodes. Grey and color modes draw directly onto a visible `<canvas>` with `fillText()` per character, sized for the display's actual pixel density to stay crisp.

Background removal uses [MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe/solutions/vision/image_segmenter), a real ML dependency and not hand rolled like the rest of this, for per frame person segmentation. The raw mask is noisy frame to frame, so it's stabilized with temporal smoothing (an exponential moving average), hysteresis (separate thresholds for a cell turning "on" versus staying "on"), and a spatial despeckle pass (a 3x3 majority filter) before being used.

Webcam luminance is also smoothed per cell, a separate exponential moving average from the mask smoothing above, to cancel out ordinary camera sensor noise. Without it, individual characters flicker between adjacent ramp values every frame even when the camera is pointed at something completely static.

## Limitations

No `.obj` or 3D model support yet.

Color mode is capped by real per character canvas draw cost. Very high resolution plus color or grey mode plus a busy, non black source will drop below 60fps; mono mode has no such ceiling.

## Roadmap

Things planned but not yet built:

* `.obj` model upload, parsed from scratch, rendered through the same ASCII pipeline as text and webcam.
* Webcam mirroring, since a true (unmirrored) camera view feels backwards for a selfie style feed.
* Fixing the render target's forced 4:3 aspect against most webcams' native 16:9, which currently means some stretching or letterboxing on the sampled feed.
* A camera selection control for anyone with more than one available device.
* Exporting the current frame as an image or a plain text file.
* Revisiting color mode's performance ceiling with a real shared glyph atlas (one packed canvas, drawn via source rectangles) rather than one `fillText()` call per character; an earlier attempt at a naive per glyph cache made things slower, not faster, so this needs to be done properly or not at all.
* General mobile and small screen responsiveness.
* Custom text mapping.
* Further performance fixes and smoothing.
* Different character maps (for example, emoji).