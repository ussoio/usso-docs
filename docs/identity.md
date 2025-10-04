# Identity directory

Identity resources manage tenant members, personal data, active sessions, and
referral programs. Every router inherits from `AbstractTenantUSSORouter`, which
enforces tenant context, authentication, and authorization before any database
access.【F:app/utils/usso_routes.py†L1-L24】

## Resource overview

| Resource | Path | Primary purpose |
| --- | --- | --- |
| Users | `/users` | Canonical account records, identifier management, roles, scopes, and workspace memberships.【F:app/apps/identity/user/routes.py†L17-L197】 |
| Profiles | `/profiles` | Personal metadata stored separately from authentication identifiers.【F:app/apps/identity/profile/routes.py†L11-L55】 |
| Sessions | `/sessions` | Audit trail for active refresh sessions and granted scopes.【F:app/apps/identity/session/routes.py†L12-L37】 |
| Referrals | `/referrals` | Invitation codes and campaign attribution for onboarding flows.【F:app/apps/identity/referral/routes.py†L1-L14】 |

The sections below break each resource into responsibilities, endpoints, backing
modules, and integration tips.

## Users

### Responsibilities

* Persist a user’s identifiers, credentials, role/scope assignments, and tenant
  activation flags.【F:app/apps/identity/user/routes.py†L17-L197】
* Delegate identifier and credential CRUD to nested routers so each identifier
  can be verified independently.【F:app/apps/identity/user/routes.py†L24-L119】
* Reuse the login pipeline during user creation to avoid duplicate identifiers
  and ensure tenant defaults are applied uniformly.【F:app/apps/identity/user/routes.py†L129-L151】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `GET /users` | Paginate users with filters for identifiers, activation status, roles, and scopes.【F:app/apps/identity/user/routes.py†L95-L119】 |
| `GET /users/{uid}` | Return a single user including identifiers, credentials, roles, and scope grants.【F:app/apps/identity/user/routes.py†L61-L93】 |
| `POST /users` | Create a user via the login pipeline; enforces unique identifiers and tenant defaults.【F:app/apps/identity/user/routes.py†L129-L151】 |
| `PATCH /users/{uid}` | Update profile links, permission grants, or activation flags; triggers session refresh when permissions change.【F:app/apps/identity/user/routes.py†L153-L178】 |
| `DELETE /users/{uid}` | Soft delete a user; downstream tokens become invalid once sessions are revoked.【F:app/apps/identity/user/routes.py†L179-L197】 |
| `GET/POST /users/{uid}/identifiers` | Manage verified identifiers (email, phone, username, service IDs).【F:app/apps/identity/user/routes.py†L24-L119】 |
| `GET/POST /users/{uid}/credentials` | Rotate passwords, passkeys, or OTP seeds tied to identifiers.【F:app/apps/identity/user/routes.py†L24-L119】 |

### Integration tips

1. Use list filters (`identifier_type`, `role`, `is_active`) to audit members
   after policy updates.【F:app/apps/identity/user/routes.py†L95-L119】
2. Rotate credentials via the nested endpoints instead of deleting users when you
   need scoped revocation (e.g., passkey reset).【F:app/apps/identity/user/routes.py†L24-L119】
3. Extend the placeholder `/users/{uid}/impersonate` handler when you implement
   admin impersonation policies.【F:app/apps/identity/user/routes.py†L61-L93】

### Supporting modules

* Schemas – `apps.identity.user.schemas` captures identifier, credential, and
  role models exposed to clients.【F:app/apps/identity/user/schemas.py†L18-L119】
* Services – `apps.identity.user.services` provides helpers for constructing
  users via the authentication pipeline and syncing sessions.【F:app/apps/identity/user/services.py†L1-L210】

## Profiles

### Responsibilities

* Maintain personal attributes (name, locale, address) separate from login
  identifiers so profile schemas can evolve independently.【F:app/apps/identity/profile/routes.py†L11-L55】
* Guarantee a single profile per user through `unique_per_user = True` and
  auto-provision missing profiles when accessed via `/profiles/mine`.【F:app/apps/identity/profile/routes.py†L11-L55】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `GET /profiles` | Tenant admins can audit member profiles with pagination and filtering helpers.【F:app/apps/identity/profile/routes.py†L18-L55】 |
