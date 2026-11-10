# Deployment Model

Pior Labs keeps applications independently deployable while sharing a common production foundation. GitHub Actions is the control plane: hosted runners validate changes, and repository-scoped self-hosted runners perform production work on the OptiPlex server.

## Ownership boundaries

| Repository | Deployment responsibility |
| --- | --- |
| `platform` | Documents the public deployment pattern and repository boundaries |
| `platform-deploy` | Provisions shared routing, networks, databases, DNS, and persistent-data contracts |
| `app-*` and `service-*` | Build images, apply schema migrations, render application runtime configuration, and deploy their own services |
| `.github` | Provides organization-level profile and shared GitHub configuration |

A shared infrastructure change is deployed from `platform-deploy`. An application-only change is deployed from that application's repository.

## Validation and production deployment

Pull requests and normal CI should use GitHub-hosted runners whenever they do not need access to private production resources. Production jobs target a self-hosted runner on the server and run only from the repository's production branch.

The standard release path is:

1. PRs validate on hosted runners, including application-owned tests and image
   builds. They never use production runners or publish deployable releases.
2. Successful pushes to main publish images to GHCR and save an attempt-specific
   release manifest with source SHA and immutable image digests. Static applications
   publish an archive with a verified checksum instead.
3. Manual deployment selects the full numeric successful main CI run ID and an
   optional exact attempt. A hosted resolver verifies workflow identity, success,
   main ancestry and release contents before production access.
4. The repository runner pulls the selected artifacts and matching source
   configuration, applies app-owned migrations, waits for container health and
   checks the application's canonical ingress. It never rebuilds application images
   or fetches a moving branch as the release source.
5. Current, previous and failed attempted releases are recorded privately on the
   host. Application rollback redeploys an eligible earlier run/attempt using current
   secrets. Migrations and persistent data are not automatically reversed.

Application deployment is manual by default. Dashboard retains automatic deployment
only after successful main CI and skips superseded builds; it also supports manual
release selection. Auth bootstrap remains a separate explicitly confirmed operation
sharing the same concurrency group as Auth deploy. Platform deployment remains
manual and retains its provisioning and external database-image controls. The shared
Caddy image is built in CI; infrastructure rollback is not database or DNS recovery.
Package publication follows its own versioned registry release workflow. Health's
learning project has hosted Go CI without a production release scaffold yet.

Every production workflow for one repository shares a concurrency group with
`cancel-in-progress: false`. This prevents a new request from cancelling in-flight
migrations. GitHub may replace a pending request; it does not provide a FIFO queue.
Readiness probes must reflect each application's actual dependencies rather than
assuming a container-status listing proves successful deployment.

Cookbook is the first proven end-to-end implementation. Other repositories adopt
this contract through their own reviewed PRs, package permissions, runner setup and
first deployment. Documentation of the pattern does not establish that every
consumer has deployed it. The owning operations guide records activation and
recovery details, including first-migration limits and artifact retention.

## Self-hosted runner convention

Every repository that deploys to production receives its own repository-scoped runner rather than sharing one organization-wide deployment runner.

The server convention is:

- one dedicated Linux account per repository;
- one runner installation under `/opt/actions-runner/<repository>`;
- one stable deployment checkout or runtime directory under `/opt`;
- workflow labels that match the repository's production workflow;
- Docker access only when that deployment requires it; and
- a systemd service installed through the GitHub runner's `svc.sh` helper.

Repository scoping is the primary isolation boundary. The dedicated Linux account and directory ownership reduce accidental cross-application access on the host.

Runner registration tokens are short-lived setup values. They are used only while running `config.sh` and are not stored as repository secrets or production configuration.

## Configuration and secrets

Non-sensitive deployment configuration belongs in GitHub Actions variables, normally within a protected `production` environment. Sensitive application values belong in Actions secrets or in platform-managed server-side secret files, according to the repository contract.

Do not store production values in:

- committed `.env` files;
- runner registration commands;
- runner service configuration; or
- public documentation.

Deployment workflows render or mount the required runtime configuration without
printing secret values. Applications use a separate `COMPOSE_PROJECT_NAME` Actions
secret matching the existing production project identity while monolithic runtime
secrets are split incrementally. Existing application-specific secret names remain
valid. Resolved host Compose files contain secrets and must stay private. Naming an
Actions environment alone does not configure required reviewers or branch rules.

## Promoting a new application to production

1. Choose the canonical `<app>.szarans.ca` hostname. The platform DNS wildcards cover it automatically; add only its Caddy route and any required edge-network aliases in `platform-deploy`.
2. Add an application-owned production Compose override, deployment workflow, readiness checks, and operations guide.
3. Configure the repository's protected production environment, variables, and secrets.
4. Provision and register the repository-scoped self-hosted runner.
5. Deploy `platform-deploy` first when shared infrastructure changed.
6. Wait for a successful main CI release and deploy its exact run ID/attempt.
7. Verify wildcard DNS resolution, readiness, authentication, routing, persistence, and backup boundaries.

## Documentation contract

The exact runner username, labels, server paths, variables, secrets, deployment procedure, and recovery commands belong in the deployable repository's operations guide. Its README should provide a short deployment summary and link to that guide.

This repository documents the reusable pattern. It intentionally does not duplicate private production values or the full operational procedure for every application.
