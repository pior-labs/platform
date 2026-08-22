# Pior Labs Platform

Public architecture and development conventions for Pior Labs self-hosted
applications.

This repository defines the paved road shared by application repositories.
Runtime infrastructure belongs to `pior-labs/platform-deploy`, central identity
and trusted OAuth client registration belong to `pior-labs/service-auth`, and
each application owns its product code, local session, database schema,
containers, and deployment workflow.

## New applications

Start with `pior-labs/template-webapp` and use
[`prompts/new-webapp-bootstrap.md`](./prompts/new-webapp-bootstrap.md) as the
bootstrap contract. Current platform and service documentation takes precedence
over copied template text when conventions evolve.

## Local application development

Pior Labs user-facing applications follow one shared browser-origin convention:

- run one application at a time on `http://localhost:5173`;
- proxy `/api/*` from Vite to that application's local API port;
- authenticate against the hosted canonical issuer at
  `https://auth.szarans.ca/api/auth`;
- register `http://localhost:5173/api/auth/oauth2/callback/auth-pior` on every
  development-enabled OAuth client; and
- configure a distinct Better Auth `cookiePrefix` for every application.

Cookies are scoped by hostname rather than port and survive after a development
server stops. A unique prefix such as `finlens` or `cookbook` prevents session
and OAuth-state cookies from one local application being interpreted by
another.

Running `service-auth` locally remains useful when developing the identity
provider itself. Normal application development uses the hosted service and
does not require the local Auth web/API processes.

## Production identity

The canonical Auth discovery document is:

```text
https://auth.szarans.ca/api/auth/.well-known/openid-configuration
```

The issuer is stable across public, LAN, Tailscale, container, and local
application-development paths. Routing differences must not create alternate
issuer identities.
