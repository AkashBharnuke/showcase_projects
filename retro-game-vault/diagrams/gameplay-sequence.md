# Gameplay Session Sequence

Editable Mermaid source, based directly on `app/src/runtimeAdapter.ts`'s message handling.

```mermaid
sequenceDiagram
    participant U as User
    participant H as React Host
    participant A as Runtime Adapter
    participant E as FreeJ2ME-Web (iframe)

    U->>H: Click "Play" on a library game
    H->>A: startRuntime(jarBlob, name, dimensions?, rotate?)
    A->>E: Create iframe, src=/engine/run.html?hostDriven=1
    E->>A: postMessage("freej2me-ready")
    A->>E: postMessage("vault-install-jar", jarBuffer, name, width?, height?, rotate?)
    E->>U: Game boots and renders in iframe
    U->>E: Plays game
    alt User exits from engine's own control
        E->>A: postMessage("freej2me-exit")
        A->>H: onExit("engine-signal")
    else User exits via host fallback control
        U->>H: Click host-level "Exit" button
        H->>A: stop()
        A->>E: Remove iframe from DOM
    end
    H->>U: Back to Library, game still persisted in IndexedDB
```

## Why dual-signaled exit matters

The engine can tell the host it exited (`freej2me-exit`), but the host never depends on that message alone — a host-level Exit control works independently of engine cooperation. If the engine hangs or never fires the message, the user is never stuck inside a game with no way out.
