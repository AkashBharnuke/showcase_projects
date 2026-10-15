# Technical Highlights

Three real bugs found and fixed while integrating with FreeJ2ME-Web — all from evidence in the repository's own planning docs (`docs/superpowers/plans/2026-09-24-game-screen-dimensions.md`) and the project README's "Engineering challenges" section, which this expands on.

## 1. A rotation bug invisible to automated tests

Some J2ME games render sideways relative to how a phone screen was meant to be held — the engine supports a "rotate display" setting for this. But feeding a rotated game both a width/height override and the rotate flag initially produced a distorted, clipped image rather than a correctly rotated one.

What made this one notable: every automated Playwright click/tap test on the affected games passed cleanly. The bug only showed up under manual testing with real mouse input against rotated games — because the distortion was in the engine's own coordinate handling during a 90°-rotated render, a code path synthetic test events never actually exercised in a way that would reveal visual distortion (tests assert on DOM visibility and bounding boxes, not pixel content). Root-causing it meant reading the emulator's own coordinate/scaling math directly and compensating on the host side, since the engine's Java source is explicitly out of scope to modify.

The project's e2e suite captures the final correct behavior as a regression test — `app/e2e/gameplay.spec.ts`'s "custom width/height/rotate renders at the correct aspect ratio" test measures the actual rendered container's aspect ratio against the expected rotated ratio within a small tolerance, rather than just asserting the iframe is visible.

## 2. A silent dropped-input bug

A related class of "the click just doesn't register" reports turned out to be the engine's own internal input event queue discarding anything that arrived before its polling loop had actually started — with no error, no console warning, nothing visible from the host side. This is the kind of bug that's easy to misdiagnose as a host-side timing issue (and was, initially) before tracing it into the engine's own queue implementation.

## 3. Aspect-ratio-correct container sizing

FreeJ2ME-Web always centers a fixed-size, integer-zoomed canvas inside whatever container it's given — it doesn't stretch to fill. Early on, a correctly-rendering game (right size, right orientation) would sit in the top-left corner of a much wider host container, leaving most of the box blank. The fix was entirely on the host side: give the runtime container an explicit CSS `aspect-ratio` computed from the game's actual stored (or default 240×320) dimensions, bounded by `max-width`/`max-height` so it still respects the surrounding layout. Getting this right across tall, wide, and rotated JARs took several real iterations — documented step-by-step in the project's own planning notes, including a specific case (`Asphalt6`) where the content rendered "correctly" by every other measure but still needed the rotate flag before its container's aspect ratio matched.

Each of these three fixes is host-only or confined to the single approved engine patch file — none required touching the emulator's Java source, consistent with the project's stated `app/`↔`engine/` boundary.
