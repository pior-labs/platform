# Security

## Overview

Pior Labs is a private, self-hosted platform for personal and household applications. Its security model prioritizes reducing public exposure, separating application and infrastructure responsibilities, limiting credentials, and keeping production operations out of public repositories.

This document describes the platform security posture. It distinguishes between controls that are already implemented, controls being standardized, and future hardening work.

## Security goals

- Keep applications and personal data inaccessible from the public internet by default.
- Minimize the number of services exposed on the host.
- Encrypt browser and API traffic with publicly trusted HTTPS certificates.
- Limit the impact of a compromised application, credential, or automation workflow.
- Keep production secrets and operational details out of public repositories.
- Preserve a clear boundary between public CI and private production deployment.

## Control status

| Area | Status |
| --- | --- |
| Private LAN and Tailscale access | Implemented |
| HTTPS through Caddy | Implemented |
| Scoped Cloudflare DNS credential | Implemented |
| SSH key-only server access | Implemented |
| Public/private repository separation | Implemented |
| Shared Docker edge network | In progress |
| Containerized Caddy edge | In progress |
| Centralized private deployment runner | Planned |
| Central authentication and SSO | Planned |
| Standardized backup and restore testing | Planned |
| Standardized vulnerability and dependency monitoring | Planned |

## Access model

Applications are intended to be accessed only from:

- the trusted local network
- authenticated devices connected through Tailscale

DNS records may be publicly resolvable, but they point to private LAN or Tailscale addresses. Cloudflare proxying is disabled for these records because the services are not intended to be publicly reachable.

Administrative tools should remain restricted to the local network or Tailscale and should not be exposed through public port forwarding.

## Host access

Administrative server access uses SSH keys rather than passwords.

The host SSH configuration disables password and keyboard-interactive authentication. Firewall rules restrict inbound access to the interfaces and services required by the platform.

Direct host access is reserved for administration and recovery. Routine application deployment is being moved toward controlled automation so fewer changes require interactive server access.

## TLS and edge security

Caddy provides the HTTPS boundary for platform applications.

Certificates are issued through ACME using the DNS-01 challenge. This allows private services to receive publicly trusted certificates without exposing an HTTP validation endpoint to the internet.

The Cloudflare API token used by Caddy is:

- scoped to the `szarans.ca` zone
- limited to the DNS permissions required for certificate validation
- stored in protected deployment configuration
- excluded from source control

The target architecture uses one containerized Caddy edge service as the only platform component publishing host ports `80` and `443`. Application containers will be reachable through private Docker networking rather than direct host port exposure.

## Secrets and credentials

Secrets are kept outside public source code and documentation.

The platform uses the following practices:

- production configuration is maintained in the private `platform-deploy` repository
- credentials are stored in GitHub Secrets or server-side environment files
- real `.env` files are excluded from Git
- API tokens are scoped to the smallest practical resource and permission set
- public documentation excludes credentials, private IP addresses, database connection strings, and recovery secrets
- validation workflows use non-production placeholder values where a real credential is unnecessary

Secrets should not be passed through command output, committed example files, container images, or pull-request logs.

## Repository and CI/CD isolation

Public application repositories should be able to test and build code without receiving production credentials.

The target workflow separates responsibilities:

1. application repositories run tests and build application images
2. images are published to a container registry
3. the private `platform-deploy` repository selects and deploys approved images
4. only the private deployment workflow can reach production Docker and environment configuration

A repository-scoped self-hosted runner is planned for `platform-deploy`. Public repositories should use GitHub-hosted runners and should not have unrestricted access to the home server, Docker socket, or private network.

Existing application-specific deployment workflows will be retired as applications migrate to the centralized model.

## Container and network isolation

Caddy and application-facing containers communicate through the shared `pior_edge` Docker network.

Only containers that must receive traffic from Caddy should join this network. Databases and internal-only services should remain on application-specific private networks.

Additional platform conventions include:

- unique service aliases across the edge network
- no direct host port publishing after edge migration
- separate application and database credentials
- minimal cross-application network access
- explicit health checks before deployment cutover

Docker access is treated as privileged because control of the Docker daemon can provide effective root access to the host.

## Database security

PostgreSQL is shared infrastructure, but application access should remain isolated.

Each application should use its own database or schema and a dedicated database role with only the permissions it requires. Administrative database credentials should not be used by application containers.

Database ports should remain private and should not be exposed to the public internet. Administrative access should occur only over trusted LAN or Tailscale paths.

Database-role isolation and migration ownership are being standardized as existing applications move into the platform deployment model.

## Authentication

Finance currently uses application-specific authentication. The target architecture replaces separate login implementations with the private `service-auth` identity provider.

Central authentication is intended to provide:

- one identity across platform applications
- consistent session and login behavior
- fewer independent password-handling implementations
- application-specific authorization after identity verification

Applications will continue to own their domain-specific users, permissions, and data. Central SSO does not replace application authorization.

The Auth service must be deployed and validated before existing application authentication is removed.


## Current limitations

Pior Labs is a personal platform and does not currently have the controls or redundancy of a managed production environment.

Known limitations include:

- a single home server remains a major availability boundary
- Docker access is highly privileged
- central SSO is not yet deployed
- deployment runner isolation is not yet fully standardized
- backup and restore verification is not yet standardized
- vulnerability monitoring is not yet consistent across all repositories
- security controls have not undergone an independent audit or penetration test
- public DNS records can reveal the existence and naming of private services

These limitations are accepted for the current scope and should be reviewed as the platform grows or stores more sensitive information.

## Related documentation

- [Architecture](architecture.md)
- [Networking](networking.md)
- [Repository model](repository-model.md)
- [Roadmap](roadmap.md)
