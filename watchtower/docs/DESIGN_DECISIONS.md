# Design Decisions

Decisions below are drawn from `specs/*.md`, `docs/TASKS.md` implementation notes, and commit history. Each links to its evidence. This is not an exhaustive ADR log — it covers the decisions with a real trade-off story behind them.

## Spec-driven development with explicit "Open Questions"

Every specification file (`specs/architecture.md`, `observability.md`, `operational-model.md`, `production-concerns.md`) ends with an **Open Questions** section, and `CLAUDE.md` defines a process: when implementation reaches an unresolved decision, stop, explain why it matters, present options, and get the decision recorded in the spec before continuing.

**Why it matters**: several decisions below (health rollup shape, alerting engine, notification channel) were deliberately *not* decided upfront — they were raised during implementation when the spec's abstraction met a concrete schema or integration gap, decided once, and written back into the spec. This avoided speculative design for decisions that were easier to make correctly once real constraints were visible.

## Single-stage flatten, not two-stage combine (health rollup)

`specs/operational-model.md` originally described Service-level health "from its own health checks," but the health-check-bearing field (`health_check_url`) actually lives on `Environment`, a Service's child. A Service can have zero, one, or many Environments.

Two ways to compute Project-level state existed:
1. Combine each Service's Environments into one Service-level value, then combine those Service-level values into a Project-level value (two-stage).
2. Flatten every non-`run_once` Service's Environments into one list and combine once (single-stage).

**Decision**: single-stage flatten (`get_project_state` in `src/watchtower/registry/health.py`). Two-stage combine would have implicitly introduced a Service-level aggregation rule the spec never defined — silently inventing scope. The single-stage version uses the one combine rule the spec does define, applied once. `GET /services/{id}/state` still exposes a Service's own combine of its Environments for direct inspection, but that value is not fed back into Project rollup.

A consequence, also decided explicitly rather than assumed: a Service with zero Environments contributes one `UNKNOWN` entry to the Project combine rather than being invisible to it.

Evidence: `docs/TASKS.md` WT-013 Implementation Notes; `specs/operational-model.md` Project-Level Rollup section.

## Service Lifecycle Kind (`long_running` vs `run_once`)

A health rollup based purely on "is the Service's latest check healthy" would permanently degrade a Project after a bootstrap/init container intentionally exits — nothing distinguished "exited by design" from "should be running but isn't."

**Decision**: add a `kind` field to Service (`long_running` default, `run_once`), and exclude every Environment of a `run_once` Service from Project rollup entirely. This was raised as a registry gap (`WT-012`) *before* the rollup logic (`WT-013`) was implemented against it, rather than patched in afterward.

Evidence: `docs/TASKS.md` WT-012; `specs/operational-model.md` Service Lifecycle Kind.

## Alerting engine: Grafana Alerting, not Alertmanager

**Decision**: Grafana Alerting is the alerting engine. A separate Alertmanager is not introduced "unless a concrete architectural constraint later requires it" — none was identified.

**Why**: Grafana is already the project's visualization layer and ships a capable native alerting engine (rule evaluation, grouping, deduplication, notification policies). Running Alertmanager alongside it would duplicate that machinery for no stated benefit — consistent with the project's stated philosophy of using mature systems as-is rather than rebuilding or doubling up on their capabilities.

Evidence: `specs/operational-model.md` Alerting & notification ownership (marked **Decided**).

## Bridging WatchTower's own health state into Grafana Alerting

Grafana Alerting natively evaluates Prometheus/Loki data; it has no built-in awareness of a health rollup computed in WatchTower's own FastAPI/PostgreSQL layer. This bridge did not exist by default and had to be designed.

**Decision**: export `HealthState` (per Project/Service/Environment) and a health-check-freshness timestamp as Prometheus gauges from the API's `/metrics` endpoint (`watchtower_health_state`, `watchtower_health_check_last_timestamp_seconds`), scraped by Alloy like any other target. Grafana Alerting rules then query these gauges exactly as they'd query any infrastructure metric — no custom bridge service, no second notification path.

**Trade-off accepted**: health state visible to Grafana is only as fresh as the last scrape interval, not instant on health-check completion. Judged acceptable given alert rule `for:` durations are already 60-120s.

Evidence: `src/watchtower/api/metrics.py`; `deploy/alloy/config.alloy`; `deploy/grafana/provisioning/alerting/rules.yml`; commits `8ca9395`, `e0e9041`, `6080593`, `30d8af1`.

## Threshold logic fix: raw value in query, comparison in Grafana's threshold stage

First draft of the infrastructure alert rules (disk/memory/CPU) put the `< 0.10` comparison directly inside the PromQL query. Caught during live verification: when the system is healthy, that query returns an *empty* result vector — indistinguishable from the metric being entirely unreachable (e.g. node-exporter down). Both cases silently produced "no data," defeating the point of `noDataState: Alerting`.

**Fix**: move the raw ratio into the query (always present when node-exporter is up, healthy or not) and the `< 0.10` comparison into Grafana's own threshold stage. Now `noDataState: Alerting` correctly means "the exporter is unreachable," not "things happen to be fine."

Evidence: `docs/TASKS.md` WT-014 Implementation Notes ("Design correction found during live verification").

## Network hardening: internal-only by default

An earlier commit (`ed8fd85`) added a `watchtower-shared` external network and host-published ports on most services, anticipating PrivatePlatform integration. A later audit (`WT-018`) found this exposure was speculative — no PrivatePlatform Nginx instance exists in this repository to integrate with yet, so the extra network and ports were pure attack surface with no present consumer.

**Decision**: removed `watchtower-shared`; removed host port publication from every service except Grafana, which keeps a fixed loopback-only mapping for the eventual reverse-proxy upstream. All internal service-to-service traffic uses the single `watchtower-internal` Docker network.

**Consequence accepted and documented, not silently absorbed**: removing Postgres's host port means the existing test suite's `localhost:<DB_PORT>`-based tests can no longer run directly from the host — they still pass from inside a container on the internal network, but there's currently no scripted containerized test runner. Recorded as an open gap in `docs/TASKS.md` (WT-018 Notes) rather than worked around.

Evidence: `docker-compose.yml` network/ports; `docs/TASKS.md` WT-018.

## What was deliberately not built

- **Custom WatchTower-side alert deduplication** — Grafana Alerting's native grouping is used instead; custom logic is explicitly deferred unless a concrete case proves it insufficient.
- **Alertmanager** — see above.
- **A private-network/reverse-proxy implementation inside WatchTower** — that's PrivatePlatform's ownership; WatchTower only documents the Grafana port it expects PrivatePlatform's Nginx to reach.
- **Dependency-based health attribution** ("API is unhealthy *because* Redis is down") — `Dependency` edges are modeled in the registry (WT-010) but intentionally not consulted by the rollup; the evaluation semantics remain an open question in `specs/operational-model.md`.
