# Retro Game Vault

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/TailwindCSS-v4-06B6D4?logo=tailwindcss&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

A local-first, browser-based retro J2ME game library and launcher. Bring your own `.jar` files and play them instantly — no installs, no accounts, nothing uploaded anywhere.

## Overview

Old J2ME (feature-phone era) games only run inside an actual emulator — which means finding one, installing it, and finding a compatible JAR file. That friction keeps most of these games buried. Retro Game Vault removes it: drop a `.jar` into the browser and it plays right there, backed by [FreeJ2ME-Web](https://github.com/zb3/freej2me-web) (a CheerpJ-compiled J2ME emulator) running fully client-side. No server ever sees the game file.

**Status:** MVP complete, containerized, and deployed.

## Tech Stack

- **Language:** TypeScript (strict)
- **Frontend:** React 19 + Vite 8, Tailwind CSS v4 (CSS-first config), Radix UI primitives
- **Storage:** IndexedDB — all JARs, metadata, and library state stay on-device, no backend
- **Runtime engine:** [FreeJ2ME-Web](https://github.com/zb3/freej2me-web), a CheerpJ-compiled J2ME emulator, vendored and embedded via iframe
- **Testing:** Playwright end-to-end tests
- **Deployment:** Multi-stage Docker build → nginx static serving, via Docker Compose

## Features

- Add any J2ME `.jar` file and play it instantly in-browser
- Local library — every added game persists across sessions, on-device only (IndexedDB)
- Manual screen width/height/rotation override, for games whose real resolution can't be auto-detected from the filename
- Library → Game Details → Gameplay flow with no page reloads and no route changes for modals
- Exit is dual-signaled (engine `postMessage` + a host-level fallback Exit control), so a stuck game never traps the user
- Playing a second game after the first always gets a clean runtime — each Gameplay session tears down and recreates the emulator iframe rather than trying to hot-swap game state inside it

All five behaviors above are exercised by the project's own Playwright suite (`app/e2e/gameplay.spec.ts`): add→play→exit→reload-persists, sequential two-game play, custom width/height/rotation rendering at the correct aspect ratio, and filename-based auto dimension detection.

## Architecture

```
┌──────────────────────────────────────────────┐
│                 User's Browser                │
│                                                │
│   React Host App                              │
│   (Library / Game Details / Gameplay screens) │
│              │                                │
│              ▼                                │
│   Runtime Adapter                             │
│   (owns iframe lifecycle + postMessage        │
│    contract — see runtimeAdapter.ts)          │
│              │                                │
│              ▼                                │
│   <iframe> — FreeJ2ME-Web                     │
│   (CheerpJ-compiled J2ME engine, vendored,    │
│    one small approved patch file)             │
│              │                                │
│              ▼                                │
│        Your .jar file                         │
└──────────────────────────────────────────────┘
```

See [`diagrams/architecture.md`](./diagrams/architecture.md) for an editable Mermaid version, and [`diagrams/gameplay-sequence.md`](./diagrams/gameplay-sequence.md) for the install/exit message sequence.

The host app owns the JAR file, game metadata, and library state as the single source of truth, persisted in IndexedDB. The engine's installed copy of a JAR inside its own virtual filesystem is a *derived* runtime install, never a second source of truth.

Every time a Gameplay session starts, the host creates a fresh iframe and re-installs the JAR into it over `postMessage`. Every time it ends, the iframe is destroyed. This sidesteps the engine's own limitation that it can't switch games in-place.

## Key Engineering Decisions

- **Integrate an unmodified third-party engine via `postMessage`, not forking it.** FreeJ2ME-Web wasn't built to be embedded — its default flow is its own file-picker page. Rather than fork the engine's Java source or duplicate its JAR-loading/RMS/display logic in the host app (which `CLAUDE.md`'s `app/`↔`engine/` boundary explicitly forbids), the project made one small, scoped patch to a single engine file (`engine/web/src/main.js`) to accept a host-driven install and emit a ready/exit signal, and built the entire integration around that contract. See [`docs/ARCHITECTURE_NOTES.md`](./docs/ARCHITECTURE_NOTES.md).
- **Tear down and recreate the iframe per session instead of hot-swapping.** FreeJ2ME-Web has no supported in-place game-switch path. Rather than reverse-engineer one, each Gameplay session gets a brand-new iframe, verified by the project's own two-games-in-sequence e2e test.
- **Dual-signaled exit.** The engine posts a `freej2me-exit` message on its own exit button, but a host-level Exit control exists independently, so a hung or unresponsive engine instance never strands the user without an exit path.
- **Build-time env var baking for the demo video URL**, over a runtime-generated `env.js` — chosen because the value (a YouTube embed URL) changes rarely, and Vite already bakes `VITE_*` vars at build time; a runtime-config layer would add moving parts for no real benefit at this project's scale.

## Technical Highlights

A deeper look at the three hardest bugs found while integrating with the real (unmodified) FreeJ2ME-Web engine is in [`docs/TECHNICAL_HIGHLIGHTS.md`](./docs/TECHNICAL_HIGHLIGHTS.md):

- A pointer-rotation bug that only ever reproduced under real mouse input, never in automated click tests.
- A silent input-drop bug in the engine's internal event queue with no error signal.
- Getting a CSS container to size itself correctly around the emulator's fixed-integer-zoomed canvas, across tall, wide, and rotated JARs.

## Running / Demo

```bash
cp .env.example .env   # set VITE_DEMO_VIDEO_URL
docker compose up --build
```

Then open the app at `http://localhost:<APP_PORT>` (the port Nginx publishes in `docker-compose.yml`).

See [`demo/demo-guide.md`](./demo/demo-guide.md) for the verified user flow (add a JAR → library → game details → play → exit) as covered by the project's e2e suite.

## Status

- **Deployment:** Live — multi-stage Docker build (Node build stage → nginx runtime stage), confirmed by `Dockerfile`, `docker-compose.yml`, and `nginx.conf` in the repository.
- **Live demo:** https://retrogamevault.akashbharnuke.tech

## Links

- Source repository: TODO — fill with actual repo URL (public or private-with-access-note, your call)
