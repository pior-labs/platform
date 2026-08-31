# Pior Labs — New Web Application Bootstrap

Use this prompt as reusable context when starting a new web application in the Pior Labs ecosystem.

---

You are helping me create a new web application within **Pior Labs**.

The application should use the established Pior Labs platform as its default foundation rather than introducing application-specific infrastructure without a clear reason.

## Operating principle

Treat the existing Pior Labs platform as the paved road for new applications.

Before designing infrastructure or integration details, inspect the current `platform`, `platform-deploy`, `service-auth`, and `package-design-system` documentation or implementation when available. Those repositories are the source of truth for values and conventions that may change over time.

If an existing convention appears outdated or unsuitable for this application, identify the conflict explicitly before deviating from it.

## Standard application stack

Prefer the established stack unless the product has a concrete requirement that justifies something different:

- TypeScript
- React + Vite frontend
- Hono API
- pnpm workspace / monorepo where appropriate
- PostgreSQL
- Drizzle ORM and migrations
- Docker / Docker Compose
- GitHub Actions CI/CD
- Caddy for edge routing and TLS
- Tailscale for private access
- Cloudflare wildcard DNS for the normal/LAN path
- `@pior-labs/design-system` for shared UI
- `service-auth` for centralized OAuth/OIDC authentication

Do not create a separate authentication system, reverse proxy, database server, deployment pattern, or design system when the shared platform already provides one.

## Repository boundaries

The application repository owns:

- application source code
- app-specific Docker configuration
- app-specific database schema and migrations
- local development configuration
- CI checks
- application-specific deployment workflow
- application documentation

Shared infrastructure does not belong in the application repository.

Use:

- `platform` for public architecture, conventions, and platform documentation
- `platform-deploy` for private production infrastructure, provisioning, routing, secrets integration, and shared deployment concerns
- `service-auth` for authentication behavior and client integration requirements
- `package-design-system` for reusable UI primitives and design conventions

## Authentication

Authentication is provided by the shared Pior Labs `service-auth` service.

Applications should act as OAuth/OIDC clients and should not implement their own username/password authentication system.

For each new application:

- create/register a dedicated OAuth client
- use the current issuer, discovery, callback, and logout conventions documented by `service-auth`
- keep production auth configuration environment-driven
- do not guess or duplicate auth endpoints when the source-of-truth repository can be inspected

Choose a stable application slug and use it consistently for the OAuth client
ID and Better Auth cookie prefix. Never reuse another application's OAuth client
ID, client secret, session secret, database, or cookie prefix.

### Local authentication contract

User-facing applications run one at a time behind the standard Vite origin
`http://localhost:5173`. An application may retain its own API port; Vite
proxies `/api/*` to that API so the browser-facing Better Auth URL and OAuth
callback remain on port `5173`.

Normal application development uses the hosted central SSO service:

```env
WEB_PORT=5173
BETTER_AUTH_URL=http://localhost:5173
BETTER_AUTH_TRUSTED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

CENTRAL_AUTH_ISSUER=https://auth.szarans.ca/api/auth
CENTRAL_AUTH_DISCOVERY_URL=https://auth.szarans.ca/api/auth/.well-known/openid-configuration
CENTRAL_AUTH_CLIENT_ID=<app-client-id>
CENTRAL_AUTH_CLIENT_SECRET=<matching-server-only-secret>
```

The OAuth client secret and application session secret are server-only. Never
place them in `VITE_*` variables or source control.

Each application creates its own local Better Auth session after central SSO.
Namespace all cookies because browser cookies are shared across localhost ports
and persist when switching applications:

```ts
export const auth = betterAuth({
  // app database, OAuth plugin, and model mapping...
  advanced: {
    cookiePrefix: '<app-slug>',
    database: {
      generateId: 'serial',
    },
  },
});
```

Add regression coverage for both `<app-slug>.session_token` and
`<app-slug>.oauth_state`.

Register both exact callbacks on the application's unique OAuth client:

