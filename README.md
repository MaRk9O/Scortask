<div align="center">

# 🎯 SCORTASK

**A real-time arcade camera game powered entirely by your own AI model — no server, no cloud, no data ever leaves your device.**

[![HTML](https://img.shields.io/badge/HTML-Single%20File-orange?style=flat-square&logo=html5)](.)
[![TensorFlow.js](https://img.shields.io/badge/TensorFlow.js-4.22.0-FF6F00?style=flat-square&logo=tensorflow)](https://www.tensorflow.org/js)
[![Privacy](https://img.shields.io/badge/Privacy-100%25%20Local-brightgreen?style=flat-square&logo=lock)](#privacy)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

![Scortask gameplay preview — camera pointing at everyday objects to score points]

</div>

---

## What is Scortask?

Scortask is a browser-based arcade game where your camera is the controller. The game gives you a target — like **"water bottle"** or **"headphones"** — and you have to find and show it to your camera as fast as possible. Every correct object earns points and bonus time. Chain them back-to-back to build a combo multiplier.

The twist: **the vision model is yours.** You bring your own `.tflite` or TensorFlow.js model, drop it in, and the game automatically figures out which of your model's classes are playable targets — filtering to only the everyday objects people realistically have nearby.

Everything runs offline in the browser. No backend. No API calls. No camera data ever transmitted.

---

## Features

- **Bring your own model** — `.tflite`, TensorFlow.js SavedModel, or Teachable Machine exports all work out of the box
- **Smart target filtering** — automatically picks sensible targets from your model's label set using word-boundary matching; wild labels like animals or exotic food are blocked
- **Rarity system** — targets are weighted as Common, Uncommon, or Rare, affecting how often they appear
- **Combo multiplier** — chain consecutive finds to multiply your score (up to ×5)
- **EMA confidence smoothing** — exponential moving average prevents flickering detections; a label must sustain confidence above 62% to register as a hit
- **Live camera HUD** — real-time confidence meter, current target, and what the model is seeing overlaid on the camera feed
- **Mobile-first layout** — camera fills the full screen on phones with a transparent HUD overlay; designed for portrait play
- **Autofocus & main camera** — requests continuous autofocus and steers away from ultra-wide lenses using resolution hints
- **100% local** — inference runs on-device via WebGL/WASM; camera frames never leave the browser sandbox

---

## How to Play

**1. Get a model**

Train one in [Teachable Machine](https://teachablemachine.withgoogle.com/train/image) using everyday objects as classes, then export it as a TensorFlow.js or TFLite model. Alternatively use any compatible image classification model.

**2. Load the game**

Open `Scortask.html` in a browser (Chrome or Safari recommended). Drop your model's files onto the upload zone. The game will scan your labels and tell you how many valid targets it found.

**3. Start**

Tap **Play**. On mobile you'll be asked to enable the camera with a dedicated tap — this is required by iOS Safari's permission model. Allow camera access when prompted.

**4. Find objects**

A target appears. Point your camera at the matching object. When the model recognises it with enough confidence, you score — a new target drops immediately. Keep going until the timer runs out.

---

## Scoring

| Action | Reward |
|---|---|
| Find a target | +5 pts base · +15 sec |
| ×2 combo | +10 pts |
| ×3 combo | +15 pts |
| ×4 combo | +20 pts |
| ×5 combo (max) | +25 pts |

Your best score is saved locally in `localStorage`.

---

## Target System

Scortask reads all class labels from your model and scores them against a curated list of universally common objects. A label qualifies as a target if it matches one of three tiers:

| Tier | Examples | Spawn weight |
|---|---|---|
| **Common** | mug, pen, phone, sock, pillow, watch, scissors | High |
| **Uncommon** | umbrella, belt, toothbrush, comb | Medium |
| *(custom labels)* | anything outside the filter | Passthrough |

Labels are matched using **word-boundary regex** — so `"can"` won't accidentally match `"pelican"`, and `"key"` won't match `"monkey"`. A hard blocklist of 80+ words (animals, vehicles, weapons, exotic food) ensures nothing absurd ever appears as a target.

---

## Model Compatibility

| Format | How to export |
|---|---|
| **Teachable Machine** | Export → TensorFlow.js or TFLite (floating point) |
| **TFLite** | Any `.tflite` image classification model |
| **TensorFlow.js** | `model.json` + weight shards in a ZIP |

The game detects whether the model expects inputs in `[-1, 1]` (Teachable Machine) or `[0, 1]` (generic) and preprocesses accordingly.

---

## Privacy

> **Your camera never leaves your device.**

Once the page finishes loading, all gameplay is offline:

- Camera frames are processed entirely in the browser via TensorFlow.js (WebGL/WASM)
- No frames, scores, or model outputs are sent anywhere
- Scores are stored in your browser's `localStorage` only
- You can disconnect from the internet after page load and the game continues to work

The only network requests are one-time page loads: the HTML from your host, TensorFlow.js from jsDelivr CDN, and Google Fonts — none of which involve camera data.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Vanilla JS (ES2020, no build step) |
| ML inference | [TensorFlow.js 4.22.0](https://www.tensorflow.org/js) + [tfjs-tflite 0.0.1-alpha.9](https://github.com/tensorflow/tfjs/tree/master/tfjs-tflite) |
| Camera | `getUserMedia` with multi-constraint fallback + continuous autofocus |
| Fonts | Orbitron · Barlow · Space Mono (Google Fonts) |
| Bundling | None — single `.html` file |

---

## Running Locally

No server required for basic use. Just open the file:

```bash
# Clone
git clone https://github.com/MaRk9O/scortask.git
cd scortask

# Open directly
open Scortask.html
```

> **Note for TFLite models:** The `tfjs-tflite` WASM runtime requires the page to be served over HTTPS or `localhost`. For local TFLite testing, use a simple server:
> ```bash
> npx serve .
> # then open http://localhost:3000
> ```
> Plain `.tflite` files will not load if you open the HTML via `file://`. TensorFlow.js models (`.json` + weights) work fine from `file://`.

---

## Project Structure

```
scortask/
└── Scortask.html    # Entire game — single self-contained file
```

Everything — styles, game logic, model loading, camera handling, scoring, UI — lives in one HTML file with no dependencies to install.

---

## License

MIT — do whatever you want with it.
