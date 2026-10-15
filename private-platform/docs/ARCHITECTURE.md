# Architecture

PrivatePlatform is the private-plane half of a two-plane VPS architecture. The public plane (`PlatformVPS`, a separate repository, under `/srv/`) is untouched by this project and never shares config, processes, or listeners with it.

## Implemented today

```
Internet → UFW (firewall) → WireGuard (private VPN subnet) → Private Nginx (nginx-private.service) → Application (localhost)
```

- **WireGuard VPN hub** — VPS acts as the hub with a fixed hub address, owner devices get static addresses from a reserved range, enrolled via `scripts/wireguard/add-peer.sh`. One UDP port is opened at the firewall for WireGuard — the only port this project opens.
- **Private Nginx instance** — a second, fully independent Nginx process (own `nginx.conf`, own systemd unit `nginx-private.service`, own `sites-available`/`sites-enabled`), bound only to the WireGuard interface. Managed via `scripts/nginx/register-app.sh`.
- **Deployment convention** — each private application lives in its own repository, cloned on the VPS to `/opt/PROJECTS/<project_name>/`, running its own `docker-compose.yml`. PrivatePlatform never vendors application source.
- **Idempotent setup pipeline** — `scripts/setup.sh` runs six ordered, re-runnable steps (`00-preflight` through `05-validate`) to bring a fresh VPS to a working private-plane state without duplicating directories or regenerating existing keys.

See [diagrams/request-path.md](../diagrams/request-path.md) for the full layered diagram and the side-by-side with the public plane.

## Demonstrated onboarding

Two real sites have been registered end-to-end through this pipeline:

| Site | Project | Notes |
|---|---|---|
| WatchTower | `watchtower` | First consumer application |
| Grafana | `watchtower` (sub-service) | Registered under WatchTower's project via the script's `<site_name> <project_name>` form, since Grafana is a container inside WatchTower's own compose stack rather than its own top-level clone |

See [diagrams/onboarding-workflow.md](../diagrams/onboarding-workflow.md) for the registration flow and its validation guard.

## Planned, not yet built

These are stated intentions in the specs, not implemented components — listed here so the distinction is explicit:

- **TLS for private domains** — sites currently serve plain HTTP over the WireGuard tunnel. Certificate strategy (public CA, internal CA, or self-signed) is an open decision in `specs/005-dns-and-tls.md`.
- **App-to-app isolation enforcement** — the principle ("one private app shouldn't reach another unless explicitly allowed") is agreed in `specs/004`, but no firewall- or Nginx-level mechanism enforces it yet.
- **Secrets-management tooling** — WireGuard keys currently live as `chmod 600` plain files on the VPS filesystem. A dedicated secrets tool (vault/sops/age) is an open item across specs 001, 002, and 009.
- **Security hardening pass** — `specs/009-security-hardening.md` catalogs gaps (repo access control, a lesser-privileged admin role for day-to-day WireGuard/Nginx management) with no implementation started.
- **Architecture diagram render** — CLAUDE.md specifies that a rendered image of the full request-path diagram should be produced once at least one app is onboarded end-to-end through specs 002/003/006. That onboarding milestone is met (WatchTower); the rendered-image step itself had not been executed as of this showcase.

## Reusability goal

The explicit design intent (`docs/project-plan.md` §26, "Non-Goals") is that this stays a private-plane platform for N applications using one shared process — not a bespoke setup repeated per app, and not a general-purpose platform built for hypothetical future needs beyond what's actually required. The two-site registration (WatchTower + Grafana) through the same unmodified script is the current evidence for that goal; it has not yet been tested with a third, unrelated application.
