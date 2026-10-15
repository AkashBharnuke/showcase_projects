# Architecture

Editable Mermaid source. Renders automatically on GitHub and most Markdown viewers.

```mermaid
flowchart TD
    User["User's Browser"]
    Host["React Host App<br/>Library / Game Details / Gameplay screens"]
    DB["IndexedDB<br/>JAR files + metadata + library state<br/>(source of truth)"]
    Adapter["Runtime Adapter<br/>(runtimeAdapter.ts)<br/>owns iframe lifecycle + postMessage contract"]
    Iframe["iframe: FreeJ2ME-Web<br/>CheerpJ-compiled J2ME engine<br/>(vendored, one approved patch file)"]
    Game["Installed .jar<br/>(derived runtime copy, not source of truth)"]

    User --> Host
    Host <--> DB
    Host --> Adapter
    Adapter -- "postMessage: vault-install-jar" --> Iframe
    Iframe -- "postMessage: freej2me-ready / freej2me-exit" --> Adapter
    Iframe --> Game

    classDef host fill:#1d4ed8,color:#fff,stroke:none;
    classDef engine fill:#b45309,color:#fff,stroke:none;
    classDef store fill:#047857,color:#fff,stroke:none;
    class Host,Adapter host;
    class Iframe,Game engine;
    class DB store;
```

## Notes

- The host app and the vendored engine communicate **only** through the `postMessage` contract shown above — this is the entire integration surface, per the project's `app/`↔`engine/` boundary rule.
- IndexedDB is the single source of truth for which games exist and their metadata. The engine's own installed JAR (inside its virtual filesystem) is always a disposable, re-created-per-session copy.
- No in-place game switching: a new Gameplay session always destroys the previous iframe and creates a fresh one.
