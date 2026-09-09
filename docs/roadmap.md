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
| Cookbook foundation | Build and deploy the first new application on the standardized platform from the start | Complete |
| Application MCP | Prove application-owned MCP interfaces against real application services | Complete |
| Cookbook expansion | Extend Cookbook into meal planning and grocery-list workflows | In progress |
| Deployment isolation | Reduce privileged production responsibilities in public application repositories | In progress |
| Documentation | Keep architecture, networking, security, and application docs aligned with production | In progress |
| Observability | Standardize health checks, monitoring, and deployment verification | In progress |
| Backups | Document and automate database, application-data, and platform backup/restore procedures | Planned |
| Cross-application assistant | Build a broader assistant on top of application-owned MCP capabilities | Planned |
| Vulnerability management | Standardize dependency and vulnerability monitoring | Later |

## Completed foundation

The original migration and standardization sequence is complete:

1. Application hostnames moved to `szarans.ca`.
2. Containerized Caddy became the shared edge.
3. Applications joined the shared platform networks.
4. Production infrastructure configuration moved into `platform-deploy`.
5. `service-auth` was deployed and validated.
6. Finance was migrated to central SSO and platform-managed PostgreSQL.
7. Split-horizon DNS removed the need for separate Tailscale application hostnames.
8. A reusable web application template and bootstrap prompt were established for future projects.
9. Cookbook was built from that standardized application pattern through a complete Phase 1 deployment.
10. Cookbook MCP v1 demonstrated that trusted assistants can consume application-owned capabilities without bypassing application business logic.

## Current priority order

1. Extend Cookbook into ad-hoc meal planning and grocery-list generation while keeping the completed recipe experience stable.
2. Evolve application-owned MCP interfaces alongside useful application capabilities so future conversational workflows can reuse the same domain services as the web UI.
3. Continue reducing privileged deployment logic in public application repositories.
4. Standardize monitoring and post-deployment verification across services.
5. Establish documented and tested backup/restore procedures for both PostgreSQL and application-owned file data.
6. Keep the template, bootstrap guidance, and public documentation aligned with patterns proven by real applications.
7. Build the broader Pior Labs assistant once enough application MCP capabilities exist to make cross-application orchestration useful.

## Direction

The platform has moved beyond proving that multiple applications can share authentication, networking, database infrastructure, design conventions, and deployment patterns.

The next stage is application capability and integration:

- deepen the usefulness of the household applications;
- expose narrow application-owned MCP interfaces where conversational access is valuable;
- let those MCP interfaces call the same services used by the web applications rather than creating a second data or business-logic layer; and
- preserve independent application ownership and deployment as cross-application workflows emerge.

## Out of scope for the current phase

The following technologies may be explored later but are not required for the current home-server platform:

- Kubernetes
- Terraform or Pulumi
- multi-node orchestration
- public multi-tenant hosting

The current priority is to deepen the reliability and repeatability of the existing Docker, Caddy, GitHub Actions, PostgreSQL, Tailscale, SSO, and application-MCP foundation before adding another infrastructure layer.

## Updating this roadmap

A roadmap item should change status only when implementation work has meaningfully begun or the capability is operating in its intended environment. Documentation or scaffolding alone is not sufficient to mark a capability complete.

## Related documentation

- [Architecture](architecture.md)
- [Repository model](repository-model.md)
- [Networking](networking.md)
- [Security](security.md)
