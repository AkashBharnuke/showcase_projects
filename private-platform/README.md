# PrivatePlatform

![Bash](https://img.shields.io/badge/Bash-4EAA25?logo=gnubash&logoColor=white)
![WireGuard](https://img.shields.io/badge/WireGuard-88171A?logo=wireguard&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

> **Status:** In Active Development — Not Production-Complete

An in-progress private infrastructure layer for a personal VPS: WireGuard VPN, a dedicated Nginx instance, and firewall rules that let self-hosted applications stay off the public internet while remaining reachable from the owner's own devices. Built as the reusable foundation under a self-hosted monitoring app (WatchTower), designed to onboard further private applications the same way.

Core networking and routing are implemented and have onboarded two real services; TLS, firewall-level app isolation, and a formal secrets-management tool remain open design decisions, tracked explicitly rather than skipped silently. See [Status](#status) below for the full breakdown.

## Overview

Most personal VPS setups end up with every self-hosted tool exposed on the public internet, protected by nothing but a login page. That's because correctly wiring up a VPN, a second reverse proxy, and firewall rules — in a way that repeats cleanly for the next app — is more setup than most one-off projects bother with.

PrivatePlatform is that setup, built once as a dedicated system instead of being bolted onto each app individually. It provides a private network boundary (WireGuard) and a private routing layer (a second, fully isolated Nginx instance). Any future private application can onboard into it through one script, instead of each app inventing its own private-access story.

The project runs spec-first: nine numbered specs (`specs/001`–`009`) state what's confirmed, proposed, or still open before implementation, and get amended with resolution notes once real deployment settles a question. That loop is visible in the repository history — see [docs/DESIGN_DECISIONS.md](./docs/DESIGN_DECISIONS.md) for concrete examples, including one real misconfiguration (Nginx briefly bound to a public interface) that was caught, fixed, and then turned into a mechanical guard in the onboarding script so it can't recur.

## Tech Stack

- **Languages:** Bash (all setup/management scripts)
- **Infrastructure:** WireGuard (VPN), Nginx (dedicated private instance, separate from the public-plane Nginx), UFW (firewall), systemd (service persistence for Nginx/WireGuard), Docker (container runtime for onboarded applications)
- **Deployment target:** a single Hostinger KVM VPS running both a public plane (separate repository, untouched by this project) and this private plane, side by side
- **Documentation approach:** numbered specs with explicit CONFIRMED / PROPOSED / OPEN status per decision, plus an operating-guide (`CLAUDE.md`) for AI-assisted development sessions

## Architecture

Full breakdown — implemented vs. planned, with diagrams — in [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md).

In short: `Internet → UFW → WireGuard (private VPN subnet) → private Nginx instance → application`. Every layer is independent by design — none substitutes for another. The private plane never shares a config file, process, or listener with the VPS's existing public-facing Nginx.

```mermaid
flowchart LR
    A[Internet] --> B[UFW] --> C[WireGuard<br/>private subnet] --> D[Private Nginx] --> E[Private apps]
```

See also:
- [diagrams/request-path.md](./diagrams/request-path.md) — full layered diagram, side-by-side with the public plane.
- [diagrams/onboarding-workflow.md](./diagrams/onboarding-workflow.md) — how a new application gets registered.
- [diagrams/spec-implementation-status.md](./diagrams/spec-implementation-status.md) — maturity of each of the nine specs.

## What's implemented and demonstrated today

- **Idempotent VPS setup pipeline** (`scripts/setup.sh` → six ordered steps): preflight checks, directory scaffolding, private Nginx deployment with its own systemd unit, WireGuard keypair generation and interface setup, a single scoped firewall rule, and a read-only post-setup validation pass. Safe to re-run — never duplicates directories, never regenerates existing keys.
- **WireGuard peer enrollment** (`scripts/wireguard/add-peer.sh`) — idempotent static-IP assignment for owner devices, live interface reload without dropping existing peers.
- **Application onboarding script** (`scripts/nginx/register-app.sh`) — the one reusable registration path for every private app: validates the app's project directory and Nginx config exist, **hard-rejects any site config that doesn't explicitly bind the private interface** (the exact mistake that caused a real incident, now caught mechanically), symlinks it into the private `sites-enabled/`, validates the full Nginx config, and reloads only the private Nginx service.
- **Two real applications registered** through that script: WatchTower (the first consumer app) and Grafana (a sub-service inside WatchTower's own stack, registered under a distinct site name to prove the script handles that case too).

## Key Engineering Decisions

Full write-up with evidence in [docs/DESIGN_DECISIONS.md](./docs/DESIGN_DECISIONS.md). Highlights:

- Private Nginx runs as a fully separate process from the public-plane Nginx — no shared config root, no shared listener — specifically because a misconfigured private-site reload must never be able to affect public traffic.
- The port-range convention used for private applications is explicitly documented as organizational only, never a security boundary — real privacy comes from the full layered chain (firewall + VPN + Nginx + app auth) together.
- Application source and PrivatePlatform's own deployment bookkeeping were deliberately kept in different filesystem locations after an early spec's original proposal was found impractical during real deployment — and the spec was corrected to match, rather than left stale.
- TLS and firewall-level app isolation are open items by design decision, not oversight — deferred until a second private application's actual requirements justify a specific mechanism, instead of guessing one now for a single-operator setup.

## Current Limitations and Known Gaps

Being direct about what isn't done, per the project's own specs:

- **No TLS yet** — private sites serve plain HTTP over the WireGuard tunnel. Certificate strategy is an explicitly open decision (`specs/005-dns-and-tls.md`).
- **No app-to-app isolation enforcement** — the principle is agreed, but no firewall- or Nginx-level mechanism blocks one private app from reaching another yet (`specs/004`).
- **No dedicated secrets-management tool** — WireGuard private keys currently live as permission-restricted (`chmod 600`) plain files on the VPS filesystem; a vault/sops/age-style tool is still an open item across multiple specs.
- **Security-hardening pass not started** — `specs/009-security-hardening.md` catalogs gaps (repo access control, a lower-privileged admin role) with no implementation yet.
- **Rendered architecture-diagram image not produced** — the project's own CLAUDE.md calls for generating one once an app is onboarded end-to-end; that onboarding milestone is met, but the image itself hadn't been generated as of this showcase (the Mermaid diagrams in this package are a stand-in, not that deliverable).
- **Only exercised with one application family so far** — WatchTower (plus its Grafana sub-service) is the only real test of the reusable-onboarding goal; a second, unrelated application hasn't yet gone through the pipeline to confirm the convention holds beyond its first user.

## Next Meaningful Milestones

In rough priority order, based on what the specs mark as open and what blocks further reuse:

1. Decide and implement a TLS strategy for private subdomains (closes `specs/005`'s main open item).
2. Implement a concrete app-to-app isolation mechanism (firewall or Nginx-level) and update `specs/004` from design-only to implemented.
3. Onboard a second, unrelated private application through the existing `register-app.sh` pipeline unchanged — the real test of the reusability goal.
4. Pick and adopt a secrets-management mechanism for WireGuard keys, closing the open item carried since spec 001.
5. Generate the rendered architecture-diagram image called for in `CLAUDE.md`, now that the onboarding milestone it depends on is met.

## Status

- **Deployment:** implemented and running on a real personal VPS for the setup/networking/routing layer described above; not a packaged or installable product, and not claimed as production-hardened.
- **Verified in this review:** directory structure, script logic, Nginx/WireGuard configs, and spec documents were read directly from the repository. Script *behavior on a live VPS* (e.g. the rollback path in `register-app.sh`, systemd unit activation) is implemented and was exercised for the two real site registrations described above, but was not independently re-run as part of preparing this showcase — treat "implemented" above as "implemented and committed," distinct from a fresh end-to-end re-verification.
- **Live demo:** none — this is private, VPN-gated infrastructure by design, not a publicly browsable service.

## Links

- Source repository: TODO — insert link if/when this repository is made available to reviewers.
