# Architecture Notes

## The integration boundary

Retro Game Vault is split into two parts with a hard, documented boundary between them:

- **The host app** — the React UI, IndexedDB-backed library/metadata, and a Runtime Adapter module (`app/src/runtimeAdapter.ts`). Owns the JAR file and all persistent state.
- **FreeJ2ME-Web** — a vendored, third-party J2ME emulator (CheerpJ-compiled Java, running as WebAssembly in the browser). Owns the actual game execution, save-state (RMS), and emulator configuration *while a game is running*.

The project's own rules (documented in the repo's `CLAUDE.md`) explicitly forbid building a second JAR loader or save-state system in the host app if the engine already has one, and restrict engine changes to a single file, with no Java source changes. That constraint shaped the entire integration: instead of duplicating engine functionality, the host drives the existing engine through the one communication channel browsers give two different origins/frames safely — `postMessage`.

## The contract (verified from `runtimeAdapter.ts`)

1. Host creates an iframe pointed at `/engine/run.html?hostDriven=1&fractionScale=1`.
2. Engine, once ready, posts `{ type: 'freej2me-ready' }`.
3. Host responds with `{ type: 'vault-install-jar', jar: <ArrayBuffer>, name, width?, height?, rotate? }`, transferring the JAR bytes directly (zero-copy, via the `Transferable` third argument to `postMessage`).
4. Engine installs the JAR into its own virtual filesystem and boots the game.
5. On exit, engine posts `{ type: 'freej2me-exit' }` — but the host also exposes its own independent `stop()` path (wired to a host-rendered Exit button) that works even if that message never arrives.

This is the entire integration surface. Nothing else crosses the iframe boundary.

## Why the engine's installed copy isn't a second source of truth

Because each Gameplay session installs the JAR fresh into a newly created iframe, the engine never accumulates state the host needs to track or reconcile. If the user plays Game A, exits, then plays Game B, the host doesn't ask the engine to "switch" — it destroys the whole iframe and builds a new one. This is simpler than it sounds, and it's the reason the project doesn't need any state-sync logic between host and engine: there's nothing to sync, because the engine's state never outlives a single session.

## Where this came from

This is not a spontaneous design — it's documented up front in `specs/engine-integration.md` and `docs/phase-0-summary.md` as the explicitly approved architecture before any implementation began, and the planning documents under `docs/superpowers/plans/` show the actual sequence of patches made to reach it (adding width/height, then rotation, then aspect-ratio-correct layout — each added incrementally and verified against real JARs before being folded into the design).
