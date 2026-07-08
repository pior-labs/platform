# Architecture

## Purpose

Pior Labs provides a shared foundation for independently developed self-hosted applications. It centralizes capabilities that should not be rebuilt by every project: routing, TLS, authentication, deployment conventions, shared packages, and database infrastructure.

## Scope

The platform runs on a home server and is intended primarily for personal and household use. Applications are available over the local network and Tailscale rather than being operated as public SaaS products.

## Current architecture

The system is functional but still contains application-specific deployment patterns.

- Dashboard and Finance are deployed independently.
- A host-installed Caddy service handles routing and HTTPS.
- Finance manages its own authentication.
- PostgreSQL is available as shared database infrastructure.
- GitHub Actions deploy applications through self-hosted runners.
- Tailscale provides private remote access.

## Target architecture

```mermaid
flowchart TB
    LAN[LAN clients] --> DNS[Cloudflare DNS]
    TS[Tailscale clients] --> DNS
    DNS --> EDGE[Containerized Caddy edge]

    EDGE --> DASH[Dashboard]
    EDGE --> FINWEB[Finance web]
    EDGE --> FINAPI[Finance API]
    EDGE --> AUTH[Authentication service]
    EDGE --> FUTURE[Future applications]

    FINWEB --> FINAPI
    FINAPI --> AUTH
    FUTURE --> AUTH

    FINAPI --> DB[(PostgreSQL)]
    AUTH --> DB
    FUTURE --> DB
```

## Component responsibilities

### Caddy edge

The shared Caddy service owns hostname routing, HTTPS certificate management, Cloudflare DNS challenges, and forwarding traffic to application containers over a private Docker network.

Once migration is complete, application containers should not publish their own web ports to the host. Caddy should be the only platform service publishing ports `80` and `443`.

### Applications

Each application repository owns its source code, tests, container images, database migrations, health endpoints, and release process. Applications expose only the services required by Caddy or other trusted platform components.

### Authentication service

`service-auth` is the central identity provider. It provides one sign-in experience while allowing each application to retain its own domain-specific users, data, and authorization rules.

Finance currently uses local authentication and will be migrated after Auth is deployed and stable.

### PostgreSQL

PostgreSQL is the standard relational database for stateful applications. Applications should use separate databases or roles so access remains isolated even when infrastructure is shared.

### Shared packages

Reusable code belongs in `package-*` repositories when it has a clear cross-application contract. The design system is the first shared package and provides common components, tokens, and visual conventions.

### GitHub Actions

Application repositories validate, build, and publish their own artifacts. Private deployment automation coordinates production changes so production credentials are not exposed to ordinary public-repository CI jobs.

## Trust boundaries

1. DNS may be public, while application access remains limited to LAN and Tailscale clients.
2. Only required application-facing containers join the shared edge network.
3. Each service receives only the database access it requires.
4. Architecture is public; operational configuration and secrets remain private.
5. Build workflows and production deployment use separate credentials and runners.

## Migration approach

1. Establish new DNS names without removing the old ones.
2. Run containerized Caddy on non-conflicting test ports.
3. Migrate Dashboard as the lowest-risk application.
4. Migrate Finance without changing its authentication model.
5. Cut over ports `80` and `443`.
6. Deploy Auth.
7. Migrate Finance to SSO.
8. Retire obsolete deployment paths and hostnames.

## Related documentation

- [Repository model](repository-model.md)
- [Networking](networking.md)
- [Security](security.md)
- [Roadmap](roadmap.md)
