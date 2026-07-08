# Roadmap

This roadmap tracks platform-level work rather than application feature development. It intentionally uses broad phases instead of fixed delivery dates.

## Status definitions

| Status | Meaning |
| --- | --- |
| Complete | Implemented and adopted |
| In progress | Active work is underway |
| Planned | Accepted direction, not yet started |
| Later | Useful but not required for the current foundation |

## Platform roadmap

| Phase | Work | Status |
| --- | --- | --- |
| Organization | Establish the Pior Labs organization and repository model | Complete |
| Shared UI | Publish and consume the shared design-system package | In progress |
| Domains | Migrate application hostnames to `szarans.ca` | In progress |
| Deployment foundation | Establish the private `platform-deploy` repository | In progress |
| Edge | Build and validate containerized Caddy with Cloudflare DNS challenges | In progress |
| Dashboard | Migrate Dashboard to the shared edge network | Planned |
| Finance deployment | Standardize Finance containers and platform networking | Planned |
| Edge cutover | Replace host-installed Caddy with the containerized edge | Planned |
| Authentication | Deploy `service-auth` to production | Planned |
| Finance SSO | Replace Finance's local login with central SSO | Planned |
| Deployment automation | Coordinate production deployments through a dedicated private runner | Planned |
| Documentation | Add authentication and deployment documentation as contracts stabilize | Planned |
| Cookbook | Build the first application using the standardized platform from the start | Later |
| Assistant | Build an MCP-enabled assistant across platform tools | Later |
| Observability | Standardize health checks, monitoring, and deployment verification | Later |
| Backups | Document and automate database and platform backup procedures | Later |

## Current priority order

1. Complete the new DNS and hostname foundation.
2. Validate containerized Caddy without disrupting the existing host service.
3. Move Dashboard behind the shared edge.
4. Move Finance behind the shared edge without changing authentication.
5. Cut over production traffic to containerized Caddy.
6. Deploy and validate the Auth service.
7. Migrate Finance to SSO.
8. Use the completed pattern for new applications.

## Out of scope for the current phase

The following technologies may be explored later but are not required for the current home-server platform:

- Kubernetes
- Terraform (or Pulumi)
- multi-node orchestration
- public multi-tenant hosting

The current priority is to make the Docker, Caddy, GitHub Actions, PostgreSQL, and Tailscale foundation consistent and reliable before adding another infrastructure layer.

## Updating this roadmap

A roadmap item should change status only when implementation work has meaningfully begun or the capability is operating in its intended environment. Planned architecture should not be marked complete based only on documentation or scaffolding.
