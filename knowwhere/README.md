# KnowWhere

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white)

KnowWhere is a location-aware web app that finds useful places (cafes, restaurants, hospitals, fuel stations) near the user's current location, shown on an interactive map and in a ranked nearby-places list.

## Overview

The problem it solves: quickly answering "what's around me right now?" without a heavyweight maps SDK or a paid places API. KnowWhere reads the browser's geolocation, lets the user pick a search radius and category, queries OpenStreetMap data through the Overpass API, and renders results with distance, address, and opening hours on a Leaflet map and a synced list.

Status: a completed, Docker-first full-stack project (frontend + backend + reverse proxy). No automated tests exist in the repository. Deployment status beyond local Docker Compose is `TODO:` — not evidenced in the repo.

## Tech Stack

- **Language(s):** TypeScript (frontend), Python (backend)
- **Frameworks/Libraries:** React + Vite, Leaflet / React-Leaflet, Lucide React icons · FastAPI, Pydantic, HTTPX
- **Infrastructure/Deployment:** Docker, Docker Compose, Nginx (reverse proxy + static file serving)
- **Testing:** none found in the repository (no test files, no test runner configured)

## Features

Implemented and verified directly in source:

- Browser geolocation lookup, with loading/error states (`frontend/src/hooks/useGeolocation.ts`)
- Configurable search radius: 500m / 1km / 2km / 5km (`backend/src/models/nearby.py`, `frontend/src/components/SearchControls.tsx`)
- Category filter: cafe, restaurant, hospital, fuel (same files)
- Interactive map with category-specific markers and map↔list selection sync (`frontend/src/components/MapView.tsx`, `mapIcons.ts`, `categoryIcons.ts`)
- Nearby-place list with distance, name, address, opening hours (`frontend/src/components/NearbyList.tsx`, `PlaceCard.tsx`)
- Loading / error / empty-result UI states (`frontend/src/App.tsx`, `useNearby.ts`)
- Backend result normalization from raw OSM data, including Haversine distance calculation and address construction from OSM tags (`backend/src/services/nearby_service.py`)
- In-memory TTL cache keyed on rounded lat/lon + radius + category, to avoid repeat Overpass calls (`backend/src/cache/memory.py`, `backend/src/controllers/nearby.py`)
- Per-IP in-memory rate limiting, 30 requests/minute (`backend/src/middleware/rate_limit.py`)
- Graceful handling of Overpass timeouts (504) and upstream failures (502) (`backend/src/controllers/nearby.py`)
- Dockerized frontend, backend, and Nginx reverse proxy via a single Compose stack (`compose.yaml`, `backend/Dockerfile`, `frontend/Dockerfile`, `nginx/nginx.conf`)

## Architecture

```text
Browser → Nginx (reverse proxy) → Frontend (React static build, served by its own Nginx)
                                 → Backend (FastAPI) → Overpass API → OpenStreetMap data
```

Backend request flow, confirmed in source (`routes/nearby.py` → `controllers/nearby.py` → `cache/memory.py` → `services/nearby_service.py` + `providers/overpass.py`):

```text
Route (/api/v1/nearby, query validation + rate limit check)
  → Controller (cache lookup, orchestrates provider call, maps exceptions to HTTP errors)
    → Cache (in-memory TTL, keyed on rounded lat/lon + radius + category)
    → Provider (Overpass query, httpx POST with 30s timeout)
    → Service (normalizes OSM elements into NearbyResult: distance via Haversine, address from addr:* tags)
```

Nginx (`nginx/nginx.conf`) routes `/api/` to the backend container and `/` to the frontend container — both frontend and backend stay on the internal Docker network and are not directly exposed.

See [diagrams/architecture.md](./diagrams/architecture.md) for the component/data-flow diagram.

## Key Engineering Decisions

- **In-memory cache and rate limiter instead of Redis/external services** — keeps the stack to three containers (backend, frontend, nginx) with no extra infra dependency. Evidenced by `backend/src/cache/memory.py` and `backend/src/middleware/rate_limit.py`, both plain in-process dicts with no external client. Trade-off (inference, not stated in code): state resets on restart and doesn't scale across multiple backend instances — acceptable for a single-container deployment.
- **Multi-stage Docker build for the frontend** — Node builds the static bundle, then only the Nginx-alpine runtime image ships, per `frontend/Dockerfile`. Keeps the runtime image small and avoids running Vite's dev server in production, as also stated in the project's own README.
- **Cache key rounds coordinates to 3 decimal places** (`round(lat, 3)` in `backend/src/controllers/nearby.py`) — trades exact-location cache precision (~110m grid) for a higher cache-hit rate on nearby repeat requests.

## Running / Demo

```bash
docker compose up --build
```

App and API are served from the same local port, with `/api/v1/` routed to the backend (per `compose.yaml` and the project README). No hosted demo or screenshots were available to capture in this session — see Known Limitations.

## Known Limitations / Not Implemented

- No automated tests (unit, integration, or e2e) exist in the repository.
- No CI pipeline (no `.github/workflows` or equivalent found).
- Cache and rate limiter are in-process and per-container — not shared across multiple backend replicas, and state is lost on restart.
- License field in the project's own README is explicitly left as a TODO ("This project's source code license should be defined here before public distribution").
- No automated OpenStreetMap attribution is rendered in the UI as of the reviewed source — the project README notes attribution "should be retained," but this wasn't verified as implemented in the frontend components reviewed.

## Status

- **Deployment:** TODO — not evidenced in the repository beyond a local Docker Compose setup; no hosting/CI config found.
- **Live demo:** none provided.

## Links

- Source repository: TODO — fill with actual repo URL
