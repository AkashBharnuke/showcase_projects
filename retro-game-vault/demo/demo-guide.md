# Demo Guide

No screenshots are included in this package — no browser-automation tooling was available in the session that generated it, so none were captured, and none are fabricated here. The flow below is the verified user journey, based directly on the project's own Playwright end-to-end test (`app/e2e/gameplay.spec.ts`), not a guess at UX.

**TODO (manual):** capture 3-4 real screenshots — empty Library, populated Library, Game Details, and Gameplay running a game — and drop them into this folder, then reference them from this guide.

## Verified user flow

1. **Library (empty state)** — first load, no games added yet.
2. **Add Game** — click "Add Game," choose a `.jar` file. If the engine can't infer the screen resolution from the filename, optional width/height/rotation fields are available.
3. **Library (populated)** — the game now appears in the list and persists across a page reload (covered explicitly by the e2e suite's reload-and-recheck step).
4. **Game Details** — click the game to open its details (Play / Delete / Back).
5. **Gameplay** — click "Play." The FreeJ2ME-Web runtime loads inside an iframe and the game boots.
6. **Exit** — click the host's "Exit" control (or the engine exits itself) and land back on the Library, game still present.

Playing a second, different game immediately after exiting the first is also explicitly covered — the runtime iframe is destroyed and recreated cleanly between sessions, with both games remaining in the library afterward.

## Live instance

https://retrogamevault.akashbharnuke.tech

## Running it locally

```bash
cp .env.example .env   # set VITE_DEMO_VIDEO_URL
docker compose up --build
```

Open `http://localhost:<APP_PORT>` (the port Nginx publishes in `docker-compose.yml`).
