# Architecture

See [`../diagrams/system-architecture.md`](../diagrams/system-architecture.md) for the component diagram and [`../diagrams/telemetry-flow.md`](../diagrams/telemetry-flow.md) / [`../diagrams/health-rollup-flow.md`](../diagrams/health-rollup-flow.md) for data flow.

## Ecosystem position

WatchTower is one independent project in a larger VPS ecosystem plan (`docs/project-plan.md`), deliberately scoped away from adjacent responsibilities:

```text
PrivatePlatform   → private network, WireGuard, private Nginx, private routing
WatchTower        → observability, health, alerting, operational intelligence   (this project)
Applications      → business logic, application data, domain behaviour
Future Platform   → platform inventory, deployment history, broader control
Steward             (explicitly out of scope today)
```

WatchTower consumes PrivatePlatform's environment (reaches it via the Docker network and the fixed Grafana port PrivatePlatform's Nginx is expected to proxy) rather than implementing its own private networking. It does not own application business logic/data, and does not take on Platform Steward responsibilities such as deployment orchestration or backup ownership.

## Components

| Component | Role | Technology |
|---|---|---|
| API | Control/API layer: registry CRUD, health-state queries, `/metrics`, `/health` | FastAPI, served via `uvicorn` |
| Worker | Polls registered Services' `/health` endpoints on an interval, records raw results | Python asyncio loop (`health_check_executor.py`) |
| PostgreSQL | Stores WatchTower's own operational metadata only (Projects/Services/Environments/Dependencies/HealthCheckResults) | PostgreSQL 16, SQLAlchemy (async) + Alembic migrations |
| Prometheus | Metrics storage and query | Prometheus v2.55 |
| Loki | Log storage, 3-5 day retention | Loki 3.2 |
| Grafana | Dashboards, alert rule evaluation, notification delivery | Grafana 11.3 |
| Alloy | Telemetry collector: scrapes metrics, tails container logs, forwards both | Grafana Alloy v1.5 |
| Node Exporter | Host-level CPU/memory/disk/network metrics | prom/node-exporter v1.8 |
| cAdvisor | Container-level resource metrics | gcr.io/cadvisor v0.49 |
| pilot-app | Dev-only fixture proving the application observability contract (logs, `/health`, `/metrics`) | FastAPI, Compose `dev` profile only |

The metadata store (PostgreSQL via API/Worker) and the telemetry stack (Prometheus/Loki/Grafana/Alloy) are architecturally separate concerns, reflecting the project's operating principle: **use mature systems for telemetry collection/storage/visualization, build custom code only for the operational-intelligence layer around them** (registry, health evaluation, alert-bridging).

## Operational domain model

Implemented in `src/watchtower/registry/` (`models.py`, `crud.py`, `router.py`, `schemas.py`):

```text
Project
  └── Service (kind: long_running | run_once)
        ├── Environment (health_check_url, HealthCheckResult history)
        └── Dependency  (edges to other Services — recorded, not yet used by health evaluation)
```

Exposed via FastAPI routes for create/read on each entity, plus read-only state endpoints:

```text
GET /environments/{id}/state
GET /services/{id}/state
GET /projects/{id}/state
```

See [`health-rollup-flow.md`](../diagrams/health-rollup-flow.md) and [`DESIGN_DECISIONS.md`](DESIGN_DECISIONS.md#single-stage-flatten-not-two-stage-combine) for how these states are computed.

## Alerting & notification path

Grafana Alerting is the alerting engine (decided; no Alertmanager). Rules are provisioned as code in `deploy/grafana/provisioning/alerting/rules.yml` (group `watchtower-health`, folder `WatchTower`):

| Rule | Severity | `for` | Signal |
|---|---|---|---|
| `wt-project-down` | critical | 0s | `watchtower_health_state{scope="project"}` == DOWN |
| `wt-project-degraded` | warning | 120s | == DEGRADED |
| `wt-project-unknown` | warning | 120s | == UNKNOWN |
| `wt-stale-health-check` | warning | 120s | health-check freshness gauge too old |
| `wt-api-scrape-failure` | critical | 60s | Prometheus `up` for the WatchTower API target |
| `wt-pilot-app-scrape-failure` | critical | 60s | `up` for pilot-app target |
| `wt-infra-disk-low` | warning | 120s | node-exporter filesystem ratio < 10% |
| `wt-infra-memory-low` | warning | 120s | node-exporter memory ratio < 10% |
| `wt-infra-cpu-saturated` | warning | 120s | node-exporter idle-CPU ratio < 10% |

Notification channel: email via SMTP, using a dedicated Gmail account (2-Step Verification + App Password), configured through `deploy/grafana/provisioning/alerting/contact-points.yml` / `policies.yml`. Credentials are supplied via environment configuration, never committed — see `.env.example`.

**Not yet covered** (explicitly, not silently): Nginx/PostgreSQL/Redis availability alerts (no exporter exists for these in this repo) and application error-rate/latency/queue alerts (pilot-app exports only a request counter, no error/latency labels). Recorded as an open gap in `docs/TASKS.md` WT-014, not fabricated here.

## Dashboards

Provisioned as code in `deploy/grafana/dashboards/` and loaded via `deploy/grafana/provisioning/dashboards/dashboards.yml`:

- **`infrastructure.json`** — "WatchTower - Infrastructure": CPU/Memory/Disk/Network usage, running container count, container restart count, per-container CPU/memory, recent container logs.
- **`application-pilot-app.json`** — "WatchTower - Application: pilot-app": application status, request totals and rate by endpoint, application logs.

## Phased delivery

The project follows an explicit 8-phase roadmap (`docs/project-plan.md` §14, `specs/README.md`). As implemented in this repository:

```text
Phase 0 — Architecture & Foundation         ✓ complete
Phase 1 — Telemetry Foundation              ✓ complete
Phase 2 — Application Observability         ✓ complete
Phase 3 — WatchTower Operational Model      ✓ complete
Phase 4 — Health Engine                     ✓ complete
Phase 5 — Alerting & Notifications          ~ in progress (WT-014 alert rules done; WT-015 notification delivery setup tracked, not yet verified end-to-end per docs/TASKS.md)
Phase 6 — Operational Correlation           — not started (blocked: Incident/Event entity shape is an open spec question)
Phase 7 — Reliability & Production Hardening — not started (blocked: security/auth/secrets/backup decisions pending)
```

Status per `docs/TASKS.md` Backlog Summary (18 tasks tracked; WT-014 the most recently completed at time of writing).
