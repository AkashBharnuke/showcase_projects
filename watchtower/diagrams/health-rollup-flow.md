# Health Evaluation & Alerting Flow

How a raw `/health` poll becomes a `HealthState`, rolls up to Project level, gets exported as a metric, and can fire a Grafana alert. This is WatchTower's own operational-intelligence layer — the part the project deliberately does not delegate to Prometheus/Grafana.

```mermaid
flowchart TD
    worker["Worker\nhealth_check_executor.py\npolls Environment.health_check_url on interval"]
    result[("HealthCheckResult\nstatus_code / error, timestamped")]
    envstate["evaluate_environment_state()\nlatest result only"]
    svcstate["get_service_state()\ncombine_states() over a Service's Environments\n(own-inspection only)"]
    projstate["get_project_state()\nflatten all non run_once Environments,\nthen combine_states()"]
    metricsep["/metrics endpoint\nwatchtower_health_state gauge\nwatchtower_health_check_last_timestamp_seconds"]
    alloy["Alloy scrapes API"]
    prom[("Prometheus")]
    rules["Grafana Alerting rules\n(wt-project-down / -degraded / -unknown / -stale-health-check)"]
    notify["Gmail SMTP notification"]

    worker -->|HTTP GET| svc{{"Registered Service's\n/health endpoint"}}
    svc --> result
    result --> envstate
    envstate -->|per Environment| projstate
    envstate -.->|per Environment, own Service only| svcstate
    projstate --> metricsep
    svcstate --> metricsep
    metricsep --> alloy --> prom --> rules --> notify
```

## Rollup rule (as implemented, `src/watchtower/registry/health.py`)

```text
Environment state  = f(latest HealthCheckResult only — no history window)
                      2xx           -> HEALTHY
                      non-2xx/error -> DOWN
                      no URL / no result yet -> UNKNOWN

Project state       = combine_states(
                         flatten of all non-run_once child Services' Environment states
                       )
                       all HEALTHY           -> HEALTHY
                       all DOWN              -> DOWN
                       some (not all) DOWN   -> DEGRADED
                       no DOWN, some UNKNOWN -> UNKNOWN
```

Key design point (see [`../docs/DESIGN_DECISIONS.md`](../docs/DESIGN_DECISIONS.md#single-stage-flatten-not-two-stage-combine)): Project rollup flattens directly over Environments rather than pre-combining each Service into one value first — a two-stage combine would have silently invented a Service-level aggregation rule not present in the spec. `run_once` Services (e.g. bootstrap/init containers) are excluded entirely from rollup since their exit is not a failure signal.

Dependency edges (`Service` → `Service`) are recorded in the registry but are **not** consulted by this rollup — dependency-based attribution ("API is unhealthy *because* Redis is down") remains an open question, tracked in `specs/operational-model.md`.
