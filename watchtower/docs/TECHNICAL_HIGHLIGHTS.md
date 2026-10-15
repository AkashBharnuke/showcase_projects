# Technical Highlights

## Spec-driven workflow with an enforced "stop and ask" rule

`CLAUDE.md` defines a strict source-of-truth hierarchy (`docs/project-plan.md` → `specs/` → implementation) and requires that any ambiguous or undecided requirement be surfaced and resolved — in the spec, not silently in code — before implementation proceeds. Multiple real decisions in this repo followed that path end-to-end: raised during implementation, options presented, decided, written back into `specs/operational-model.md`'s "Open Questions" / "Decided" sections, then implemented. See [`DESIGN_DECISIONS.md`](DESIGN_DECISIONS.md) for concrete instances (health rollup shape, alerting engine choice, the health-state-to-Grafana-Alerting bridge).

## Health rollup with a provable, test-covered combine rule

`src/watchtower/registry/health.py` implements a small, pure state machine (`evaluate_environment_state`, `combine_states`) rather than an ad hoc "degraded if something's wrong" heuristic. The rollup rule is written once, applied consistently at both Service- and Project-level inspection, and the edge cases that make rollups easy to get subtly wrong are explicitly covered: a `run_once` service's exit must not count as a failure; a service with zero environments must still contribute an `UNKNOWN` rather than vanishing from the combine. `tests/test_health.py` has 28 cases targeting exactly these boundaries (per `docs/TASKS.md` WT-013 Implementation Notes).

## Caught a real "no data" vs "healthy" bug during live verification, not code review

The first draft of the infrastructure alert rules (disk/memory/CPU) put the threshold comparison inside the PromQL query itself. That meant a healthy system and an unreachable node-exporter both produced an empty result vector — Grafana's `noDataState: Alerting` couldn't tell them apart. This was caught by actually querying live Prometheus data during verification (not by inspection), and fixed by moving the raw ratio into the query and the comparison into Grafana's own threshold stage. Documented with the exact query that exposed it in `docs/TASKS.md` WT-014.

## Alert rules verified against a real failure/recovery cycle, not just schema validation

Rather than stopping at "the YAML loads," verification for WT-014 brought up the real Compose stack, stopped `pilot-app`, confirmed the corresponding Grafana Alerting rule transitioned to `state:"Alerting"` with the correct severity via both Grafana's rule-evaluation API and its Alertmanager-compatible `/v2/alerts` endpoint, then restarted the container and confirmed recovery. Exact commands and API responses are recorded in `docs/TASKS.md`.

## Network exposure walked back after an audit, not left to "probably fine"

An earlier commit added a second Docker network and host-published ports across most services, anticipating a future PrivatePlatform integration. A later self-audit (`WT-018`) found this was speculative exposure with no present consumer, and removed it — reducing published host ports to just Grafana (loopback-only, for the eventual reverse-proxy upstream). The audit also surfaced and documented a real consequence (host-run integration tests losing direct Postgres access) rather than quietly working around it.

## Metadata/telemetry boundary enforced structurally, not just by convention

PostgreSQL is explicitly scoped to WatchTower's *own* operational metadata (Projects, Services, Environments, Dependencies, HealthCheckResults) — never application business data — a constraint stated in three separate places (`docs/project-plan.md`, `specs/architecture.md`, `CLAUDE.md`) and reflected in the schema itself (`src/watchtower/registry/models.py` has no entity resembling application domain data). Metrics/log storage and visualization are deliberately left to Prometheus/Loki/Grafana rather than rebuilt — the custom code in this repo is scoped to the operational-intelligence layer (registry, health evaluation, the metrics-export bridge), consistent with the project's stated philosophy of not rebuilding mature observability systems.
