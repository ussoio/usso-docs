# Tenant & integration resources

Tenant endpoints provision organisations, rotate signing keys, and expose
multi-tenant configuration. Integration endpoints implement OAuth 2.0 and OIDC
flows so tenants can act as their own identity providers.

## Tenants

### Responsibilities

* Create and manage tenant metadata, domain routing rules, and signing key
  material.【F:app/apps/tenant/routes.py†L1-L59】
* Associate tenants with the authenticated creator and enforce authorization via
  the tenant base router.【F:app/utils/usso_routes.py†L1-L24】【F:app/apps/tenant/routes.py†L1-L33】
* Provide update hooks so operators can rotate secrets, toggle activation flags,
  and adjust configuration payloads.【F:app/apps/tenant/routes.py†L55-L59】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /tenants` | Provisions a tenant, seeds signing keys, and links the creator account.【F:app/apps/tenant/routes.py†L33-L53】 |
| `GET /tenants` | Lists tenants available to the caller (typically super administrators).【F:app/apps/tenant/routes.py†L1-L33】 |
| `GET /tenants/{uid}` | Returns tenant metadata, including domains and configuration payloads.【F:app/apps/tenant/routes.py†L1-L33】 |
| `PATCH /tenants/{uid}` | Updates metadata, domains, and activation flags; rotates secrets when provided.【F:app/apps/tenant/routes.py†L55-L59】 |
| `DELETE /tenants/{uid}` | Removes a tenant record when decommissioning an environment (inherits base delete).【F:app/apps/tenant/routes.py†L1-L33】 |

### Integration checklist

1. Capture generated signing keys securely during creation; they are required for
   JWT verification by downstream services.【F:app/apps/tenant/routes.py†L33-L53】
2. Configure domain lists so tenant discovery resolves correctly for each
   environment.【F:app/apps/tenant/routes.py†L1-L33】
3. Use PATCH to rotate secrets regularly and to broadcast configuration changes to
   dependent services.

## OAuth 2.0 and OIDC

### Responsibilities

* Implement the OAuth authorisation code, refresh token, and revocation flows for
  third-party clients.【F:app/apps/tenant/integration/routes.py†L1-L328】
* Provide consent screens, introspection, and userinfo payloads for OIDC relying
  parties.【F:app/apps/tenant/integration/routes.py†L29-L304】
* Publish discovery metadata and JWKS so clients can fetch configuration without
  hardcoding endpoints.【F:app/apps/tenant/integration/oidc.py†L1-L37】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `GET /oauth/authorize` | Validates client and scope requests, then directs users to login or returns consent metadata.【F:app/apps/tenant/integration/routes.py†L29-L126】 |
| `GET /oauth/consent` | Records consent decisions and issues authorization codes back to redirect URIs.【F:app/apps/tenant/integration/routes.py†L128-L184】 |
| `POST /oauth/token` | Exchanges authorization codes or refresh tokens for access/refresh tokens; supports PKCE.【F:app/apps/tenant/integration/routes.py†L186-L244】 |
| `GET /oauth/userinfo` | Returns OIDC claims for the authenticated user.【F:app/apps/tenant/integration/routes.py†L246-L273】 |
| `POST /oauth/introspect` | Validates presented tokens for resource servers.【F:app/apps/tenant/integration/routes.py†L275-L304】 |
| `POST /oauth/revoke` | Revokes refresh or access tokens according to the OAuth spec.【F:app/apps/tenant/integration/routes.py†L306-L328】 |
| `GET /.well-known/openid-configuration` | Publishes issuer metadata, endpoints, and supported grant types.【F:app/apps/tenant/integration/oidc.py†L1-L37】 |
| `GET /.well-known/jwks.json` | Exposes signing keys for JWT verification by clients.【F:app/apps/tenant/integration/oidc.py†L5-L18】 |

### Integration checklist

1. Register OAuth clients with redirect URIs, scopes, and PKCE requirements that
   match your application stack.【F:app/apps/tenant/integration/routes.py†L29-L244】
2. Host consent UX that calls `/oauth/consent` with the decision payload; USSO’s
   handler is designed to be embedded into your UI layer.【F:app/apps/tenant/integration/routes.py†L128-L184】
3. Cache the discovery document and JWKS but respect `Cache-Control` headers so
   key rotation propagates quickly.【F:app/apps/tenant/integration/oidc.py†L1-L37】
=======
# Tenant & Integration Resources

Tenant routes provision and configure organizations, while the integration
endpoints expose OAuth 2.0 and OpenID Connect features that federate identities
into third-party applications.

## Tenants (`/tenants`)

The tenant router provisions organizations, stores domain lists, and manages
crypto material for signing tokens. Creation seeds secrets automatically and
associates the authenticated creator. 【F:app/apps/tenant/routes.py†L1-L54】

Key operations:

* `POST /api/sso/v1/tenants` – Creates a tenant with domains and algorithm
  choices; the router injects freshly generated signing keys. 【F:app/apps/tenant/routes.py†L33-L53】
* `GET /api/sso/v1/tenants` – Lists tenants visible to the caller; useful for
  super-admin control planes.
* `PATCH /api/sso/v1/tenants/{uid}` – Updates metadata, domains, or activation
  flags. 【F:app/apps/tenant/routes.py†L55-L59】

Why we have it: multi-tenant deployments need a first-class resource for domain
routing, configuration overrides, and secret rotation.

## OAuth 2.0 & OIDC (`/oauth` & `/.well-known`)

The integration package implements a full authorization server: authorization,
consent, token issuance, token introspection, and revocation. 【F:app/apps/tenant/integration/routes.py†L1-L188】

Highlights:

* `GET /api/sso/v1/oauth/authorize` – Validates client and scope requests, then
  redirects unauthenticated users to login or returns consent metadata for UI
  rendering. 【F:app/apps/tenant/integration/routes.py†L29-L126】
* `GET /api/sso/v1/oauth/consent` – Records user consent and issues authorization
  codes to redirect URIs. 【F:app/apps/tenant/integration/routes.py†L128-L184】
* `POST /api/sso/v1/oauth/token` – Exchanges authorization codes or refresh tokens
  for access/refresh tokens; PKCE is supported. 【F:app/apps/tenant/integration/routes.py†L186-L244】
* `GET /api/sso/v1/oauth/userinfo` – Returns user claims for OIDC clients.
  【F:app/apps/tenant/integration/routes.py†L246-L273】
* `POST /api/sso/v1/oauth/introspect` – Validates tokens for resource servers.
  【F:app/apps/tenant/integration/routes.py†L275-L304】
* `POST /api/sso/v1/oauth/revoke` – Revokes refresh or access tokens. 【F:app/apps/tenant/integration/routes.py†L306-L328】

Discovery endpoints advertise metadata and JWKS for relying parties:

* `GET /.well-known/openid-configuration` – Publishes issuer URLs, supported
  grants, and endpoints. 【F:app/apps/tenant/integration/oidc.py†L1-L37】
* `GET /.well-known/jwks.json` – Exposes signing keys for JWT verification.
  【F:app/apps/tenant/integration/oidc.py†L5-L18】

Why this resource exists: it lets tenants act as identity providers for their own
applications, enabling SSO and OAuth integrations without deploying external auth
servers.
