# Application Onboarding Workflow

How a new private application joins the platform, per the convention confirmed across `specs/001` and `specs/006` and implemented by `scripts/nginx/register-app.sh`. Demonstrated once in practice (WatchTower, plus Grafana as a sub-service registered under WatchTower's project).

```mermaid
flowchart TD
    A[App repository exists independently<br/>e.g. WatchTower] --> B["Clone to VPS:<br/>/opt/PROJECTS/&lt;project_name&gt;/"]
    B --> C[App's own docker-compose.yml<br/>runs from its own directory]
    C --> D["Author Nginx site config:<br/>nginx/sites-available/&lt;project_name&gt;.conf<br/>(must bind the WireGuard interface address)"]
    D --> E["Run scripts/nginx/register-app.sh &lt;project_name&gt;"]
    E --> F{Validates:<br/>project dir exists,<br/>site config exists,<br/>listen directive is private-only}
    F -- fail --> G[Abort — nothing linked]
    F -- pass --> H[Symlink into<br/>nginx/sites-enabled/]
    H --> I[nginx -t validates full config]
    I -- fail --> J[Roll back symlink]
    I -- pass --> K[Reload nginx-private.service only]
    K --> L[App reachable via<br/>WireGuard → private Nginx]
```

**Status:** this exact flow has been run for two real site registrations (`watchtower.conf`, `grafana.conf`) — the script and its validation guard are implemented and exercised, not just designed. What remains unverified in this session: there is no automated test suite confirming the rollback path (`F -- fail`, `I -- fail`) actually triggers correctly; that logic is present in the script but not independently proven here.
