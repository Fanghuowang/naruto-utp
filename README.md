# naruto-rasengan

A browser demo that turns your webcam into a Naruto effect: open your palm and a rasengan appears above your hand. Two hands = two rasengans.

## Purpose

- Detects hands via MediaPipe Hands, draws a glowing finger-tracking skeleton on top of the webcam feed, and overlays a rasengan video above any open palm.
- Supports up to two simultaneous hands, each with its own color palette and its own independent rasengan.
- Does **not** record, upload, or transmit any video — all hand tracking happens locally in the browser.
- Does **not** require any backend, build step, or framework — it is a single static HTML file plus one video asset.

## When to use

- You want a quick, copy-paste-able demo of MediaPipe Hands in vanilla JS.
- You want a fun party-trick page that maps a hand gesture (open palm) to a visual effect.
- You want a reference for: per-hand state, gesture edge-detection, blending a transparent-feeling video on top of a webcam canvas.

Don't use it as a basis for anything that needs precise, calibrated hand pose (it's a vibe demo, not a motion-capture pipeline).

## Inputs

Required:
- A device with a webcam.
- A modern browser with `getUserMedia` support (Chrome, Edge, Safari 14+, Firefox).
- The page must be served from a secure origin (`http://localhost` or `https://...`). Opening `index.html` directly via `file://` will block the camera in most browsers.
- `assets/rasengan.mp4` — the rasengan video referenced by `<video src="assets/rasengan.mp4">`. The repo currently ships `assets/naruto.mp4`; either rename the file to `rasengan.mp4` or change the `src` attributes in `index.html` to match.

Optional / tunable (constants inside `index.html`):
- `PALETTES.Left` / `PALETTES.Right` — bone, joint, and glow colors per hand. Defaults: warm orange (left), cool cyan (right).
- `lift = handSize * 1.8` in `placeOrb` — how high above the palm the orb floats. Increase to raise it further.
- `charge += 0.06 / -0.18` in `onResults` — fade-in / fade-out speed. Bigger positive = orb appears faster; bigger negative = orb disappears faster.
- `maxNumHands: 2` — raising this past 2 currently won't help because there are only two `<video class="rasengan">` slots in the markup.
- `minDetectionConfidence` / `minTrackingConfidence` — lower (e.g. `0.5`) for easier detection at odd angles, higher for fewer false positives.

Known bad inputs:
- `file://` URL — camera will be blocked. Serve via a local web server (see Run instructions).
- Missing `assets/rasengan.mp4` — skeleton still draws, but no orb. Check the DevTools console for a 404.
- Camera permission denied — nothing renders. Re-grant permission in browser site settings and reload.

## Output contract

On screen, top to bottom in z-order:
1. Mirrored webcam feed (so it feels like a mirror).
2. Hand skeleton overlay: thin colored bones (lineWidth 3) with a colored glow, plus filled joint dots (radius 3.5). Left hand = orange bones + yellow joints. Right hand = cyan bones + magenta joints.
3. A radial vignette darkening the edges.
4. Up to two rasengan videos, blended with `mix-blend-mode: screen`, positioned above each open palm, opacity proportional to per-hand charge.
5. A hint banner at the bottom.

Behavioral guarantees ("done" criteria):
- Opening a palm causes the corresponding orb to fade in within ~1 second and the video to restart from frame 0.
- Closing the palm or removing the hand causes the orb to fade out within ~0.3 seconds.
- Each hand's state is independent — one open + one closed = one orb visible.
- The skeleton tracks landmarks in real time (no perceptible lag at 30 FPS on a modern laptop).
- No console errors during normal operation.

## Guardrails

- The page **only** uses the local webcam stream. No network calls except CDN fetches for MediaPipe scripts on page load.
- Camera access requires explicit user permission via the browser prompt — don't try to suppress it.
- The rasengan asset is a third-party clip (`assets/naruto.mp4`); replace it before redistributing if you don't have rights to it.
- The MediaPipe model files are served from `cdn.jsdelivr.net`; if you need offline use, mirror them locally and update the `locateFile` callback.
- Don't deploy this to a public URL that auto-loads the camera without context — make the "open your palm" affordance obvious.

## Examples

### Example A (happy path)

Input:
- User opens the page on `http://localhost:8000`, grants camera permission, holds up an open right palm to the camera.

Expected output shape:
- A cyan + magenta skeleton appears tracking the right hand.
- Within ~1 second, a glowing rasengan orb fades in and floats above the palm, scaled and lifted relative to hand size.
- Closing the hand into a fist causes the orb to fade out within ~0.3 seconds; the skeleton stays as long as the hand is visible.

### Example B (two hands)

Input:
- Both hands raised, both palms open.

Expected output shape:
- Two skeletons, one in the warm palette and one in the cool palette.
- Two independent rasengans, each tracking its own palm. Closing one hand fades only that orb out.

### Example C (edge case — no hands / closed fist)

Input:
- No hand in frame, or hand in frame but fist closed.

Expected output shape:
- No skeleton, no orb. Webcam + vignette + hint banner still visible. Any previously-visible orb decays to opacity 0 within ~0.3 seconds.

## Run instructions

From the project root:

```bash
cd "/path/to/naruto-rasengan"
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser and grant camera permission.

Artifacts:
- The single source file is `index.html`.
- Video assets live in `assets/`. The `<video>` tags currently reference `assets/rasengan.mp4` — keep that file there (or rename `naruto.mp4` to match).
- Nothing is written to disk at runtime.

## Change log

- **v0.1** — initial rewrite from the reference repo (`gprem09/naruto`). Single `naruto.mp4` reused for both hands; per-hand palettes (warm orange / cool cyan) with distinct bone vs joint colors; custom landmark drawing via `arc()`; radial-gradient vignette replacing the multiply-blend darkness layer.
- **v0.2** — thinned the skeleton lines (bone `lineWidth` 6 → 3, joint radius 6 → 3.5) for a less heavy overlay.
- **v0.3** — rasengan now floats *above* the palm instead of sitting on it: anchored at the wrist↔mid-knuckle midpoint and lifted upward in screen space (lift = `handSize * 1.8`), independent of hand rotation.
- **v0.4** — added `autoplay preload="auto"` to the rasengan videos and a `console.warn` on play failure so silent autoplay/policy issues are visible.
- **v0.5** — switched the orb asset reference from `naruto.mp4` to `rasengan.mp4` (rename your asset accordingly).
