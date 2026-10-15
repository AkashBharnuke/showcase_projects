# Operations

Condensed from the project's own `docs/guide/deployment-guide.md`, written
after the completed Phase 6 migration (all five projects verified live).

## The three operator commands

| Script | Purpose | Touches |
|---|---|---|
| `deploy.sh <project>` | Build/start one project, install its HTTP Nginx site | One project's output dir/containers + one Nginx file |
| `ssl.sh <project>` | Issue/enable HTTPS for one project | Certbot + one Nginx SSL file |
| `status.sh <project>` | Read-only health/state report | Nothing — never deploys, reloads, or issues certs |

Every script takes exactly one argument, the project name. There is no
`--type` flag anywhere — deployment type always comes from that project's own
metadata file.

## Worked troubleshooting case: `knowwhere`'s compose filename

During the real migration, `deploy.sh knowwhere` failed cleanly with "no such
file" because `projects/knowwhere.conf` still listed `COMPOSE_FILE=docker-compose.yml`,
while the actual file on disk was `compose.yaml`. The fix was a one-line
metadata correction (`COMPOSE_FILE=compose.yaml`) — `deploy.sh` itself never
assumes a filename, it only reads whatever `COMPOSE_FILE` says. This is
documented as the general recovery pattern: if a compose project's deploy
fails this way, check the actual filename in its `PROJECT_DIR` and correct
the metadata, never rename the project's compose file to match an
assumption.

## Recovery patterns (from real exit-path behavior, not just design intent)

| Failure point | What happens | Operator action |
|---|---|---|
| Unknown project / bad metadata | Exit 1, nothing touched | Fix `projects/<name>.conf`, re-run |
| Build or compose step fails | Exit 1, Nginx untouched | Fix the app/build issue, re-run |
| `nginx -t` fails after install | Exit 1, symlink/backup auto-rolled-back | Fix Nginx conf syntax, re-run |
| Reload fails (config was valid) | Exit 1, flagged as unexpected | Check `systemctl status nginx` manually |
| HTTP health check fails | Exit 1, infra already applied | Investigate the app — not a config issue; a bare retry has sometimes resolved a timing race |
| `certbot certonly` fails | Exit 1, HTTP site still serving | Check DNS/rate-limit/webroot, retry `ssl.sh` later |
| `nginx -t` fails after SSL install | Exit 1, SSL symlink rolled back, cert kept | Fix Nginx conf, re-run `ssl.sh` (no reissuance needed) |
| Rollback itself fails | Distinct `ROLLBACK ERROR` reported alongside original | Manual intervention — the one case automation doesn't fully recover |

Routing-level rollback across all cases uses Git, not custom tooling:

```bash
git checkout <prior-commit> -- nginx/sites-available/<project>.conf nginx/sites-available/<project>-ssl.conf
./scripts/deploy.sh <project>
```

Certificates are never affected by a routing rollback — they live outside Git
under `/etc/letsencrypt/`, keyed by domain name.

## Validation commands used during operation

```bash
nginx -t                              # config syntax + semantic check
systemctl status nginx                # is nginx actually running
ls -la /etc/nginx/sites-enabled       # what's actually live
docker compose ps                     # (compose projects) are containers up
certbot certificates                  # cert validity/expiry for all domains
curl -I https://<domain>              # actual external response
journalctl -u nginx -n 50             # recent nginx errors
```
