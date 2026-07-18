# Frame-generation brief — "ambient-drift"

Hand this spec to your image/video AI (or a render script). It defines the
exact pixels AVAL needs. The companion `motion.json` is already written to
match these numbers — if you change any number here, change it there too.

## Hard requirements (AVAL will reject the build otherwise)

| Property        | Value                                              |
|-----------------|----------------------------------------------------|
| Output          | **72 PNG files**                                   |
| Filenames       | `frame-0000.png` … `frame-0071.png` (zero-padded 4 digits) |
| Location        | this project's `frames/` folder                    |
| Dimensions      | **512 × 512 px**, identical for every frame        |
| Pixel format    | **RGBA** with a true, per-pixel **alpha channel**  |
| Background      | **Fully transparent** (alpha 0) — no baked-in white, black, or checkerboard |
| Timing          | Designed for **24 fps → ~3.0 s** total             |

## The one rule that matters most: seamless loop

Frame **0071 must flow into frame 0000 with no visible jump.** The animation
plays forever; any discontinuity at the seam reads as a stutter every 3 seconds.

The reliable way to guarantee this for an ambient scene is **integer-cycle
motion**: every moving element completes a whole number of cycles across the
72 frames, so at frame 72 everything is back exactly where frame 0 started.

- A drifting element on a **sine path** should use a period that divides 72
  (e.g. 72, 36, 24, 18 frames per cycle) — never an arbitrary period.
- **Twinkle / opacity pulses** likewise cycle on a divisor of 72.
- **Drift that travels** (e.g. slowly upward) should **wrap**: as one element
  exits the top, an identical element enters the bottom at the matching offset,
  so the overall field looks unchanged at the seam.

If your generator can't be made cyclic, fall back to a **boomerang** (frames
0→35 forward, then 35→0 reversed to fill 36→71) — seam-free but the motion
visibly reverses.

## The scene

An **ambient, atmospheric loop** — soft, slow, unobtrusive, meant to sit behind
or beside UI. Suggested direction (adapt freely):

- Several **soft glowing orbs / motes of light** at different sizes and depths,
  drifting gently on looping sine paths, with subtle parallax (larger = nearer =
  moves more; smaller = farther = moves less).
- A slow **twinkle** on each mote (opacity breathing) on a 24- or 36-frame cycle.
- Optional faint **drifting haze or particles** for depth.
- Muted, cohesive palette; nothing that hard-cuts or flashes.
- Everything on transparent background so it composites over any page color.

Keep it calm: small amplitudes, slow speeds. Ambient means it should be almost
subliminal, not busy.

## Two ways to actually generate the frames

**Route A — per-frame image model.** Generate 72 stills from a fixed prompt +
seed, advancing a motion parameter each frame. Best control over the loop math;
watch for flicker between frames (lock seed, describe the scene identically each
time, only change positions). Abstract motes tolerate minor flicker well.

**Route B — video model → extract → matte alpha.** Prompt a text/image-to-video
model for a 3 s seamless ambient loop, then:
1. Extract frames: `ffmpeg -i clip.mp4 -vf fps=24 frames/frame-%04d.png`
   (this yields `frame-0001…`; renumber to start at `0000`, or set
   `firstNumber` to 1 in `motion.json`).
2. If the clip is opaque, remove the background **per frame** to get real alpha
   (a background-removal tool / matting model), re-saving as RGBA PNG.

Most AI video is opaque, so Route B needs the matting step to get transparency.
Route A can paint on transparency directly.

## Verify before compiling

```sh
# Count = 72
ls frames/frame-*.png | wc -l

# Every frame 512x512 with an alpha channel (RGBA)?  Look for "512x512" + "rgba".
ffprobe -v error -select_streams v -show_entries stream=width,height,pix_fmt \
  -of csv=p=0 frames/frame-0000.png
# Or, with ImageMagick:
identify -format "%f %wx%h %[channels]\n" frames/frame-0000.png
```

You want size `512x512` and channels including alpha (`rgba` / `srgba`).

## Then compile

From the repo root (or wherever the CLI is installed):

```sh
npx avl compile motions/ambient-drift/motion.json --out dist/ambient-drift
```

See this project's `README.md` for inspect/validate and the page embed markup.
