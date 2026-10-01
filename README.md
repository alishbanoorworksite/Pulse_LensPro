# PulseLens Pro

**Real-time usability and behaviour tracker in a single HTML file.**

PulseLens Pro instruments a web page as the visitor uses it. It detects frustration signals (rage clicks, dead clicks, cursor jitter, idle time), turns them into a live usability score, and visualises interaction as heatmaps, a session replay and a funnel. It also includes interactive tests for three HCI laws (Fitts's, Hick's and Miller's). No build step, no framework, no dependencies to install.

> **Privacy:** all tracking runs in the browser tab. Nothing is uploaded anywhere. Data leaves the page only when you click an export button.

---

## Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Exported data](#exported-data)
- [How the key metrics work](#how-the-key-metrics-work)
- [Project structure](#project-structure)
- [Browser support and limitations](#browser-support-and-limitations)
- [Customisation](#customisation)
- [License](#license)

---

## Features

### Core usability tracking
| Capability | Description |
|---|---|
| Live metrics | Elapsed time, attention state (active/idle), total clicks, cursor travel distance, direction changes, keyboard vs mouse share |
| Rage-click detection | 3 or more clicks within 600 ms inside a 40 px radius raise an on-screen alert |
| Dead-click detection | Clicks on non-interactive elements are counted and logged |
| Usability score | A 0–100 gauge penalised by rage clicks, dead clicks, jitter and idle time |
| Sparklines | Live cursor speed (px/s) and clicks per interval |
| Interaction maps | Mouse-movement heatmap, click map and scroll-depth map, switchable by tab |
| Attention (dwell time) | Time spent hovering each tracked card |
| Funnel drop-off | Aggregated across visits on the same device via `localStorage` |

### Session tools
- **Session recording, tagging and clips:** record a session, tag moments, and mark clip start and end points.
- **Session replay:** play back the recorded cursor path at 1×, 2× or 4×.
- **Form analytics, exit-intent detection and a feedback widget.**
- **Local chat panel and speech-to-text transcription** (Web Speech API, Chrome only).
- **Optional webcam expression module** (best-effort; see [limitations](#browser-support-and-limitations)).

### HCI laws (interactive tests)
- **Fitts's Law:** target acquisition time.
- **Hick's Law:** decision time against number of options (2, 4, 8, 16).
- **Miller's Law:** working-memory digit span.

### Behaviour insights (page-level tracker)
A dedicated card at the bottom of the dashboard. It works in full-page coordinates, so overlays stay aligned on long pages.

- Numbered **click dots** and a **click scatter** overlay
- **Mouse path** with direction arrows and Start/End markers
- **Page visibility** changes (tab switched or hidden)
- **Card visibility** events (a dashboard card entering or leaving the viewport)
- **Summary dialog:** clicks, mouse points, scrolls, scroll depth, time on page, clicks per minute and the most-clicked elements
- **Undo last click, clear visuals, reset insights, view data** and a live event log
- A **separate JSON export** that does not interfere with the main session export

---

## Quick start

No installation is required.

**Option 1 — open the file**
```text
Double-click index.html
```

**Option 2 — serve it locally** (recommended; some browser features such as the camera and speech APIs behave better over `http://localhost`)
```bash
npm start
# or
python3 -m http.server 3000
```
Then open <http://localhost:3000>.

**Option 3 — GitHub Pages**
Push the repository, then go to **Settings → Pages → Deploy from branch** and select the root of `main`.

---

## Usage

1. Open the page. Tracking starts automatically.
2. Move the mouse, click, scroll and type. The **Live session metrics** card and the usability gauge update in real time.
3. Switch the **Interaction maps** tabs to change the full-page overlay.
4. Use **Record session**, **Tag moment** and **Mark clip start** to capture moments, then replay them in the **Session replay** card.
5. Run the Fitts, Hick and Miller tests from their cards.
6. Scroll to **Behaviour insights** for click dots, the mouse path and the summary dialog.
7. Click **Export session JSON** (header) or **Export insights JSON** (Behaviour insights card) to save data.

---

## Exported data

| Button | File | Contents |
|---|---|---|
| Export session JSON | `usability-session.json` | Duration, click, rage and dead-click counts, scroll depth, cursor distance, direction changes, keyboard/mouse events, usability score, Miller span, dwell times, Fitts and Hick trials, tags, clips, form-field analytics, feedback, transcript, funnel data and the event timeline |
| Export insights JSON | `behaviour-insights.json` | Session ID, duration, summary counts, click list (with page and viewport coordinates and element labels), saved mouse points, scroll events and the full insight event stream |

A representative example of the insights export is in [`sample-behaviour-insights.json`](sample-behaviour-insights.json).

Insight event types: `click`, `mousemove`, `scroll`, `page_visibility`, `element_visibility`, `export`.

---

## How the key metrics work

- **Rage click:** the current click plus at least two earlier clicks within 600 ms and 40 px.
- **Dead click:** the click target is not inside `a, button, input, select, textarea, [role="button"], [data-track], [data-funnel-step], label, summary`.
- **Idle:** no activity for 5 seconds.
- **Usability score:** starts at 100. It subtracts 9 per rage click, 4 per dead click, up to 30 for cursor direction changes (0.35 each) and 8 while idle.
- **Scroll depth (insights card):** `(scrollY + viewportHeight) / documentHeight`, kept as the maximum reached.
- **Mouse sampling (insights card):** one point per 60 ms, capped at 5,000 points. Scroll events are sampled every 180 ms.

---

## Project structure

```text
.
├── index.html                       # The entire app: HTML, CSS and JavaScript
├── sample-behaviour-insights.json   # Example of the insights export
├── package.json                     # Metadata and a local-server script
└── README.md
```

The JavaScript is organised in three parts inside `index.html`:

1. `UsabilityTracker`: the core engine (an IIFE exposing `init`).
2. A bootstrap call that maps DOM element IDs to the engine.
3. The **Behaviour insights** add-on: a self-contained IIFE with scoped `bh-` CSS and IDs, independent of the engine.

---

## Browser support and limitations

- Designed for modern evergreen browsers (Chrome, Edge, Firefox, Safari).
- **Speech-to-text** uses the Web Speech API and works in Chromium-based browsers only.
- **Webcam expressions** load `face-api.js` and model weights from a CDN. If the network is blocked or camera permission is denied, the module degrades gracefully and the rest of the app is unaffected.
- **Fonts** (Playfair Display, Poppins) load from Google Fonts. The page falls back to system serif and sans-serif when offline.
- **Funnel data** is stored in `localStorage` on one device. It is a demo of aggregation, not cross-user analytics. Point it at a shared backend for real funnels.
- **Chat** is a local demo. It needs a WebSocket backend for multi-user use.
- Event and mouse-point buffers are capped to protect memory on long sessions.
- On pages where `body` hides horizontal overflow, overlay size follows the document's scroll width and height, and refreshes once per second.

---

## Customisation

- **Theme:** edit the CSS variables in `:root` and `html[data-theme="dark"]` (`--brand`, `--bg-0`, `--ink-0` and so on).
- **Rage-click sensitivity:** change `RAGE_WINDOW_MS`, `RAGE_RADIUS_PX` and `IDLE_MS` near the top of the engine.
- **Track your own elements:** add `data-track="Name"` to any element to include it in dwell tracking and card-visibility events.
- **Funnel steps:** add `data-funnel-step="step-name"` to elements to record when they are reached.
- **Insights limits:** adjust `MAX_MOVES`, `MAX_EV` and `LOG_LIMIT` in the insights script.

---

## License

Released under the MIT License. Add a `LICENSE` file with your name and year before publishing.
