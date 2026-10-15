# Design Decisions

Decisions with evidenced rationale from `specs/`, commit history, and the
post-migration review docs. Labeled where the reasoning is the project's own
stated rationale vs. an inference from the evidence.

## Certbot: `certonly --webroot` only, never `certbot --nginx`

**Decision:** every certificate issuance in the repo uses
`certbot certonly --webroot -w /var/www/certbot-challenge -d <domain>`.
`certbot --nginx` does not appear anywhere.

**Why (stated):** `certbot --nginx` rewrites live Nginx config in place,
which creates drift between what Git tracks and what's actually running.
Keeping Certbot scoped to writing only certificate files under
`/etc/letsencrypt/` means Git remains the single source of truth for 100% of
routing config, with no out-of-band edits to reconcile.

**Trade-off:** more manual wiring (the repo has to carry both an HTTP-only
config and a separate HTTPS config per project, enabled in sequence) in
exchange for configs that are fully auditable and revertible through normal
`git checkout`.

## Metadata-driven projects, one file per project

**Decision:** every project is defined by exactly one `projects/<name>.conf`
(`NAME`, `TYPE`, `DOMAIN`, `NGINX_SITE`, `PROJECT_DIR`, plus type-specific
fields). Scripts take only a project name — never a `--type` flag or other
override.

**Why (stated):** removes a second identity path for the same project. The
collision check in `load_project` (added `PVS-020`, covered by
`tests/test_load_project_collision.sh`) rejects two metadata files claiming
the same `NGINX_SITE` before any file is written, directly addressing a
historical incident (`retro_game_vault.conf` + `retrogamevault.conf` both
defining the same `server_name` — documented in `specs/07-safety.md`).

**Trade-off (inference):** the compose proxy port is the one value that
didn't make it into metadata — it's hand-written per `-ssl.conf` file. The
project's own architecture review (`docs/architecture/flow.md`) flags this as
an inconsistency worth resolving if a sixth compose project is added, rather
than treating it as settled.

## No glob, anywhere, in any script

**Decision:** every Nginx-file-touching operation (`install_nginx_site`,
`rollback_nginx_site`) is scoped to exactly one filename, derived from the one
project's metadata. Structurally verified: no script contains a glob over
`*.conf`.

**Why (stated):** directly traces to a historical failure mode documented in
`specs/07-safety.md` — the legacy `deploy-nginx-configs.sh` used
`cp *.conf /etc/nginx/sites-available/`, a blind overwrite capable of
clobbering every project's config in one run. The redesign's core safety
property is that deploying project A can never change project B's files,
enforced by this one structural rule rather than by convention or code
review.

## Explicit `default_server` catch-all, always enabled

**Decision:** `nginx/sites-available/default.conf` defines `server_name _`
blocks on both port 80 and 443 with `default_server`, returning `444` for any
unmatched request. Present from early in the redesign, not conditional on any
project being deployed.

**Why (stated):** without an explicit catch-all, Nginx falls back to
whichever server block was defined first — meaning an unrelated or
misdirected request (including a private/internal hostname reaching public
Nginx by mistake) could silently be served by homepage or another real
project's config. `return 444` closes the connection instead.

## Bash + Nginx + Docker Compose + Certbot + Git only — no IaC, no orchestration

**Decision:** explicitly out of scope per `specs/00-overview.md`: Kubernetes,
Terraform, Ansible, a deployment database, generic orchestration, custom
CI/CD.

**Why (stated):** single-maintainer VPS running five projects. The project
treats this as a deliberate scale-matched choice, not an oversight — tooling
sized to the actual operational surface rather than anticipating growth that
hasn't happened.

## Health-check failure does not trigger rollback

**Decision:** in both `deploy.sh` and `ssl.sh`, a failed final `curl` health
check exits 1 but does **not** roll back the Nginx config change that
preceded it.

**Why (stated):** by the time the health check runs, `nginx -t` has already
passed and Nginx has already reloaded — the config itself is valid. A health
check failure at that point is classified as an application-level problem
(the app isn't responding correctly), not a configuration problem, so
reverting the config would not fix the actual issue and would mask that the
routing change did succeed.
