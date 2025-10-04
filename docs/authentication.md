# Authentication service

The authentication router coordinates every way a person or service signs in to
USSO. Two routers mount beneath `/api/sso/v1`: the default `/auth` router and a
captcha-protected variant used for high-risk entry points.【F:app/apps/identity/auth/routes.py†L51-L123】

## Responsibilities

The module exists to:

* Discover tenant context and enforce tenant-specific security policies during
  login, registration, and token refresh flows.【F:app/apps/identity/auth/routes.py†L81-L211】
* Issue, rotate, and revoke access/refresh tokens tied to durable `AuthSession`
  records.【F:app/apps/identity/auth/services.py†L370-L612】
* Offer multiple proof mechanisms—password, magic link, OTP, passkey, OAuth, QR
  brokered approvals—without duplicating orchestration logic per tenant.【F:app/apps/identity/auth/routes.py†L195-L414】

### Module map

| File | Purpose |
| --- | --- |
| `routes.py` | Declares the FastAPI routers, guards, and route handlers for every login-related endpoint.【F:app/apps/identity/auth/routes.py†L51-L449】 |
| `services.py` | Implements the login pipeline, session persistence, OTP dispatch, OAuth callbacks, and token builders.【F:app/apps/identity/auth/services.py†L1-L997】 |
| `schemas.py` | Defines request/response models, payload enums, and validation helpers shared across routes.【F:app/apps/identity/auth/schemas.py†L1-L163】 |
| `enums.py` | Models identifiers, secrets, and multi-step authentication flows that tenants compose into policies.【F:app/apps/identity/auth/enums.py†L9-L220】 |
| `validator.py` | Optional KYC hook that verifies identities with external services before issuing tokens.【F:app/apps/identity/auth/validator.py†L1-L44】 |
| `utils.py` | Helper functions for normalising identifiers, constructing URLs, and parsing magic-link payloads.【F:app/apps/identity/auth/utils.py†L1-L275】 |

## Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /auth/login` | Runs the login pipeline; optionally registers new users; returns access/refresh tokens and profile payloads.【F:app/apps/identity/auth/routes.py†L81-L188】 |
| `GET /auth/refresh` | Rotates access tokens using the refresh cookie for browser clients.【F:app/apps/identity/auth/routes.py†L125-L150】 |
| `POST /auth/refresh` | Accepts a refresh token in the body for non-cookie clients and issues a fresh token pair.【F:app/apps/identity/auth/routes.py†L150-L188】 |
| `GET /auth/logout` / `POST /auth/logout` | Revokes the refresh session and clears authentication cookies/headers.【F:app/apps/identity/auth/routes.py†L189-L211】 |
| `POST /auth/request-magic-link` | Generates a signed login URL and dispatches it over the identifier’s default channel.【F:app/apps/identity/auth/routes.py†L195-L242】 |
| `GET /auth/magic-link` | Validates the token embedded in the magic link and resumes the login pipeline.【F:app/apps/identity/auth/routes.py†L242-L274】 |
| `POST /auth/request-otp` | Sends an OTP via SMS, email, or authenticator app after passing captcha/rate-limit checks.【F:app/apps/identity/auth/routes.py†L313-L343】 |
| `POST /auth/passkey` | Orchestrates WebAuthn authentication challenges for registered authenticators.【F:app/apps/identity/auth/routes.py†L276-L311】 |
| `POST /auth/qr` | Initiates a cross-device QR flow and returns a polling token for the client.【F:app/apps/identity/auth/routes.py†L252-L312】 |
| `GET /auth/qr/status` | Polls the QR session status until the companion device approves or rejects the login.【F:app/apps/identity/auth/routes.py†L252-L312】 |
| `GET /auth/oauth/{provider}/authorize` | Redirects to the configured OAuth provider and stores state in an HTTP-only cookie.【F:app/apps/identity/auth/routes.py†L344-L380】 |
| `GET /auth/oauth/{provider}/callback` | Exchanges provider credentials, updates tenant identity records, and resumes login.【F:app/apps/identity/auth/routes.py†L380-L414】 |
| `POST /auth/verify-token` | Validates an access or refresh token and returns the canonical payload data for internal services.【F:app/apps/identity/auth/routes.py†L416-L437】 |
| `GET /auth/me` | Returns the authenticated session payload derived from cookies or headers for client introspection.【F:app/apps/identity/auth/routes.py†L438-L449】 |