| `GET /profiles/mine` | Returns the caller’s profile, creating one if it does not exist.【F:app/apps/identity/profile/routes.py†L18-L55】 |
| `PATCH /profiles/mine` | Lets authenticated users update their own profile data with validation.【F:app/apps/identity/profile/routes.py†L18-L55】 |
| `PATCH /profiles/{uid}` | Tenant admins update another member’s profile when required.【F:app/apps/identity/profile/routes.py†L18-L55】 |

### Integration tips

* Pair profile PATCH calls with `/users/{uid}` updates when synchronising contact
  methods or marketing preferences across systems.
* Extend the schema module to add custom profile fields; the routers will expose
  them automatically.【F:app/apps/identity/profile/routes.py†L11-L55】

## Sessions

### Responsibilities

* Track refresh sessions, issued scopes, `amr` claims, and device metadata for
  auditing and anomalous login detection.【F:app/apps/identity/session/routes.py†L12-L37】【F:app/apps/identity/session/schemas.py†L25-L73】
* Allow administrators to revoke individual sessions without deleting the user or
  invalidating all tokens simultaneously.【F:app/apps/identity/session/routes.py†L20-L36】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `GET /sessions` | Paginate sessions with filters for user ID and creation ranges.【F:app/apps/identity/session/routes.py†L20-L36】 |
| `DELETE /sessions/{uid}` | Revoke a specific session via the inherited delete handler.【F:app/apps/identity/session/routes.py†L12-L37】 |

### Integration tips

* Use session revocation together with `/auth/logout` to force token refresh
  after permission changes.
* Export the session audit data to your SIEM; the schema exposes device, IP, and
  authentication method details.【F:app/apps/identity/session/schemas.py†L25-L73】

## Referrals

### Responsibilities

* Generate single-use or multi-use invitation codes tied to campaigns or existing
  members.【F:app/apps/identity/referral/routes.py†L1-L14】
