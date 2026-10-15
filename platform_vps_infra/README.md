# Platform VPS Infra

![Bash](https://img.shields.io/badge/Bash-4EAA25?logo=gnubash&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Certbot](https://img.shields.io/badge/Certbot-003A70)

Metadata-driven infrastructure for a single-VPS public platform hosting five
projects behind one Nginx instance — Nginx routing, SSL certificate
issuance/renewal, and zero-downtime deployment, driven entirely by one
plain-text metadata file per project.

## Overview

A single VPS serves five independent projects — two static sites and three Docker Compose applications — under one domain and its subdomains.

Instead of hand-editing Nginx and running ad hoc deploy commands per project, every project is defined by one `key=value` metadata file. Three scripts (`deploy.sh`, `ssl.sh`, `status.sh`) read that metadata and apply the same validated, rollback-safe sequence to any project, static or Compose-based, with no per-project branching logic outside the scripts themselves.

This is a completed redesign of a prior ad hoc version of the same system. The project's own `specs/07-safety.md` documents the specific historical incidents — a blind-overwrite deploy script, a duplicate `server_name` collision — that the current design was built to structurally prevent, not just patch.

**Status: migrated and live** — all five projects are deployed to the production VPS as of the completed Phase 6 migration (see `docs/guide/deployment-guide.md` in the source repo, adapted here as [`docs/OPERATIONS.md`](docs/OPERATIONS.md)).

## Tech Stack

- **Language:** Bash (all scripts), no framework or runtime dependency beyond
  what's already on a standard Linux VPS
- **Web/TLS:** Nginx (routing + TLS termination), Certbot (`certonly --webroot`
  mode only — never `certbot --nginx`)
- **Deployment targets:** static file output (two projects) and Docker
  Compose (three projects)
- **Networking:** UFW firewall (SSH + HTTP/HTTPS only)
- **Testing:** plain bash assertions (`tests/test_load_project_collision.sh`),
  `bash -n` syntax checks, `nginx -t` semantic validation — no test framework,
  matched to repo scale (single maintainer, no CI)
- **Source control:** Git as the sole rollback mechanism for routing config

## Architecture

Full breakdown: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) · Diagrams:
[`diagrams/architecture.md`](diagrams/architecture.md)

In short: the repo owns the **public plane only** — public Nginx, public SSL/Certbot triggers, domain routing, and the platform homepage's static source.

It structurally cannot reach `/opt`, a separate private platform, WireGuard, private Nginx, or any application repo's own code, Dockerfiles, env files, or business logic. `install_nginx_site`/`rollback_nginx_site` only ever touch the one named project's own filenames, and no script contains a glob over `*.conf`. An explicit `default_server` catch-all returns `444` for any request that doesn't match one of the five deployed projects' domains.

## Key Engineering Decisions

Full rationale with evidence: [`docs/DESIGN_DECISIONS.md`](docs/DESIGN_DECISIONS.md)

- **`certbot certonly --webroot` only, never `certbot --nginx`** — keeps 100%
  of routing config in Git with no out-of-band rewrites to reconcile.
- **No glob, anywhere, in any script** — a structural fix for a historical
  incident where a blind `cp *.conf` could overwrite every project's Nginx
  config in one run.
- **One metadata file per project, collision-checked** — `load_project`
  rejects two projects claiming the same `NGINX_SITE` before any file is
  written, closing a historical duplicate-`server_name` incident.
- **Health-check failure never triggers config rollback** — by that point
  Nginx is already valid and reloaded; a failed health check is treated as an
  app-level issue, not a config issue, so reverting would mask that routing
  actually succeeded.
- **No Kubernetes/Terraform/Ansible/custom CI** — deliberately scoped to
  Bash + Nginx + Compose + Certbot + Git for a single-maintainer, five-project
  VPS, not sized for hypothetical scale.

## Repository Structure

```
platform-vps-infra/
├── scripts/
│   ├── deploy.sh              deploy one project (static or compose)
│   ├── ssl.sh                 issue/enable HTTPS for one project
│   ├── status.sh               read-only status report for one project
│   ├── install-nginx.sh       one-time VPS bootstrap: install Nginx
│   ├── install-docker.sh      one-time VPS bootstrap: install Docker + Compose plugin
│   ├── setup-firewall.sh      one-time VPS bootstrap: configure UFW
│   └── lib/common.sh          shared helpers: load_project, nginx install/rollback
├── projects/                  one metadata file per project
├── nginx/sites-available/     committed Nginx configs (HTTP + HTTPS pairs)
├── platform-homepage/         homepage's own static source
├── tests/                     plain-bash assertion scripts
└── specs/                     design/specification record
```

## Status

- **Deployment:** live — five projects migrated to production VPS (Phase 6
  complete), verified via the operator deployment guide's worked examples.
- **Live demo:** not applicable — this is infrastructure tooling, not a
  user-facing application. The platform it deploys (`akashbharnuke.tech` and
  its subdomains) is the production outcome of this repo's scripts.

## Links

- Source repository: TODO — fill with actual repo URL