## Core workflows

### Login pipeline

1. **Tenant discovery** – Resolve the tenant from headers or domain and load any
   temporary overrides (pre-verified identities, one-time flows).【F:app/apps/identity/auth/routes.py†L87-L123】
2. **Flow processing** – `process_login_flow` validates identifiers, secrets, and
   tenant policy before optionally creating a user and stamping audit events.【F:app/apps/identity/auth/services.py†L86-L234】
3. **Token issuance** – `create_token_response` issues signed access/refresh
   tokens, persists the session, and collects request metadata for anomaly
   detection.【F:app/apps/identity/auth/services.py†L370-L461】
4. **Response normalisation** – `build_login_response` standardises headers,
   cookies, and JSON payloads across browser and native clients.【F:app/apps/identity/auth/services.py†L562-L612】

### Refresh and logout

* Refresh endpoints call `validate_refresh_session` to confirm the token is
  active and matches the stored session footprint before rotating secrets.【F:app/apps/identity/auth/services.py†L461-L562】
* Logout handlers delete the session record, revoke downstream refresh tokens,
  and clear cookies in one place.【F:app/apps/identity/auth/routes.py†L189-L211】

## Passwordless and MFA options

| Feature | Notes |
| --- | --- |
| Magic links | Signs a JWT containing the identifier, sends via the default channel, and replays the login pipeline when redeemed.【F:app/apps/identity/auth/routes.py†L195-L274】 |
| OTP | Supports SMS, email, and TOTP delivery while respecting captcha and tenant rate limits via shared dependencies.【F:app/apps/identity/auth/routes.py†L313-L343】 |
| QR sign-in | Demonstrates a brokered approval flow between devices; ships with an in-memory store that should be replaced in production deployments.【F:app/apps/identity/auth/routes.py†L252-L312】 |
| Passkeys/WebAuthn | Integrates with tenant relying-party metadata to validate authenticator challenges.【F:app/apps/identity/auth/routes.py†L276-L311】 |

## Delegated identity (OAuth)

OAuth providers use shared authorisation/callback handlers that are agnostic of
the underlying provider. The callback exchanges the code/token, stores provider
credentials on the tenant identity, and resumes the login pipeline so downstream
logic remains uniform.【F:app/apps/identity/auth/routes.py†L344-L414】

## Token inspection helpers

* `POST /auth/verify-token` decodes tokens and returns the canonical
  `PayloadData`, enabling service-to-service introspection without reimplementing
  signature checks.【F:app/apps/identity/auth/routes.py†L416-L437】
* `GET /auth/me` resolves the active session using the same dependency stack as
  other routers to keep introspection consistent.【F:app/apps/identity/auth/routes.py†L438-L449】

## Configuration and policy building blocks

The enums module captures the primitives tenants combine into login policies:

* `AuthIdentifier` normalises supported lookup keys (email, phone, username,
  service IDs) and returns the validator for each type.【F:app/apps/identity/auth/enums.py†L9-L37】
* `AuthSecret` describes which proof methods (password, OTP, passkey, OAuth) are
  permitted for each identifier and how they mark identities as verified.【F:app/apps/identity/auth/enums.py†L39-L79】
* `AuthFlow` and `AuthStep` express multi-step policies (e.g., password + OTP)
  with validation ensuring the flow is coherent before use.【F:app/apps/identity/auth/enums.py†L82-L220】

## Integration checklist

1. Configure tenant login policies with the identifiers, secrets, and flow steps
   you want to enforce.【F:app/apps/identity/auth/enums.py†L9-L220】
2. Register redirect URIs and client credentials for OAuth providers referenced
   in `/auth/oauth/{provider}` routes.【F:app/apps/identity/auth/routes.py†L344-L414】
3. Provision captcha keys in environments that should guard high-risk routes via
   the captcha-protected router.【F:app/apps/identity/auth/routes.py†L51-L123】
4. Replace the demo QR/session store and OTP delivery hooks with your production
   transport (Redis, email/SMS provider) before launch.【F:app/apps/identity/auth/routes.py†L252-L343】

Password reset and manual activation endpoints are stubs today; extend the router
with tenant-specific logic where required.【F:app/apps/identity/auth/routes.py†L344-L373】
