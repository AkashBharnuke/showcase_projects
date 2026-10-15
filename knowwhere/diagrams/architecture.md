# KnowWhere Architecture

Component and request-flow diagram, derived from `compose.yaml`, `nginx/nginx.conf`, and the backend source tree (`backend/src/routes`, `controllers`, `cache`, `services`, `providers`).

```mermaid
flowchart TD
    Browser["Browser"]
    Nginx["Nginx reverse proxy\n(container: nginx)"]
    Frontend["Frontend\nReact + Vite, built static,\nserved by its own Nginx\n(container: frontend)"]
    Backend["Backend API\nFastAPI\n(container: backend)"]
    Overpass["Overpass API\n(overpass-api.de)"]
    OSM["OpenStreetMap data"]

    Browser --> Nginx
    Nginx -- "/ " --> Frontend
    Nginx -- "/api/" --> Backend
    Backend --> Overpass
    Overpass --> OSM
```

## Backend request flow

```mermaid
flowchart LR
    Route["Route\n/api/v1/nearby\nquery validation, rate-limit check"]
    Controller["Controller\ncache lookup, orchestration,\nexception to HTTP mapping"]
    Cache["Cache\nin-memory TTL\nkey: rounded lat/lon + radius + category"]
    Service["Service\nnormalize OSM elements,\nHaversine distance, address build"]
    Provider["Provider\nOverpass query via httpx"]

    Route --> Controller
    Controller --> Cache
    Controller --> Provider
    Provider --> Service
    Service --> Controller
```

Notes:
- Both diagrams are Mermaid source only — no local Mermaid renderer (`mmdc`) was available in this session, so no PNG/SVG was generated.
- Cache and rate limiter are both in-process Python dicts (`backend/src/cache/memory.py`, `backend/src/middleware/rate_limit.py`) — no Redis or external store involved.
