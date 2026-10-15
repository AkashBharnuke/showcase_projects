# Design Decisions

Each item below traces to a specific spec section or commit — no invented rationale. Where the reasoning is inferred rather than stated outright, it's marked as inference.

## 1. Separate Nginx instance, not shared config with the public plane

**Decision:** PrivatePlatform runs its own Nginx process (`nginx-private.service`), own `nginx.conf`, own `sites-available`/`sites-enabled`, bound only to the WireGuard interface (the VPN hub address). It never shares a listener or config root with the existing public-plane Nginx (`PlatformVPS`, under `/srv/`).

**Why (evidenced — CLAUDE.md, specs/003):** the two systems share one VPS but must never share failure domains. A misconfigured private site reload must not be able to touch the public Nginx instance, and vice versa. Commit `7b15fc8` ("Fix private Nginx binding to all interfaces instead of private-only") shows this boundary was violated once in practice and then corrected — a real bug, not a hypothetical.

**Alternative considered:** a single Nginx instance with server blocks split by `listen` address. Rejected (per spec 003) in favor of full process separation, trading some duplication for a harder isolation guarantee.

## 2. Port range convention is never a security boundary

**Decision:** private applications are assigned ports from a fixed numeric range (WatchTower and its Grafana sub-service each get one), but this numbering carries no access-control meaning on its own.

**Why (evidenced — project-plan.md §12, CLAUDE.md):** explicitly stated as a non-goal to treat port numbers as security. Actual privacy comes from the combination of firewall + WireGuard + Nginx + application auth. This matters because it's a common infra mistake — relying on "it's just an internal port" as if that were access control.

## 3. `register-app.sh` hard-rejects any non-private `listen` directive

**Decision:** the one reusable onboarding script greps every site config for its `listen` directives and refuses to symlink the site into `sites-enabled/` unless each one explicitly binds the WireGuard interface address.

**Why (evidenced — script logic, confirmed by Explore-agent read of `scripts/nginx/register-app.sh`):** this directly prevents the class of bug fixed in commit `7b15fc8` from recurring for any future app — instead of relying on a human to remember the rule every time, the registration script enforces it mechanically. This is the kind of guard that's cheap to add once and saves a repeat incident.

## 4. App source and deployment config are deliberately kept in different places

**Decision:** application source/compose files live at `/opt/PROJECTS/<project_name>/` on the VPS, cloned from each app's own independent repository. `PrivatePlatform`'s own `deployments/<app-name>/` directory (originally proposed in spec 001) is bookkeeping-only, largely unused in practice.

**Why (evidenced — spec 001 §5, explicitly annotated "confirmed during implementation — supersedes the original proposal"):** this is a spec that was corrected after real deployment experience, not guessed upfront. It keeps PrivatePlatform strictly an infra/deployment-config repo, never an application source host — enforced by the repo boundary rule in CLAUDE.md.

## 5. TLS and firewall-level app isolation are deliberately deferred, not skipped by oversight

**Decision:** private sites currently serve plain HTTP (bound to the WireGuard interface, no TLS directives) over the WireGuard tunnel. App-to-app isolation beyond "reachable only via Nginx" has no enforcement mechanism yet.

**Why (evidenced — spec 005 §16 OPEN items, spec 004 §17 OPEN items):** both specs explicitly flag the certificate strategy and the isolation-enforcement mechanism as unresolved decisions requiring further design work, rather than silently shipping an insecure default and calling it done. The project-plan (§21) explicitly warns against building generic infrastructure for hypothetical needs — so these stay open until a real second private app's requirements justify a specific mechanism, rather than guessing one now.

**Inference:** deferring TLS specifically (vs. e.g. firewall rules, which got one concrete UFW rule already) suggests the WireGuard tunnel's own encryption was judged sufficient for the current single-operator use case, with TLS termination treated as a separate, lower-urgency hardening pass. This reasoning is not stated outright in any spec — it is this report's inference from what was built first.

## 6. One reusable registration script, never a per-app variant

**Decision:** `scripts/nginx/register-app.sh` is explicitly required (CLAUDE.md) to be the only onboarding script, parameterized by `<project_name>` (and optionally a separate `<site_name>` for sub-services like Grafana) — never forked per application.

**Why (evidenced — CLAUDE.md "Never create a per-app variant of this script"; commit `b9d3ce1` adding the site-name/project-name split specifically to support Grafana without breaking this rule):** keeps onboarding a fixed, auditable procedure instead of accumulating bespoke scripts per app — directly serves the project's stated reusability goal.
