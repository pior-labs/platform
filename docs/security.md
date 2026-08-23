# Security

## Overview

Pior Labs is a private, self-hosted platform for personal and household applications. Its security model prioritizes reducing public exposure, separating application and infrastructure responsibilities, limiting credentials, and keeping production operations out of public repositories.

This document describes the platform security posture. It distinguishes between controls that are implemented, controls still being standardized, and future hardening work.

## Security goals

- Keep applications and personal data inaccessible from the public internet by default.
- Minimize the number of services exposed on the host.
- Encrypt browser and API traffic with publicly trusted HTTPS certificates.
- Limit the impact of a compromised application, credential, or automation workflow.
- Keep production secrets and operational details out of public repositories.
- Preserve a clear boundary between public source code and privileged production operations.

## Control status

| Area | Status |
| --- | --- |
| Private LAN and Tailscale access | Implemented |
| Split-horizon private DNS | Implemented |
| HTTPS through containerized Caddy | Implemented |
| Scoped Cloudflare DNS credential | Implemented |
| SSH key-only server access | Implemented |
| Public/private repository separation | Implemented |
| Shared Docker edge network | Implemented |
| Central authentication and SSO | Implemented |
| Per-application PostgreSQL databases and roles | Implemented |
| Production deployment isolation | In progress |
| Standardized backup and restore testing | Planned |
| Standardized vulnerability and dependency monitoring | Planned |

## Access model

Applications are intended to be accessed only from:

- the trusted local network
- authenticated devices connected through Tailscale

The same `szarans.ca` application hostname can be used on both access paths because split-horizon DNS resolves it to the appropriate private address.

Public DNS records may reveal service names, but they do not by themselves make the applications publicly reachable. Administrative tools should remain restricted to trusted LAN or Tailscale access and should not be exposed through public port forwarding.

## Host access

Administrative server access uses SSH keys rather than passwords.

The host SSH configuration disables password and keyboard-interactive authentication. Firewall rules restrict inbound access to the interfaces and services required by the platform.

Direct host access is reserved for administration and recovery. Routine application deployment is automated where practical so fewer changes require interactive server access.

## TLS and edge security

Caddy provides the HTTPS boundary for platform applications.

Certificates are issued through ACME using the DNS-01 challenge. This allows private services to receive publicly trusted certificates without exposing an HTTP validation endpoint to the internet.

The Cloudflare API token used by Caddy is:

- scoped to the `szarans.ca` zone
- limited to the DNS permissions required for certificate validation
- stored in protected deployment configuration
- excluded from source control

The shared containerized Caddy edge receives application traffic and forwards it to application-facing containers over the `pior_edge` Docker network. Application services are not intended to be directly reachable from untrusted networks.

## Secrets and credentials

Secrets are kept outside public source code and documentation.

The platform uses the following practices:

- production infrastructure configuration is maintained in the private `platform-deploy` repository
- credentials are stored in protected GitHub Secrets or server-side secret/environment files as appropriate
- real `.env` files are excluded from Git
- API tokens are scoped to the smallest practical resource and permission set
- application databases use dedicated roles rather than PostgreSQL administrative credentials
- public documentation excludes credentials, private IP addresses, database connection strings, and recovery secrets
- validation workflows use non-production placeholder values where a real credential is unnecessary

Secrets should not be passed through command output, committed example files, container images, or pull-request logs.

## Repository and CI/CD isolation

Public application repositories should be able to test and build code without unnecessarily receiving broad production credentials.

Production routing, database provisioning, and infrastructure configuration are separated into the private `platform-deploy` repository. Some application-specific deployment responsibilities still remain while the deployment model is being standardized further.

The longer-term direction is to keep privileged production coordination narrowly scoped to dedicated deployment automation while public repositories focus on source, validation, builds, migrations, and release artifacts.

Self-hosted runners and Docker access are treated as privileged because they can provide effective control over the production host.

## Container and network isolation

Caddy and application-facing containers communicate through the shared `pior_edge` Docker network.

Only containers that must receive traffic from Caddy should join this network. Databases and internal-only services remain on private data or application-specific networks unless another trusted platform component requires access.

Platform conventions include:

- unique service aliases across the edge network
- separate application and database credentials
- minimal cross-application network access
- explicit health checks around deployments
- no reliance on publicly exposed database ports

## Database security

PostgreSQL is shared infrastructure, but application access is isolated.

Stateful applications use their own logical databases and dedicated database roles. Production database provisioning is coordinated by the private platform deployment layer, and application containers do not use the PostgreSQL administrator account.

Database ports remain private. Administrative database access should occur only over trusted LAN or Tailscale paths.

## Authentication

`service-auth` is deployed as the platform identity provider and provides centralized OAuth 2.1 / OpenID Connect authentication.

Finance has been migrated to this shared SSO model. Central authentication provides:

- one identity across participating platform applications
- consistent sign-in and session behavior
- fewer independent password-handling implementations
- a reusable authentication contract for future applications

Applications continue to own their domain-specific authorization, users, permissions, and data. Central SSO establishes identity; it does not replace application authorization.

## Current limitations

Pior Labs is a personal platform and does not have the controls or redundancy of a managed production environment.

Known limitations include:

- a single home server remains a major availability boundary
- Docker and self-hosted runner access are highly privileged
- deployment isolation is not yet fully standardized across all repositories
- backup and restore verification is not yet standardized
- vulnerability and dependency monitoring is not yet consistent across all repositories
- security controls have not undergone an independent audit or penetration test
- public DNS can reveal the existence and naming of private services

These limitations are accepted for the current scope and should be reviewed as the platform grows or stores more sensitive information.

## Related documentation

- [Architecture](architecture.md)
- [Networking](networking.md)
- [Repository model](repository-model.md)
- [Roadmap](roadmap.md)
