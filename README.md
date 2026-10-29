# Projection Mapper

A free, self-contained projection mapping web app. No build step, no CDN
dependencies, no account, no ads — just open `index.html` in a browser and it
works offline. It is designed for a small Android projector (tested with a
Tecaki in mind), used in a single window on the projector itself.

## Use it

1. Open the app (locally: open `index.html`; hosted: the GitHub Pages URL
   for this repo).
2. The first surface starts on the built-in **test pattern**. Drag its four
   orange corner handles until the pattern lines up with the real surface
   you want to project onto.
3. Switch that surface's **Source** to an image or video and upload a local
   file (video loops). Or pick a solid colour.
4. Add more surfaces with **+ Add surface**. Surfaces render in list order
   (later surfaces draw on top); use the ↑ / ↓ buttons to reorder.
5. Click **Output mode** (or press **F**) to hide all UI and show only the
   black canvas. Press **F** again, or the small ✕ button, to return to the
   editor. Use the **Fullscreen** button for a true full-screen projection.
6. Use **Save project** / **Load project** to keep your corner layouts.

## How the warp works

Each surface is a unit square texture warped by a true 3×3 perspective
homography, solved in JavaScript from the four dragged corners and applied
in a vanilla WebGL vertex shader. The shader outputs
`gl_Position = (H·p).xy` with `w = (H·p).z`, so the GPU's perspective
division does the projective interpolation — dragging a corner warps the
media, it does not merely scale or skew it (an affine-only transform would
look wrong on a keystoned surface and would show a seam on the quad
diagonal).

## Important: media files are NOT saved in a project

A saved project JSON stores, per surface:

- name, source type, solid colour
- corner positions (as fractions of the canvas, so a project survives a
  different screen size)
- opacity
- surface order
- the media file's **name** only

It does **not** store the image or video bytes. After loading a project,
re-upload each image/video file (the surface list flags surfaces that still
need media, and they show the test pattern until you do). This keeps the
project file tiny and avoids any copyrighted media being baked into a repo.

## Projector notes

- Run the app on the projector itself in Output mode, or on a laptop
  connected to the projector by HDMI. No second output window is used, so
  there is no window-syncing to break.
- Keep the projector as square-on to the surface as you can; extreme angles
  cost resolution.
- Video must be a format the projector's browser can play (H.264 MP4 is the
  safest bet). Videos play muted and looped, because browsers block
  autoplay with sound.
- If a projector browser is old or weak on WebGL, use a laptop over HDMI
  instead — the app itself is the same page either way.

## Limitations

- Flat quad surfaces only — no curved-surface mesh warping, no
  multi-projector edge blending, no 3D calibration.
- No audio, no scene timeline, no live effects. One texture per surface.
- Corner positions are fractions of the canvas, so if you change the
  projector's aspect ratio a saved layout will stretch with it.
- Degenerate quads (corners crossed, collinear, or coincident) are skipped
  in rendering until you drag them back into a valid quad.
