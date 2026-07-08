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

## Public and private repositories

Application source, reusable packages, and architecture may be public when they contain no personal data or operational secrets.

Repositories containing production configuration, credentials, private endpoints, or sensitive operational details remain private.
