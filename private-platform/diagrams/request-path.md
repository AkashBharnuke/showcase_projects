# Request Path — Implemented vs Planned

Layered access chain for a private application, as confirmed in `specs/002`, `specs/003`, and implemented by `scripts/setup/*`. Each layer is independent — none substitutes for another (explicit project rule).

```mermaid
flowchart TD
    A[Internet] --> B[VPS Firewall — UFW]
    B --> C[WireGuard VPN<br/>private subnet]
    C --> D[Private Nginx<br/>nginx-private.service]
    D --> E[Application<br/>localhost]
    E --> F[Application Auth]

    style B fill:#2563eb,color:#fff
    style C fill:#2563eb,color:#fff
    style D fill:#2563eb,color:#fff
    style E fill:#6b7280,color:#fff
    style F fill:#6b7280,color:#fff
```

**Legend:** blue = implemented and scripted (UFW rule, WireGuard interface, private Nginx instance). Gray = owned by each application's own repository, outside this platform's scope — shown for completeness, not claimed as built here.

## Side-by-side with the public plane

The private plane never shares a process, config file, or listener with the existing public plane (`PlatformVPS`, under `/srv/`). This separation is a hard constraint, not an implementation detail.

```mermaid
flowchart LR
    subgraph Public["Public plane — PlatformVPS (separate repo, untouched)"]
        PI[Internet] --> PN[Public Nginx] --> PA[Public apps]
    end

    subgraph Private["Private plane — PrivatePlatform (this repo)"]
        WI[Internet] --> UFW[UFW] --> WG[WireGuard<br/>private subnet] --> PRN[Private Nginx<br/>nginx-private.service] --> PRA[Private apps]
    end

    VPS[One VPS, one filesystem] --- Public
    VPS --- Private
```

One VPS, two independent planes. The private plane's Nginx binds only the WireGuard interface — `scripts/nginx/register-app.sh` hard-rejects any site config that binds `0.0.0.0` or a bare port, to prevent an app leaking onto the public interface by mistake.
