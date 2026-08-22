# New Pior Labs Web Application Bootstrap

Use this document when creating an application from
`pior-labs/template-webapp`. Replace every placeholder with an application
identity before implementing product features.

## Repository identity

Choose and use consistently:

- repository name: `app-<name>`;
- application slug: `<name>`;
- production hostname: `https://<name>.szarans.ca`;
- OAuth client ID: a unique stable ID, normally `<name>`;
- Better Auth cookie prefix: a unique stable prefix, normally `<name>`; and
- API/container aliases owned by the application deployment contract.

Never reuse another application's OAuth client ID, client secret, session
secret, database, or cookie prefix.

## Local development contract

User-facing applications run one at a time behind the standard Vite origin:

```text
http://localhost:5173
```

The application may retain its own API port. Vite proxies `/api/*` to that API,
so Better Auth's browser-facing base URL and callback remain on port `5173`.

Use the hosted central SSO service:

```env
WEB_PORT=5173
BETTER_AUTH_URL=http://localhost:5173
BETTER_AUTH_TRUSTED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

CENTRAL_AUTH_ISSUER=https://auth.szarans.ca/api/auth
CENTRAL_AUTH_DISCOVERY_URL=https://auth.szarans.ca/api/auth/.well-known/openid-configuration
CENTRAL_AUTH_CLIENT_ID=<app-client-id>
CENTRAL_AUTH_CLIENT_SECRET=<matching-server-only-secret>
```

The client secret and application session secret are server-only and must never
use `VITE_*` names or enter source control.

## Better Auth client configuration

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

Add regression coverage that checks both generated names:

```text
<app-slug>.session_token
<app-slug>.oauth_state
```

## Central OAuth client registration

Add a unique client in `pior-labs/service-auth` with:

- the canonical production application URI;
- `https://<name>.szarans.ca/api/auth/oauth2/callback/auth-pior`;
- `http://localhost:5173/api/auth/oauth2/callback/auth-pior`;
- scopes `openid profile email offline_access`;
- authorization code and refresh token grants;
- PKCE; and
- the matching raw client secret in protected environments.

After changing a production client registration:

1. merge the `service-auth` change;
2. run **Bootstrap Auth Production** to reseed OAuth clients; and
3. run **Deploy Auth Production** to restart Auth and reload cached clients.

## Platform integration

Before enabling production deployment:

1. provision the database, role, and server-managed connection file in
   `platform-deploy`;
2. add canonical Caddy routing and split DNS;
3. configure application and Auth environment secrets;
4. run application migrations;
5. verify login, callback, session restoration, protected API access, and
   logout; and
6. document any intentional deviation from this paved road in the application
   repository.
