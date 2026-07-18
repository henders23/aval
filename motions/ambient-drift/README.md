# ambient-drift

A single seamless looping ambient scene with transparency (packed alpha),
authored for AVAL. 512×512, 24 fps, 72-frame (~3 s) loop.

## Files

- `motion.json` — the AVAL project (canvas, alpha, source, loop unit, codec ladder).
- `FRAME-BRIEF.md` — the exact spec for generating the source PNG frames with an AI.
- `frames/` — drop `frame-0000.png` … `frame-0071.png` here (RGBA, 512×512).

## Workflow

1. **Generate frames** per `FRAME-BRIEF.md` into `frames/`. Verify count, size,
   and that they carry a real alpha channel.

2. **Compile** the codec bundle (needs FFmpeg/FFprobe with libaom-av1, libvpx-vp9,
   libx265, libx264 on your PATH):

   ```sh
   npx avl compile motions/ambient-drift/motion.json --out dist/ambient-drift
   ```

   Produces `av1.avl`, `vp9.avl`, `h265.avl`, `h264.avl`, and `build.json`.

3. **Inspect / validate** (optional):

   ```sh
   npx avl inspect  dist/ambient-drift/av1.avl
   npx avl validate dist/ambient-drift/av1.avl
   ```

4. **Embed** on a page. Copy the exact `type` and `integrity` values from
   `dist/ambient-drift/build.json` (the `sourceMarkup` field gives you the lines
   directly). Register the element once:

   ```html
   <script type="module" src="/motion.js"></script>

   <aval-player state="idle" width="512" height="512">
     <source src="/ambient-drift/av1.avl"  type='application/vnd.aval; codecs="av01..."'>
     <source src="/ambient-drift/vp9.avl"  type='application/vnd.aval; codecs="vp09..."'>
     <source src="/ambient-drift/h265.avl" type='application/vnd.aval; codecs="hvc1..."'>
     <source src="/ambient-drift/h264.avl" type='application/vnd.aval; codecs="avc1..."'>
     <img slot="fallback" src="/ambient-drift.png" alt="">
   </aval-player>
   ```

   ```js
   // motion.js
   import { defineAvalElement } from "@pixel-point/aval-element";
   defineAvalElement();
   ```

## Transparency note

Alpha is carried by the **AV1 and VP9** renditions; supporting browsers pick one
of those and you get true transparency. **H.264 / H.265** are opaque fallbacks.
If you only care about alpha-capable browsers, you can delete the `h264` and
`h265` entries from `motion.json` for a smaller bundle.

## Growing to multiple states later

This is a one-state loop today (`idle` only). To add interactivity (e.g. a
hover reaction), you'd author additional `units` as frame ranges of a longer
source, add `states` and `edges` between them, and connect them at `portalFrames`
— see `examples/grass-rabbit/motion.json` and `docs/states-and-triggers.md`.
