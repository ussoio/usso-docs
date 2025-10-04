# Machine credentials

USSO exposes two credential types for machine-to-machine access: API keys for
simple shared-secret integrations and agents for JWT-backed service accounts.
Both respect tenant scopes and reuse the authentication session infrastructure to
produce familiar access tokens.【F:app/apps/identity/api_key/routes.py†L10-L74】【F:app/apps/identity/agent/routes.py†L11-L74】

## API keys

### Responsibilities

* Generate revocable secrets tied to the creating user so downstream access can
  be audited by owner and scope.【F:app/apps/identity/api_key/routes.py†L34-L75】
* Enforce scope assignment rules that prevent callers from granting permissions
  they do not already hold.【F:app/apps/identity/api_key/routes.py†L34-L57】
* Provide a verification endpoint that rebuilds the canonical `UserData`
  structure for downstream services.【F:app/apps/identity/api_key/routes.py†L71-L75】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /api-keys` | Generates a key, stores the hash, and returns the plaintext secret once.【F:app/apps/identity/api_key/routes.py†L57-L69】 |
| `GET /api-keys` | Lists the caller’s keys with scope and audit metadata.【F:app/apps/identity/api_key/routes.py†L34-L57】 |
| `POST /api-keys/verify` | Validates a presented key and returns the resolved `UserData` payload.【F:app/apps/identity/api_key/routes.py†L69-L75】 |
| `DELETE /api-keys/{uid}` | Revokes a key by removing the stored hash (inherited delete handler).【F:app/apps/identity/api_key/routes.py†L34-L57】 |

### Integration checklist

1. Capture the plaintext secret at creation; USSO never returns it again.
2. Store hashes or secrets in your secret manager and rotate keys on a fixed
   schedule.
3. Use the verification endpoint when backend services need to convert a key
   into the standard authentication payload without calling `/auth/login`.

### Supporting modules

* `apps.identity.api_key.schemas` defines the API key data structures returned to
  clients.【F:app/apps/identity/api_key/schemas.py†L1-L63】
* `apps.identity.api_key.services` handles hashing, scope validation, and key
  material generation.【F:app/apps/identity/api_key/services.py†L1-L104】

## Agents

### Responsibilities

* Represent long-lived service accounts that authenticate using signed JWTs and
  receive regular USSO access/refresh tokens in response.【F:app/apps/identity/agent/routes.py†L11-L71】
* Honour tenant configuration flags (`tenant.config.agents.enabled`) before
  provisioning credentials.【F:app/apps/identity/agent/routes.py†L32-L38】
* Share the login response builder with human flows so cookies/headers remain
  consistent for downstream APIs.【F:app/apps/identity/agent/routes.py†L47-L71】【F:app/apps/identity/auth/services.py†L370-L612】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /agents` | Creates an agent record and returns the one-time client secret when enabled.【F:app/apps/identity/agent/routes.py†L32-L47】 |
| `GET /agents` | Lists agents for the tenant (inherits list filters from the base router).【F:app/apps/identity/agent/routes.py†L11-L47】 |
| `PATCH /agents/{uid}` | Updates metadata, scopes, or activation flags for an agent.【F:app/apps/identity/agent/routes.py†L11-L47】 |
| `DELETE /agents/{uid}` | Revokes an agent by deleting the record and associated secrets.【F:app/apps/identity/agent/routes.py†L11-L47】 |
| `POST /agents/auth` | Accepts a signed JWT, validates it, and issues an access/refresh token pair.【F:app/apps/identity/agent/routes.py†L47-L71】 |

### Integration checklist

1. Provision private keys per agent and restrict their storage; the plaintext
   client secret is shown only once.【F:app/apps/identity/agent/services.py†L1-L86】
2. Configure JWT audiences and expiration windows that align with your service
   mesh; expired or mismatched tokens are rejected by `agent_auth`.
3. Use agent-issued access tokens to call other routers just like human-issued
   tokens—they carry the same payload fields.【F:app/apps/identity/auth/services.py†L370-L612】

### Supporting modules

* `apps.identity.agent.schemas` models agent metadata and authentication
  payloads.【F:app/apps/identity/agent/schemas.py†L1-L79】
* `apps.identity.agent.services` generates secrets, validates JWTs, and produces
  session responses.【F:app/apps/identity/agent/services.py†L1-L186】
