# Repository Model

## Purpose

Pior Labs uses a polyrepo structure so applications, shared services, packages, and deployment infrastructure can evolve independently while following common platform contracts.

## Naming conventions

| Pattern | Purpose | Examples |
| --- | --- | --- |
| `app-*` | User-facing applications | `app-dashboard`, `app-finance-tracker` |
| `service-*` | Independently deployed shared services | `service-auth` |
| `package-*` | Reusable libraries and packages | `package-design-system` |
| `platform` | Public architecture and standards | This repository |
| `platform-deploy` | Private production deployment implementation | Caddy, routing, shared networks |
| `.github` | Organization profile and shared GitHub configuration | Organization README |

## Ownership boundaries

### Application repositories own

- application source code
- tests, linting, and type checking
- application-specific Docker images
- database schema and migrations
- health endpoints
- development setup
- application release notes

### Service repositories own

- service source code and API contracts
- service-specific data models
- client integration documentation
- service images and migrations
- compatibility and versioning decisions

### Package repositories own

- reusable code with a stable consumer-facing contract
- package builds and publishing
- versioning and changelogs
- usage documentation

### `platform` owns

- public architecture documentation
- repository conventions
- shared platform contracts
- current-versus-target state
- platform roadmap

### `platform-deploy` owns

- production hostname routing
- shared Docker networks
- Caddy image and configuration
- deployment coordination
- production environment handling
- operational validation, cutover, and rollback procedures

## Application contract

Applications joining the platform should:

1. provide repeatable local development instructions
2. build a container image for each deployable component
3. expose a health endpoint where appropriate
4. keep secrets outside the repository
5. use PostgreSQL for relational persistence unless another choice is documented
6. integrate with the shared design system when applicable
7. use central Auth instead of introducing another login system
8. join the shared edge network using unique aliases
9. avoid publishing host ports once edge routing is active
10. document any platform-level dependency

## When to create a repository

A component deserves its own repository when it has an independent lifecycle, deployment boundary, package contract, or clear area of ownership.

Small utilities and tightly coupled modules should remain inside the application that owns them. Repository count should reflect meaningful boundaries rather than every internal package.

## Public and private repositories

Application source, reusable packages, and architecture may be public when they contain no personal data or operational secrets.

Repositories containing production configuration, credentials, private endpoints, or sensitive operational details should remain private.
