# Architecture

Adapted from the project's own `docs/architecture/flow.md`, written after the
real Phase 6 migration and verified directly against script/config behavior
(not the original design plan). Diagrams: [`../diagrams/architecture.md`](../diagrams/architecture.md).

## Components

- **Public Nginx** — the only process terminating internet traffic for the
  public plane. Each project gets two config files: `<name>.conf` (port 80,
  ACME challenge + redirect to HTTPS) and `<name>-ssl.conf` (port 443, actual
  app serving — `root`/`try_files` for static, `proxy_pass` for compose).
- **Certbot** — issues/renews TLS certificates via `certonly --webroot` only.
  `certbot --nginx` is never invoked anywhere in the repo, by design (see
  Design Decisions).
- **Project metadata** (`projects/<name>.conf`) — one plain `key=value` file
  per project, the single source of truth for its type, domain, and paths.
  Scripts never take a `--type` flag; type always comes from this file.
- **Deployment scripts** (`deploy.sh`, `ssl.sh`, `status.sh`) — bash, no
  framework, each scoped to exactly one project per invocation.
- **Default catch-all** (`default.conf`) — explicit `default_server` on both
  80 and 443 that returns `444` for any unmatched `Host` header, closing the
  connection rather than falling through to any real site.

## Request routing

Nginx matches `server_name` against the five deployed projects' domains; an
unmatched request always lands on the catch-all, never on homepage or any
other site by accident. The 443 catch-all presents a self-signed snakeoil
certificate purely to complete the TLS handshake before closing — it is never
reachable by a request matching a real project's `server_name`.

Compose projects proxy to fixed local ports, hand-written into each
`-ssl.conf` file, one per deployed project — the one per-project value that
lives outside metadata (see Known Limitations in the main README).

## Deployment lifecycle

`deploy.sh <project>` dispatches on metadata `TYPE`:

- **static**: optional build command, copy output into `OUTPUT_DIR`.
- **compose**: `docker compose -f <COMPOSE_FILE> up -d --build`, confirmed by
  checking at least one container reports `running`.

Both paths converge on the same Nginx sequence: install the project's HTTP
site file (never a glob — exactly one file, named from metadata), `nginx -t`,
reload on success, `curl` health check. A failed `nginx -t` triggers
`rollback_nginx_site`, which undoes only the symlink/backup that specific
`install_nginx_site` call just created — every other project's state is
provably untouched (enforced structurally: no script contains a glob over
`*.conf`, and `tests/test_load_project_collision.sh` plus the isolation-test
procedure in `specs/10-testing.md` back this up operationally).

A failed health check is treated as an app-level problem, not a config
problem — Nginx is already valid and reloaded by that point, so there is
nothing to roll back; the script exits 1 and leaves the already-applied infra
changes in place.

## SSL lifecycle

`ssl.sh <project>` requires the project's HTTP site to already be enabled
(checked explicitly, aborts otherwise). It runs `certbot certonly --webroot`,
then installs the `-ssl.conf` file through the same install/validate/
reload/verify sequence as `deploy.sh`. If `nginx -t` fails after installing
the SSL file, only the Nginx symlink is rolled back — the certificate itself
is kept, since it was already issued successfully and Certbot's own
idempotency means re-running `ssl.sh` later does not re-trigger issuance.

Renewal runs entirely through Certbot's own systemd timer, outside this
repo's control; the one repo-adjacent artifact is a one-time VPS-side deploy
hook that reloads Nginx only after an actual renewal (never on a no-op run).

## Component ownership boundary

This repository owns the public plane only — public Nginx, public SSL/Certbot
triggers, public domain routing, and the platform homepage's static source.
It structurally cannot touch `/opt`, `PrivatePlatform`, WireGuard, private
Nginx, or any application repo's own code/Dockerfile/Compose/env/business
logic: `install_nginx_site`/`rollback_nginx_site` only ever read or write
`$NGINX_SITE`-derived filenames for the one project being deployed, and
`deploy.sh` never parses a compose file or names a service — its entire
compose-side contract is "build, bring up, check running," against whatever
the application repo already provides.

## Known limitations (verified during migration, not yet resolved)

- Compose proxy ports are hand-written per `-ssl.conf` file rather than a
  metadata field — the one inconsistency with the otherwise fully
  metadata-driven design.
- The renewal deploy-hook's presence on the live VPS cannot be verified from
  this repo alone — it's a documented one-time manual step, confirmed done
  during migration but not something repo state alone can prove.
- A health-check timing quirk (occasional single failed `curl` immediately
  after a reload, succeeding on an unmodified retry) is observed and
  documented, not root-caused — both scripts are safe to re-run when it
  happens.