* Record redemption metadata so marketing and activation teams can attribute
  growth channels.【F:app/apps/identity/referral/routes.py†L1-L14】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /referrals` | Issue a new referral record according to tenant limits.【F:app/apps/identity/referral/routes.py†L1-L14】 |
| `GET /referrals` | List referral codes with pagination and filter hooks.【F:app/apps/identity/referral/routes.py†L1-L14】 |
| `DELETE /referrals/{uid}` | Revoke a code to prevent future redemptions.【F:app/apps/identity/referral/routes.py†L1-L14】 |

### Integration tips

* Extend the schema to record campaign metadata (UTM tags, partner IDs) and feed
  it into marketing analytics.
* Pair referral creation with the authentication magic-link flow to send invite
  links immediately.【F:app/apps/identity/auth/routes.py†L195-L274】
=======
# Identity Resources

Identity endpoints power core account creation, authentication, and lifecycle
management features. Every route is tenant-aware and enforces authorization
checks through `AbstractTenantUSSORouter`, which fetches the active tenant and
user context before allowing access. 【F:app/utils/usso_routes.py†L1-L24】

## Authentication (`/auth`)

The authentication router orchestrates multi-step login flows, refreshes, and
session termination while delegating risk checks (captcha, rate limiting) to
router-level dependencies. Two routers ship out of the box: the plain `/auth`
router and a captcha-enforced variant for high-risk operations. 【F:app/apps/identity/auth/routes.py†L1-L96】

### Login pipeline

`POST /api/sso/v1/auth/login` accepts any identifier/secret pair that maps to an
`AuthIdentifier` (`email`, `phone`, `username`, etc.) and an `AuthSecret`
(`password`, `otp`, `oauth`, `magic_link`, and more). After resolving the tenant
context, the handler defers to `process_login_flow`, which encapsulates the full
decision tree: verify credentials, optionally register the user, mark
identifiers as verified, and activate the account when allowed. 【F:app/apps/identity/auth/routes.py†L58-L110】【F:app/apps/identity/auth/services.py†L86-L234】

`process_login_flow` pulls together helpers from `services.py`:

* `register_new_user` provisions an account when self-service registration is
  enabled. 【F:app/apps/identity/auth/services.py†L86-L134】
* `verify_identifier` confirms that the login method validates the identifier
  and stamps `verified_at` to support trust-based activation. 【F:app/apps/identity/auth/services.py†L136-L177】
* `activate_user_if_eligible` evaluates tenant approval rules (`none`,
  `identifier`, etc.) to flip `is_active` during first login. 【F:app/apps/identity/auth/services.py†L179-L234】

Once the login flow returns a user, `create_token_response` materializes access
and refresh tokens, persists an `AuthSession`, and records request metadata such
as user agent and IP address. The response wrapper `build_login_response`
standardizes headers/cookies so frontends do not have to manipulate JWTs by
hand. 【F:app/apps/identity/auth/routes.py†L83-L110】【F:app/apps/identity/auth/services.py†L370-L461】【F:app/apps/identity/auth/services.py†L562-L612】

### Session refresh & logout

`GET /api/sso/v1/auth/refresh` and `POST /api/sso/v1/auth/refresh` both rely on
`check_session_validity` to ensure the refresh token still maps to an active
`AuthSession`. The GET variant reads the refresh cookie, whereas the POST variant
accepts a raw token for native clients. Both return a `RefreshResponse` built by
`build_refresh_response`, which re-issues access tokens without exposing refresh
tokens in the body. 【F:app/apps/identity/auth/routes.py†L112-L164】【F:app/apps/identity/auth/services.py†L614-L709】

`GET/POST /api/sso/v1/auth/logout` clears refresh cookies and marks the session
document as deleted if it still exists, enabling central revocation. Domains are
derived from tenant configuration so the cookie scope matches the requesting
origin. 【F:app/apps/identity/auth/routes.py†L166-L211】

### Passwordless flows

The router provides multiple passwordless or secondary-factor entry points, all
wrapped in captcha enforcement and per-route rate limiting:

| Endpoint | Purpose |
| --- | --- |
| `POST /api/sso/v1/auth/request-magic-link` | Generates a signed link and delivers it via the identifier’s default messaging channel. Delivery flows through the tenant messaging services configured in `create_otp`. 【F:app/apps/identity/auth/routes.py†L173-L205】【F:app/apps/identity/auth/services.py†L816-L866】 |
| `GET /api/sso/v1/auth/magic-link` | Consumes the magic link token, extracts the identifier from the JWT payload, and replays the standard login pipeline. 【F:app/apps/identity/auth/routes.py†L206-L274】 |
| `POST /api/sso/v1/auth/request-otp` | Issues a one-time code through the tenant’s configured channels, auto-selecting SMS or email based on identifier type. Channel validation lives in the `OTPRequest` schema. 【F:app/apps/identity/auth/routes.py†L313-L343】【F:app/apps/identity/auth/schemas.py†L87-L142】 |
| `POST /api/sso/v1/auth/qr` & `GET /api/sso/v1/auth/qr/status` | Implements a demo QR-login loop that stores session state in-memory; swap in Redis or a database for production usage. 【F:app/apps/identity/auth/routes.py†L252-L312】 |

Passkey/WebAuthn support is scaffolded via `POST /api/sso/v1/auth/passkey`, which
generates WebAuthn authentication options and validates authenticator responses
against the tenant’s relying-party metadata. 【F:app/apps/identity/auth/routes.py†L276-L311】

### OAuth & delegated identity

`GET /api/sso/v1/auth/oauth/{provider}/authorize` redirects to the provider
configured on the tenant, storing state in an HTTP-only cookie. The callback
(`GET /oauth/{provider}/callback`) exchanges the authorization code for userinfo,
persists the provider credential, and reuses the standard login handler with the
resulting `id_token`. 【F:app/apps/identity/auth/routes.py†L344-L414】

### Token inspection helpers

`POST /api/sso/v1/auth/verify-token` gives internal services a way to validate
access/refresh tokens and receive the canonical `PayloadData`. `GET
/api/sso/v1/auth/me` simply introspects the bearer cookie/header to fetch the
current session payload. Both use tenant-scoped token verifiers, so tokens from
other tenants are rejected. 【F:app/apps/identity/auth/routes.py†L416-L449】

### Supporting modules

The auth package also includes:

* `schemas.py` – request/response models for login, refresh, OTP, and QR flows.
  `PayloadData` extends USSO’s base `UserData` with helper serialization logic.
  【F:app/apps/identity/auth/schemas.py†L1-L163】
* `services.py` – core orchestration for login flows, token issuance, OTP
  generation, and session persistence. 【F:app/apps/identity/auth/services.py†L1-L997】
* `utils.py` – helper for inferring identifier type from magic link payloads.
  【F:app/apps/identity/auth/utils.py†L250-L275】
* `validator.py` – third-party integrations (e.g., Jibit KYC) that back certain
  tenant flows. 【F:app/apps/identity/auth/validator.py†L1-L44】

### Authentication configuration primitives

`enums.py` captures the building blocks used by tenant-specific login flows.
`AuthIdentifier` enumerates every supported lookup key (email, phone, Telegram,
QR session, service IDs, and more) and exposes a validator helper so new flows
reuse the canonical normalization logic. 【F:app/apps/identity/auth/enums.py†L9-L37】
`AuthSecret` records the available proof mechanisms—passwords, OTP variants,
magic links, passkeys, OAuth tokens, and Telegram bot tokens—and maps each
method to the identifier it verifies. 【F:app/apps/identity/auth/enums.py†L39-L79】

For richer orchestration, `AuthFlow` combines `AuthStep` objects to describe the
sequence of factors a tenant requires. Validation ensures every flow defines at
least one identifier and one step, while each step enforces uniqueness and the
number of factors needed to pass. The module ships with ready-made flows (simple
password, password + OTP, password-or-OTP) that mirror the inline examples and
serve as templates for custom policies. 【F:app/apps/identity/auth/enums.py†L82-L220】

### Not yet implemented

Placeholders exist for `POST /api/sso/v1/auth/reset-password` and `POST
/api/sso/v1/auth/activate`; both currently `pass` because the underlying
business rules are still being defined. They are intentionally left documented so
integrators know where to plug in once requirements finalize. 【F:app/apps/identity/auth/routes.py†L345-L373】

## Users (`/users`)

`UserRouter` exposes CRUD endpoints plus helper actions to manage identifiers,
credentials, and impersonation flows. Results include role, scope, and workspace
assignments for fine-grained authorization. 【F:app/apps/identity/user/routes.py†L16-L150】

Important workflows:

* `POST /api/sso/v1/users` – Creates a user given an identifier type/value and
  optional role assignments; reuses existing accounts if the identifier already
  exists. 【F:app/apps/identity/user/routes.py†L87-L123】
* `PATCH /api/sso/v1/users/{uid}` – Updates profile and permission fields,
  automatically refreshing sessions when scopes/roles change. 【F:app/apps/identity/user/routes.py†L125-L170】
* `POST /api/sso/v1/users/{uid}/identifiers` – Adds secondary identifiers (e.g.,
  extra emails) with verification tracking. 【F:app/apps/identity/user/routes.py†L37-L63】【F:app/apps/identity/user/schemas.py†L18-L76】
* `POST /api/sso/v1/users/{uid}/credentials` – Manages password or passkey
  secrets per identifier. 【F:app/apps/identity/user/routes.py†L65-L84】【F:app/apps/identity/user/schemas.py†L78-L119】
* `POST /api/sso/v1/users/{uid}/impersonate` – Reserved for administrators to
  assume another user’s identity for troubleshooting. 【F:app/apps/identity/user/routes.py†L25-L36】【F:app/apps/identity/user/routes.py†L172-L197】

Why we have it: tenants need a unified view of their members, including
credentials, identifiers, and roles, to orchestrate activation flows and audit
changes.

## Profiles (`/profiles`)

Profiles separate personal metadata from identity records and support a “mine”
endpoint so authenticated users can manage their own profile. The router enforces
unique profiles per user and auto-creates missing records. 【F:app/apps/identity/profile/routes.py†L5-L44】

Key usage:

* `GET /api/sso/v1/profiles/mine` – Fetches the caller’s profile, creating one if
  needed. 【F:app/apps/identity/profile/routes.py†L13-L24】【F:app/apps/identity/profile/routes.py†L32-L36】
* `PATCH /api/sso/v1/profiles/mine` – Allows users to self-update profile
  details. 【F:app/apps/identity/profile/routes.py†L16-L23】【F:app/apps/identity/profile/routes.py†L38-L44】
* Tenant admins can manage other profiles via `PATCH /profiles/{uid}`. 【F:app/apps/identity/profile/routes.py†L24-L31】

Why this resource exists: it decouples personal/contact data from login
credentials so applications can evolve profile schemas independently.

## Sessions (`/sessions`)

Sessions capture the authentication method, scope grants, and expiration state
for each login or API credential. They support filtering by user and creation
windows for audits. 【F:app/apps/identity/session/routes.py†L1-L33】【F:app/apps/identity/session/schemas.py†L1-L73】

Usage highlights:

* `GET /api/sso/v1/sessions` – Lists sessions with pagination and optional
  filters (`user_id`, `created_at` range) for compliance checks. 【F:app/apps/identity/session/routes.py†L19-L33】
* Session documents record auth methods (`amr`), granted scopes, and sliding
  expiration metadata via `max_age_minutes`. 【F:app/apps/identity/session/schemas.py†L25-L73】

Why we have it: visibility into active sessions enables forced logouts when
roles change and supports security analytics.

## API Keys (`/api-keys`)

The API key router lets tenants create scoped programmatic credentials and
verify incoming keys. Creation enforces that the requester cannot grant scopes
they do not already possess. 【F:app/apps/identity/api_key/routes.py†L1-L70】

How to use it:

* `POST /api/sso/v1/api-keys` – Issues a new key after generating a hashed
  secret and returning the raw value once. 【F:app/apps/identity/api_key/routes.py†L34-L63】【F:app/apps/identity/api_key/schemas.py†L25-L51】
* `GET /api/sso/v1/api-keys` – Lists active keys for the caller with usage
  metadata. 【F:app/apps/identity/api_key/routes.py†L26-L33】【F:app/apps/identity/api_key/schemas.py†L11-L31】
* `POST /api/sso/v1/api-keys/verify` – Validates a presented key and resolves it
  to a `UserData` payload for downstream authorization. 【F:app/apps/identity/api_key/routes.py†L17-L24】【F:app/apps/identity/api_key/routes.py†L65-L70】

Why this resource exists: it provides first-party API credentials, including
scope enforcement and rotation metadata, without leaking hashed secrets.

## Agents (`/agents`)

Agents model service accounts that can authenticate with JWTs and receive scoped
USSO tokens. The router subclasses the shared tenant router to inherit CRUD
behaviour, then adds a bespoke `/auth` route that accepts an agent-signed JWT,
validates it, and issues first-party tokens via the same response builder used
for user logins. 【F:app/apps/identity/agent/routes.py†L1-L61】

Workflows:

* `POST /api/sso/v1/agents` – Provisions an agent, generating client secrets and
  associating it with a tenant. Agent creation is gated by tenant configuration
  so operators can disable the feature. The heavy lifting is delegated to
  `services.generate_agent`, which stores hashed secrets and returns the one-time
  plaintext to the caller. 【F:app/apps/identity/agent/routes.py†L21-L45】【F:app/apps/identity/agent/services.py†L1-L86】
* `POST /api/sso/v1/agents/auth` – Exchanges an agent-issued JWT for an access
  token usable against other routes. 【F:app/apps/identity/agent/routes.py†L13-L20】【F:app/apps/identity/agent/routes.py†L39-L64】

Why we have it: agents act as non-human actors (e.g., background jobs) that need
first-class authentication with scoped access similar to users.

## Referral Codes (`/referrals`)

Referral codes track invitation flows for tenants and reuse the tenant-aware
router for CRUD operations. 【F:app/apps/identity/referral/routes.py†L1-L14】

Use `POST /api/sso/v1/referrals` to mint codes and `GET /api/sso/v1/referrals`
to audit usage, leveraging the base router’s pagination and filtering support.
