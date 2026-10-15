# System Architecture

Component layout as implemented in `docker-compose.yml` and `deploy/`. All services share one Docker network (`watchtower-internal`); only Grafana is published to the host (loopback-only, for the upstream reverse proxy).

```mermaid
flowchart TB
    subgraph ext["Observed Workloads"]
        pilot["pilot-app\n(dev-only fixture)"]
    end

    subgraph wt["WatchTower — docker compose stack"]
        api["API\nFastAPI"]
        worker["Worker\nhealth-check executor"]
        pg[("PostgreSQL\noperational metadata")]
        alloy["Alloy\ntelemetry collector"]
        prom[("Prometheus\nmetrics")]
        loki[("Loki\nlogs")]
        grafana["Grafana\ndashboards + alerting"]
        node["Node Exporter"]
        cadvisor["cAdvisor"]
    end

    smtp[["Gmail SMTP\n(notification channel)"]]

    api -->|reads/writes| pg
    worker -->|polls /health, writes results| pg
    worker -->|/health polling| pilot
    api -->|/metrics scraped by Alloy| alloy
    pilot -->|/metrics, stdout logs| alloy
    node -->|host metrics| alloy
    cadvisor -->|container metrics| alloy
    alloy -->|remote_write| prom
    alloy -->|push| loki
    grafana -->|query| prom
    grafana -->|query| loki
    grafana -->|alert notifications| smtp

    classDef devOnly stroke-dasharray: 4 3
    class pilot devOnly
```

## Notes

- **Control/API layer** (`api`, `worker`, `postgres`) is WatchTower's own code — the operational model (Projects/Services/Environments/Dependencies, health rollup) lives here, not in Prometheus/Grafana.
- **Telemetry layer** (`prometheus`, `loki`, `grafana`, `alloy`, `node-exporter`, `cadvisor`) is intentionally off-the-shelf — per the project's stated philosophy, WatchTower does not reimplement metrics/log storage or visualization.
- `pilot-app` is a dev-only fixture (Docker Compose `dev` profile) used to prove the application observability contract end to end; it is not a production dependency.
- Grafana is the only service with a published host port, bound to loopback only and reserved for the private platform's reverse proxy. Every other service is reachable only on the internal Docker network — see [`../docs/DESIGN_DECISIONS.md`](../docs/DESIGN_DECISIONS.md#network-hardening).
