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
| Shared UI | Publish and consume the shared design-system package | Complete |
| Domains | Standardize application hostnames under `szarans.ca` | Complete |
| Split DNS | Use one application hostname across LAN and Tailscale access | Complete |
| Deployment foundation | Establish the private `platform-deploy` repository | Complete |
| Edge | Run containerized Caddy with Cloudflare DNS challenges | Complete |
| Dashboard | Run Dashboard behind the shared edge | Complete |
| Finance deployment | Standardize Finance containers, platform networking, and PostgreSQL | Complete |
| Authentication | Deploy `service-auth` to production | Complete |
| Finance SSO | Replace Finance's local login with central SSO | Complete |
| Application template | Establish a reusable foundation for future web applications | Complete |
| Deployment isolation | Reduce privileged production responsibilities in public application repositories | In progress |
| Documentation | Keep architecture, networking, security, and application docs aligned with production | In progress |
| Cookbook | Build the first new application on the standardized platform from the start | Planned |
| Observability | Standardize health checks, monitoring, and deployment verification | In progress |
| Backups | Document and automate database and platform backup/restore procedures | Planned |
| Assistant | Build an MCP-enabled assistant across platform tools | Later |
| Vulnerability management | Standardize dependency and vulnerability monitoring | Later |

## Completed foundation

The original migration sequence is complete:

1. Application hostnames moved to `szarans.ca`.
2. Containerized Caddy became the shared edge.
3. Applications joined the shared platform networks.
4. Production infrastructure configuration moved into `platform-deploy`.
5. `service-auth` was deployed and validated.
6. Finance was migrated to central SSO and platform-managed PostgreSQL.
7. Split-horizon DNS removed the need for separate Tailscale application hostnames.
8. A reusable web application template and bootstrap prompt were established for future projects.

## Current priority order

1. Finish reconciling public documentation with the operational platform.
2. Use the standardized foundation for the Cookbook rather than introducing a new application-specific pattern.
3. Continue reducing privileged deployment logic in public application repositories.
4. Standardize monitoring and post-deployment verification across services.
5. Establish documented and tested backup/restore procedures.
6. Expand the platform assistant and MCP layer after the application ecosystem is established.

## Out of scope for the current phase

The following technologies may be explored later but are not required for the current home-server platform:

- Kubernetes
- Terraform or Pulumi
- multi-node orchestration
- public multi-tenant hosting

The current priority is to deepen the reliability and repeatability of the existing Docker, Caddy, GitHub Actions, PostgreSQL, Tailscale, and SSO foundation before adding another infrastructure layer.

## Updating this roadmap

A roadmap item should change status only when implementation work has meaningfully begun or the capability is operating in its intended environment. Documentation or scaffolding alone is not sufficient to mark a capability complete.

## Related documentation

- [Architecture](architecture.md)
- [Repository model](repository-model.md)
- [Networking](networking.md)
- [Security](security.md)
