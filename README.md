# GESTURA ✦

A real-time 3D particle system you control with your hands. Open your palm to push 6,000 particles outward. Make a fist and they explode. Pinch to pull them back in. Point one finger and the entire system shifts color as you hold it.

It runs in the browser. No installation. No server (for most features). No dependencies beyond a few CDN scripts.

![GESTURA preview](preview.gif)

---

## What it does

Your webcam feeds into a hand-tracking model (TensorFlow.js + HandPose), which classifies your gesture every frame and maps it to particle behavior. The particles themselves run in Three.js with a custom GLSL shader — additive blending, smooth point sprites, per-particle color and size animation.

There are eight particle shapes you can switch between:

| Shape | What it looks like |
|---|---|
| �Sphere | Fibonacci-distributed points on a rotating globe |
| Heart | Parametric heart curve extruded into a 3D volume |
| Flower | Polar rose with 8 petals, animated wave height |
| Saturn | Ring system (60% of particles) + sphere core |
| Fireworks | 40 burst origins, each with 150 particles arcing outward |
| DNA | Two interlocked helices rotating at different phases |
| Galaxy | 3-arm spiral with per-particle scatter offset |
|  Wave | 120×50 grid with layered sine wave interference |

Switching shapes triggers a smooth cubic-eased morph — every particle interpolates from its current position to the target shape over about 700ms.

---

## Gestures

| Gesture | Effect |
|---|---|
| 🖐 Open palm | Expand the formation outward |
| ✊ Fist | Trigger an explosion burst |
| 🤏 Pinch | Contract everything inward |
| ☝ One finger | Continuous hue rotation |
| ✌ Peace sign | Jump to the next shape |

The classifier runs a majority vote over the last 10 frames to filter out noise. Pinch distance is normalised against hand size so it works whether you're 30cm or a metre from the camera.

If you don't want to use the camera — or can't — the **Gesture Pad** on the right side of the screen gives you the same five gestures as clickable buttons. It works on `file://`, works on mobile, works everywhere.

---

## Getting started

### Option 1 — just open it

Download `particle-gestures.html` and open it in Chrome or Edge. The particle system starts immediately. The Gesture Pad works without any server.

For hand tracking, you'll need a secure context (the browser requires it for camera access):

```bash
# If you have Node installed:
npx serve .

# Or Python:
python3 -m http.server 3000
```

Then open `http://localhost:3000/particle-gestures.html`.

### Option 2 — host it

Drop the file anywhere that serves over HTTPS. Camera permission will be requested automatically on load.

---

## Keyboard shortcuts

If you're on desktop without a camera, or just prefer keys:

| Key | Action |
|---|---|
| `1` – `8` | Switch to shape 1–8 |
| `←` / `→` | Previous / next shape |
| `Space` | Explosion burst |
| Drag | Rotate the formation |
| Scroll | Zoom in / out |
| Pinch (touch) | Zoom on mobile |

---

## How the hand tracking works

The camera feed goes to `handpose.load()` from `@tensorflow-models/handpose@0.0.7`, which returns 21 3D landmark coordinates per detected hand — one for each joint and fingertip.

From those 21 points, gesture classification looks at:

- **Finger extension**: Is the fingertip above the PIP joint in image space? (`tip.y < pip.y`)
- **Pinch distance**: Euclidean distance between thumb tip and index tip, divided by wrist-to-middle-MCP distance to normalise for hand size
- **Finger count**: How many of the four non-thumb fingers are extended

That's it. No ML for the gesture layer — just geometry.

Worth knowing: the video element has `transform: scaleX(-1)` in CSS to mirror it like a selfie cam. The skeleton overlay canvas applies the same mirror via `ctx.scale(-1, 1)` so the skeleton lines up correctly.

### Shader note

Three.js r128's `ShaderMaterial` injects `attribute vec3 color` into the vertex shader when `vertexColors: true` is set. Declaring a second `color` attribute in the shader body causes a GLSL ES 3.0 duplicate declaration error. The particle color attribute is named `pColor` here to avoid that conflict, with `vertexColors: false` on the material.

---

## Browser compatibility

| Browser | Particles | Hand tracking |
|---|---|---|
| Chrome 90+ | ✅ | ✅ |
| Edge 90+ | ✅ | ✅ |
| Firefox 88+ | ✅ | ✅ (may be slower) |
| Safari 15+ | ✅ | ⚠️ WebGL backend varies |
| Mobile Chrome | ✅ | ✅ (rear cam defaults, switch manually) |

TensorFlow.js uses the WebGL backend for inference. On devices without GPU acceleration it falls back to CPU, which runs noticeably slower. The particle system itself is unaffected either way.

---

## File structure

Everything is in a single HTML file. No build step, no node_modules, no config. The external dependencies load from CDN at runtime:

```
particle-gestures.html
│
├── Three.js r128           (cdnjs — 3D renderer + shader)
├── TensorFlow.js 3.21.0    (jsdelivr — unified bundle, required by handpose)
└── HandPose 0.0.7          (jsdelivr — 21-point hand landmark model)
```

The TF.js load order matters: `tf.min.js` must load before `handpose.min.js`. The unified bundle (`tf.min.js`) is required — the split `tfjs-core` + `tfjs-backend-webgl` packages don't include `loadGraphModel`, which handpose calls internally.

---

## Customising

It's one file, so customisation is just editing the JS directly. A few things people tend to change:

**Particle count** — `const N = 6000` near the top. More particles = more GPU load. 10,000 is fine on a modern machine; 20,000 starts to push it on integrated graphics.

**Adding a shape** — Add an entry to the `TPL` object and a matching palette to `PAL`. The morph system picks it up automatically.

```js
TPL.cube = (i, t) => {
  const face = Math.floor(i / (N / 6));
  // ... return [x, y, z]
};
PAL.cube = [[1, 0.5, 0.1], [0.8, 0.3, 0.9]];
```

Then add a button to `#tbar`:
```html
<button class="tb" data-t="cube">🟧 Cube</button>
```

**Gesture sensitivity** — The pinch threshold is `normPinch < 0.22` in `classify()`. Lower = tighter pinch required. The history window is `const HIST = 10` — more frames = smoother but slower response.

---

## License

MIT. Use it, break it, build on it.

---

## Credits

Built on [Three.js](https://threejs.org), [TensorFlow.js](https://www.tensorflow.org/js), and the [HandPose model](https://github.com/tensorflow/tfjs-models/tree/master/handpose). The hand landmark topology comes from the MediaPipe team at Google — the same model that runs on Android's live AR filters.
