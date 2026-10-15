# WatchTower

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F2CC60?logo=grafana&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

> **Status:** Actively Developed — Not Yet Deployed to Production

Operational observability platform for a personal VPS ecosystem — centralized metrics, logs, health evaluation, and alerting across applications and infrastructure, built around mature open-source telemetry systems rather than reimplementing them.

## Overview

Running several projects on one VPS raises a simple question: is everything actually working? WatchTower answers that without manually checking each project — it tracks what's running, what's healthy, and what failed, and when.

It uses Prometheus, Loki, and Grafana for the heavy lifting (collecting, storing, and visualizing telemetry) instead of reimplementing them. Custom code is limited to the operational-intelligence layer on top of those tools: a registry of projects/services/environments, health-state evaluation and rollup, and a bridge that feeds that health data into Grafana's own alerting.

Development follows a spec-driven workflow: a project plan sets direction, `specs/` define precise behavior and constraints, and `CLAUDE.md` requires that ambiguous decisions get raised and recorded rather than silently assumed. See [`docs/DESIGN_DECISIONS.md`](docs/DESIGN_DECISIONS.md) for real examples of that process in action.

**Status:** actively developed, not yet deployed to production infrastructure. Phases 0–4 (foundation, telemetry, application observability, operational model, health engine) are complete. Phase 5 (alerting) has alert rules implemented and live-verified; notification delivery is tracked but not yet confirmed end-to-end. See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md#phased-delivery) for the full phase breakdown.

## Tech Stack

- **Language:** Python 3.11+
- **Core application:** FastAPI (API), asyncio-based worker (health-check executor)
- **Database:** PostgreSQL 16, SQLAlchemy 2.0 (async), Alembic migrations
- **Telemetry:** Prometheus, Loki, Grafana 11, Grafana Alloy, Node Exporter, cAdvisor
- **Testing:** pytest, pytest-asyncio (6 test modules under `tests/`)
- **Deployment:** Docker Compose, multi-service network-isolated stack

## Features

- **Operational entity registry** — Projects, Services (with a `long_running`/`run_once` lifecycle kind), Environments, and Dependency edges, with CRUD APIs (`src/watchtower/registry/`).
- **Health-check execution** — a worker polls registered services' `/health` endpoints on an interval and records timestamped results (`src/watchtower/worker/health_check_executor.py`).
- **Health evaluation & rollup** — Environment states derived from the latest check result; Project-level rollup (HEALTHY/DEGRADED/DOWN/UNKNOWN) via a single-stage flatten-and-combine, with `run_once` services correctly excluded (`src/watchtower/registry/health.py` — see [design rationale](docs/DESIGN_DECISIONS.md#single-stage-flatten-not-two-stage-combine)).
- **Health-state → Prometheus bridge** — `HealthState` and check-freshness exported as Prometheus gauges from `/metrics`, scraped by Alloy, so Grafana Alerting can evaluate WatchTower's own operational model like any other metric (`src/watchtower/api/metrics.py`).
- **Infrastructure telemetry** — VPS CPU/memory/disk/network via Node Exporter, container resource usage via cAdvisor, centralized logs via Loki (3-5 day retention).
- **Application observability contract** — a pilot application (`pilot-app/`) demonstrates the full contract: structured stdout logs, `/health`, Prometheus-compatible `/metrics`.
- **Grafana Alerting as code** — 9 provisioned alert rules (project health states, API/pilot-app scrape failures, infra disk/memory/CPU thresholds) in `deploy/grafana/provisioning/alerting/rules.yml`, verified against a real stop/start failure cycle.
- **Provisioned dashboards** — infrastructure and application dashboards as JSON, loaded automatically on Grafana startup (`deploy/grafana/dashboards/`).

## Architecture

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for full component and boundary detail, and:

- [`diagrams/system-architecture.md`](diagrams/system-architecture.md) — components and how they connect
- [`diagrams/telemetry-flow.md`](diagrams/telemetry-flow.md) — metrics/log path from source to dashboard/alert
- [`diagrams/health-rollup-flow.md`](diagrams/health-rollup-flow.md) — how a `/health` poll becomes a Project-level state and an alert

## Key Engineering Decisions

See [`docs/DESIGN_DECISIONS.md`](docs/DESIGN_DECISIONS.md) for the full set, with evidence. Highlights:

- Single-stage flatten for health rollup, to avoid inventing an undefined Service-level aggregation rule.
- Grafana Alerting chosen over adding a separate Alertmanager — no concrete need identified for running both.
- A real "no data vs. healthy" bug in the alert threshold logic was caught by live verification against real Prometheus data, not just config review, and fixed by moving the comparison into Grafana's threshold stage.
- Docker network/port exposure was walked back after a self-audit found speculative exposure with no present consumer.

More engineering highlights (test coverage choices, live verification practices) in [`docs/TECHNICAL_HIGHLIGHTS.md`](docs/TECHNICAL_HIGHLIGHTS.md).

## Running / Demo

This is an infrastructure/platform project, not a user-facing app — there is no hosted demo. To run it locally:

```bash
docker compose up            # core stack: API, worker, Postgres, Prometheus, Loki, Grafana, Alloy, Node Exporter, cAdvisor
docker compose --profile dev up   # adds the pilot-app fixture
```

Grafana is reachable on loopback only once healthy. Requires a `.env` populated from `.env.example` (not included in this package — see the source repository).

## Status

- **Deployment:** not deployed to production infrastructure — developed and verified in a local Docker Compose environment.
- **Live demo:** none (infrastructure project; see Running / Demo above).

## Links

- Source repository: TODO — add link if/when this repository is made public or shared with the recipient.
