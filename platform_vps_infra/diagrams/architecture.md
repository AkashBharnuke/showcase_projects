# Architecture Diagrams

Mermaid source, verified against the live repo's scripts and Nginx configs (see
`../docs/ARCHITECTURE.md` for the full narrative). No renderer was available in
this environment — these are source diagrams; paste into any Mermaid-compatible
viewer (GitHub renders `.md` Mermaid blocks natively) or run `mmdc` to export
PNG/SVG.

## System ownership boundary

```mermaid
flowchart TB
    subgraph pub[platform-vps-infra - this repo]
        pn[Public Nginx config]
        ps[Public SSL and Certbot triggers]
        pr[Public domain routing]
        ph[Platform homepage]
        pm[Project metadata]
    end
    subgraph app[Application repos - own checkouts]
        ac[Own code and Dockerfile]
        adc[Compose file]
        ae[Env files]
        abl[Business logic]
    end
    subgraph priv[Private plane - never touched by this repo]
        popt[opt directory]
        pp[PrivatePlatform]
        wg[WireGuard]
        pnx[Private Nginx]
        pap[Private applications]
    end

    pub -. builds, routes to, issues certs for .-> app
```

## Request flow

```mermaid
flowchart LR
    internet([Internet request])
    nginx{{Public Nginx}}
    default[default.conf - return 444]
    static[Static project - homepage, dockyard]
    compose[Compose project - wishmesurprise, knowwhere, retrogamevault]

    internet --> nginx
    nginx -- domain matches compose project --> compose
    nginx -- domain matches static project --> static
    nginx -- no server_name matches --> default
```

## Deployment flow (`deploy.sh`)

```mermaid
flowchart TD
    op([Operator runs deploy.sh project])
    load[load_project validates metadata]
    fail1[Exit 1 - nothing touched]
    static{TYPE}
    buildstatic[Build if needed, copy to OUTPUT_DIR]
    buildcompose[docker compose up -d --build, then ps check]
    install[install_nginx_site - copy and symlink]
    test{nginx -t}
    rollback[rollback_nginx_site]
    reload[reload nginx]
    health{curl health check}

    op --> load
    load -- invalid --> fail1
    load -- valid --> static
    static -- static --> buildstatic
    static -- compose --> buildcompose
    buildstatic --> install
    buildcompose --> install
    install --> test
    test -- fail --> rollback
    test -- pass --> reload
    reload --> health
```

## SSL issuance flow (`ssl.sh`)

```mermaid
flowchart LR
    http[HTTP site already enabled by deploy.sh]
    check{HTTP site enabled}
    abort1[Exit 1 - run deploy.sh first]
    certbot[certbot certonly - webroot mode]
    abort2[Exit 1 - nginx untouched]
    certfiles[(cert files written under letsencrypt)]
    installssl[install SSL conf]
    test2{nginx -t}
    rollback2[rollback symlink, cert is kept]
    reload2[reload nginx]
    verify[curl https verify]

    http --> check
    check -- no --> abort1
    check -- yes --> certbot
    certbot -- fail --> abort2
    certbot -- success --> certfiles
    certfiles --> installssl
    installssl --> test2
    test2 -- fail --> rollback2
    test2 -- pass --> reload2
    reload2 --> verify
```
