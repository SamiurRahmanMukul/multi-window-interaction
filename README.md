## Multi‑Window State Sync Demo (Vite + TypeScript)

A small demo app that uses the SharedWorker API to synchronize minimal window state (position and size) across multiple browsing contexts of the same origin. Each window draws a canvas circle at its center and red lines to the centers of all other open windows.

### Tech stack

- **Build tool**: Vite 5
- **Language**: TypeScript 5
- **Platform APIs**: SharedWorker, Canvas 2D

### Prerequisites

- **Node**: 18+ (recommended 20+)
- **Package manager**: npm (bundled with Node)

SharedWorker requires serving over HTTP(S). Do not open `index.html` from the filesystem.

### Getting started

- **Install dependencies**

  ```bash
  npm install
  ```

- **Start dev server**

  ```bash
  npm run dev
  ```

  Open the printed URL (typically `http://localhost:5173`).

- **See multi‑window sync**
  - Open one or more additional tabs or windows at the same URL.
  - Resize or move any window; others will receive updates and you will see red lines connecting centers.

### Build and preview

- **Build for production**

  ```bash
  npm run build
  ```

  This runs `tsc` then `vite build`, emitting static assets into `dist/`.

- **Preview the production build**

  ```bash
  npm run preview
  ```

### Project structure

```path
index.html                # Canvas and script entry
src/
  main.ts                 # App entry; draws canvas + wiring
  worker.ts               # SharedWorker: tracks all windows, broadcasts sync
  workerHandler.ts        # Client wrapper for worker messaging + change detection
  windowState.ts          # Window state capture + diffing utilities
  types.ts                # Shared TypeScript types for messages/state
```

### How it works

- **Worker bootstrap**: Each window constructs a `SharedWorker` via `new URL("worker.ts", import.meta.url)` so Vite can bundle it.
- **Connection**: On connect, the worker assigns a monotonically increasing `id` to the window and stores its initial `WindowState`.
- **Sync**: When any window changes (size/position), the client posts a `windowStateChanged` message. The worker updates internal state and broadcasts a `sync` message with all windows' states to every connected client.
- **Render**: The client computes relative coordinates and draws lines from the current window's center to each other window's center on the canvas.

Key files to explore:

- `src/worker.ts` — message routing and broadcast
- `src/workerHandler.ts` — connection lifecycle and throttled change reporting
- `src/main.ts` — canvas rendering and event loop

### Scripts

- **dev**: `vite`
- **build**: `tsc && vite build`
- **preview**: `vite preview`

### Browser notes

- SharedWorker is supported in modern Chromium and Firefox. Safari support has improved in recent releases; ensure you are on an up‑to‑date version.
- All windows/tabs must share the same origin for a SharedWorker instance to be shared.

### Troubleshooting

- "Failed to construct 'SharedWorker'": ensure you are using the dev server or `vite preview`; do not open files via `file://`.
- Nothing renders: check that the canvas exists (`#canvas`) and that your browser supports SharedWorker.
- No lines between windows: make sure multiple tabs/windows are open at exactly the same origin, and wait a moment for the periodic change detection.

### Related post

- [LinkedIn: Multi‑Window state sync demo](https://www.linkedin.com/feed/update/urn:li:activity:7171523027929645057/)

### License

ISC
