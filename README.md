# PostureTracker

> An in-browser, real-time posture correction and session-tracking tool powered by machine learning — no installation, no server, no data leaving your device.

---

## Elevator Pitch

Most of us spend hours hunched over a keyboard without realising it. PostureTracker uses your webcam and Google's MediaPipe Pose model to watch your upper body in real time, alert you the moment you start to slouch, and remind you to take a break before fatigue sets in. Everything runs entirely inside your browser — your camera feed is never uploaded anywhere.

---

## Key Features

### Real-Time Upper-Body Posture Detection
- Streams your webcam through the MediaPipe Pose ML model at up to 30 fps
- Detects 33 body landmarks and renders a colour-coded skeleton overlay on a live `<canvas>` element — green for torso, purple for arms, pink for legs
- Key posture landmarks (nose, left shoulder, right shoulder) are highlighted with dashed rings for clarity

### Personalised Calibration
- Sit up straight and click **Calibrate** — the app records the Y-axis distance between your nose and the midpoint of your shoulders as your personal baseline
- Accommodates any height, chair height, or camera angle
- In every subsequent frame, the live distance is compared against your baseline; dropping below **85%** triggers the bad posture alert

### Bad Posture Alert System
- **Posture Status card** switches between ✓ Good Posture (green) and ⚠ Bad Posture (red, pulsing) in real time
- **Radial arc gauge** fills from amber to red the longer you hold bad posture, with a linear bar underneath mirroring the fill
- If you stay in bad posture for **5 continuous seconds**, a single-tone Web Audio API beep fires and a toast notification slides up from the video feed
- The gauge and timer reset the instant your posture improves

### Customisable Session Timer
- Digital **HH:MM:SS** clock counts active time only — it pauses automatically whenever the ML model loses sight of your pose
- **Session Length slider** (1–60 minutes) lets you choose your break interval before starting; the slider is locked while a session is running to prevent accidental changes
- When the session limit is reached, a full-screen **Take a Break!** overlay appears with an animated icon, a three-note ascending chime (C5 → E5 → G5, distinct from the posture beep), and a native desktop notification

### Web Audio & Desktop Notifications
- **Bad posture beep** — 520 Hz sine wave with a fast fade envelope; fires once per continuous bad-posture event
- **Session complete chime** — gentle triangle-wave ascending chord (C5 / E5 / G5) spaced 280 ms apart
- **Notification API** — permission is requested on page load; when the session ends a desktop notification reading *"Session Complete! Time to stretch."* fires even if the app is in a background tab
- The `AudioContext` is warmed up on the first user gesture, so browser autoplay policies never block the alerts

### Start / Stop Controls
- **Enable Camera** button on the splash screen requests webcam access and loads the ML model
- **Stop Tracking** (header) fully halts the camera feed and detection loop, pauses the session timer at its current value, clears all active overlays, and resets bad-posture state
- **Resume Tracking** restarts the camera and picks up the session clock exactly where it left off
- **Calibrate** button appears only while tracking is active; flashes green on a successful calibration

---

## Tech Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 |
| Styling | CSS3 (custom properties, CSS Grid, SVG arc gauge) |
| Logic | Vanilla JavaScript (ES2020+) |
| ML Model | [MediaPipe Pose](https://google.github.io/mediapipe/solutions/pose) via CDN |
| Audio | Web Audio API (`AudioContext`, `OscillatorNode`) |
| Notifications | Browser Notification API |
| Build tooling | [Vite](https://vitejs.dev/) (dev server only — the output is a single self-contained HTML file) |

### MediaPipe CDN scripts used

```html
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose/pose.js"></script>
```

---

## How to Run

### Option 1 — Open directly in a browser (simplest)

Because the entire application is a single `index.html` file with no build step and no back-end, you can run it by simply opening the file:

```bash
# Clone or download the repository, then:
open artifacts/pose-detector/index.html
# or drag the file into any modern browser window
```

> **Browser requirements:** Chrome 90+, Edge 90+, or Firefox 90+ with webcam access. Safari is supported but desktop notifications may behave differently.

### Option 2 — Run on Replit (recommended for sharing)

1. Fork or import the project on [Replit](https://replit.com)
2. The **Pose Detector** workflow starts automatically via Vite
3. Click the preview URL — no extra setup needed
4. Allow camera and notification permissions when prompted

### Option 3 — Local Vite dev server

```bash
# Prerequisites: Node.js 18+ and pnpm
pnpm install
pnpm --filter @workspace/pose-detector run dev
# Then open http://localhost:<PORT> in your browser
```

---

## Usage Guide

1. **Open the app** and click **Enable Camera** — allow camera access when the browser prompts
2. Allow **desktop notifications** when prompted (optional, but recommended for background alerts)
3. **Set your Session Length** using the slider (default: 8 minutes) before starting
4. Sit up straight in front of the camera, then click **Calibrate** in the header
5. The app will now monitor your posture in real time — the dashboard panel on the right shows your posture status, bad posture timer, and session clock
6. Click **Stop Tracking** at any time to pause; click **Resume Tracking** to continue from where you left off
7. When the session ends, dismiss the break overlay with **Resume Session** to start a fresh session

---

## Architecture Notes

- The webcam stream is piped into MediaPipe via the `Camera` utility on every frame; results are drawn onto an `<canvas>` element that sits absolutely positioned over the `<video>` element, both mirrored with `transform: scaleX(-1)` so the view feels natural
- Posture detection uses only three normalised landmarks (`nose.y`, `left_shoulder.y`, `right_shoulder.y`), making it robust to partial occlusion of the lower body
- The session timer runs on a 1-second `setInterval` gated on `latestLandmarks !== null`, so it accurately tracks only time when a pose is actively detected
- All state is in-memory; nothing is persisted to `localStorage` or any external service

---

## License

MIT — free to use, modify, and distribute.
