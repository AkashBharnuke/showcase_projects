# Telemetry Flow (Metrics & Logs)

How a metric or log line gets from a workload to a Grafana dashboard or alert. Traces are not implemented in this phase (project plan Phase 6, not yet started).

```mermaid
flowchart LR
    subgraph sources["Sources"]
        api_m["WatchTower API\n/metrics (HealthState gauges)"]
        pilot_m["pilot-app\n/metrics"]
        host["Node Exporter\n(CPU/mem/disk/net)"]
        containers["cAdvisor\n(container stats)"]
        logs["Container stdout\n(all services)"]
    end

    alloy["Grafana Alloy\n(scrape + push)"]
    prom[("Prometheus\ntime-series storage")]
    loki[("Loki\nlog storage, 3-5 day retention")]
    grafana["Grafana"]
    alertrules["Alert Rules\n(watchtower-health group)"]
    smtp[["Gmail SMTP"]]

    api_m -->|scrape| alloy
    pilot_m -->|scrape| alloy
    host -->|scrape| alloy
    containers -->|scrape| alloy
    logs -->|collect| alloy

    alloy -->|remote_write| prom
    alloy -->|push| loki

    prom --> grafana
    loki --> grafana
    prom --> alertrules
    alertrules -->|fires| grafana
    grafana -->|notification policy| smtp
```

## What actually flows today

| Signal | Producer | Path | Consumer |
|---|---|---|---|
| Infra metrics (CPU/mem/disk/net) | Node Exporter | Alloy → Prometheus | Infrastructure dashboard, 3 alert rules |
| Container metrics | cAdvisor | Alloy → Prometheus | Infrastructure dashboard (container panels — see caveat below) |
| `watchtower_health_state` gauge | WatchTower API `/metrics` | Alloy scrape → Prometheus | 3 alert rules (project down/degraded/unknown) |
| `watchtower_health_check_last_timestamp_seconds` gauge | WatchTower API `/metrics` | Alloy scrape → Prometheus | Staleness alert rule |
| pilot-app request counter | pilot-app `/metrics` | Alloy scrape → Prometheus | Application dashboard, scrape-liveness alert |
| Container/application logs | stdout | Alloy → Loki | Log panels in both dashboards |

**Known environment caveat**: cAdvisor cannot resolve per-container stats on Docker Desktop (Windows/Mac) because its engine uses containerd snapshotters rather than the classic dockerd overlayfs layout cAdvisor expects. Host-level cgroup metrics still work; real per-container CPU/memory/restart data is expected to resolve on a native Linux Docker host (the target VPS). Documented in `docker-compose.yml`'s `cadvisor` service comment.