```text
https://<app>.szarans.ca/api/auth/oauth2/callback/auth-pior
http://localhost:5173/api/auth/oauth2/callback/auth-pior
```

After changing a production client registration, merge the `service-auth`
change, run **Bootstrap Auth Production** to reseed clients, then run **Deploy
Auth Production** to restart Auth and reload cached registrations.

## Database

PostgreSQL is provided centrally by the Pior Labs platform.

Each application should normally receive:

- its own database
- its own PostgreSQL role
- its own generated credential
- permissions scoped to that database

Database and role provisioning belong in `platform-deploy`.

Applications should consume platform-managed production database credentials rather than maintaining duplicate database passwords in multiple secret stores whenever the current platform pattern supports it.

Manage schema changes through Drizzle migrations.

## Networking

Production applications run as Docker containers behind the shared Caddy edge.

Use the current Pior Labs networking conventions documented by `platform` and implemented by `platform-deploy`.

Each application uses one canonical hostname:

- `<app>.szarans.ca`

The same hostname is used on the trusted LAN and through Tailscale. Cloudflare and Tailscale wildcard DNS return the appropriate private address for the client's network context, so applications should not create per-app DNS records or depend on separate `.ts.szarans.ca` hostnames.

This single-hostname model should also be reflected in application configuration and OAuth callback registration: production needs one canonical application URL rather than separate LAN and Tailscale variants.

Applications should not independently expose public host ports unless there is a specific operational reason.

Use the shared Docker edge network where required by the current platform architecture.

## Deployment

Applications are deployed with GitHub Actions and self-hosted runners using the established Pior Labs deployment pattern.

A production deployment should generally be repeatable and capable of:

1. obtaining the intended application revision
2. applying the production environment configuration
3. building or updating Docker images
4. running database migrations
5. recreating application containers safely
6. verifying service health

Do not rely on undocumented manual server changes as part of normal deployment.

Never commit production secrets to the repository.

## UI and design system

Use `@pior-labs/design-system` for shared UI components, typography, colors, tokens, and layout conventions where appropriate.

Application-specific UI may extend the design system, but avoid recreating common components that belong in the shared package.

If a reusable component emerges during application development, consider whether it belongs upstream in `package-design-system` rather than permanently inside the app.

## Health and operations

Every long-running application service should expose an appropriate health endpoint and Docker health check where practical.

Design new services so they can integrate cleanly with shared operational capabilities such as:

- monitoring
- backups
- centralized logging
- observability

Do not create bespoke versions of those systems unless the application has an immediate requirement that the platform does not satisfy.

## Starting a new application

When initializing a new Pior Labs web application, work through the following sequence:

1. Define the product requirements and smallest useful first release.
2. Start from the current Pior Labs web application template when available and appropriate.
3. Identify required frontend, API, database, worker, storage, or MCP components.
4. Define the database and PostgreSQL role that need to be provisioned.
5. Define the `service-auth` OAuth client and callback requirements.
6. Choose the canonical application hostname. The platform wildcards cover it automatically; identify only exceptional DNS or Docker-alias requirements.
7. Identify required `platform-deploy` changes.
8. Establish local development configuration.
9. Establish CI checks.
10. Establish repeatable production deployment.
11. Add health checks and operational verification.
12. Document the application architecture and any justified deviations from platform conventions.

## Expected behavior from the agent

Prefer reuse over reinvention.

When proposing a new dependency, service, infrastructure component, or architectural pattern, first determine whether Pior Labs already provides an equivalent capability.

Separate product decisions from platform decisions. Product-specific requirements should stay in the application; reusable infrastructure improvements should be considered for the shared platform.

Do not assume documentation is current when repository state can be inspected. Reconcile documentation with implementation when necessary.

## Current objective

We are starting a new Pior Labs web application.

Application: `<APP_NAME>`

Initial product idea:

`<PRODUCT_DESCRIPTION>`

First help define the application's requirements and architecture. Then determine what can come directly from the existing Pior Labs foundation and what app-specific work or platform provisioning is required.
