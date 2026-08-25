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

The trigger policy is repository-specific. High-impact infrastructure, bootstrap, and newly introduced application deployments remain manual through `workflow_dispatch`. A proven application workflow may deploy after successful CI on a push to `main`, as Finance currently does.

This separation keeps routine validation isolated from the home server while making production access controlled and auditable.

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

Deployment workflows render or mount the required runtime configuration without printing secret values.

## Promoting a new application to production

1. Define the hostname, edge-network aliases, database role, and persistent-storage requirements in `platform-deploy`.
2. Add an application-owned production Compose override, deployment workflow, readiness checks, and operations guide.
3. Configure the repository's protected production environment, variables, and secrets.
4. Provision and register the repository-scoped self-hosted runner.
5. Deploy `platform-deploy` first when shared infrastructure changed.
6. Deploy the application from its production branch.
7. Verify readiness, authentication, routing, persistence, and backup boundaries.

## Documentation contract

The exact runner username, labels, server paths, variables, secrets, deployment procedure, and recovery commands belong in the deployable repository's operations guide. Its README should provide a short deployment summary and link to that guide.

This repository documents the reusable pattern. It intentionally does not duplicate private production values or the full operational procedure for every application.
