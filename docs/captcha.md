# Captcha

USSO ships with a lightweight captcha service inspired by Altcha. It issues
proof-of-work challenges that protect sensitive flows such as login, OTP, and
magic-link requests without introducing external dependencies.【F:app/apps/captcha/routes.py†L1-L20】【F:app/apps/captcha/schemas.py†L1-L71】

## Responsibilities

* Generate signed challenges with adjustable difficulty and expiry windows so
  tenants can throttle automated traffic.【F:app/apps/captcha/schemas.py†L21-L58】
* Provide a validation dependency (`verify_captcha`) that upstream routers attach
  to rate-limited endpoints.【F:app/apps/captcha/dependencies.py†L1-L80】
* Emit responses compatible with Altcha widgets, making it easy to embed in web
  clients.【F:app/apps/captcha/schemas.py†L1-L58】

## Endpoint map

| Method & path | Summary |
| --- | --- |
| `GET /captcha/challenge` | Returns a signed challenge payload containing salt, max search value, difficulty, and expiry.【F:app/apps/captcha/routes.py†L8-L16】 |
| `POST /captcha/validate` | Validates a solved challenge and returns a confirmation token for downstream use.【F:app/apps/captcha/routes.py†L18-L21】 |

## Integration checklist

1. Fetch a challenge before calling high-risk routes and display it to users via
   an Altcha-compatible widget.【F:app/apps/captcha/routes.py†L8-L16】
2. Submit the solved payload to `/captcha/validate`; include the returned token in
   subsequent requests guarded by the captcha dependency.【F:app/apps/captcha/routes.py†L18-L21】【F:app/apps/captcha/dependencies.py†L1-L80】
3. Adjust difficulty, expiry, and salt length in tenant configuration or by
   extending the schema defaults to match your threat model.【F:app/apps/captcha/schemas.py†L21-L58】
=======
# Captcha Resource

The captcha resource issues computational challenges (compatible with Altcha)
that protect sensitive flows like login and magic link requests. 【F:app/apps/captcha/routes.py†L1-L20】【F:app/apps/captcha/schemas.py†L1-L71】

## Why it exists

Brute-force defenses are critical for identity systems. The captcha endpoints
hand out signed challenges with short expirations and configurable difficulty so
tenants can throttle automation without external dependencies. 【F:app/apps/captcha/schemas.py†L21-L58】

## How to use it

1. `GET /api/sso/v1/captcha/challenge` to obtain a random challenge payload. The
   response includes the salt, signature, expiry, and maximum search number. 【F:app/apps/captcha/routes.py†L8-L16】【F:app/apps/captcha/schemas.py†L21-L58】
2. Solve the puzzle client-side and submit the encoded payload to
   `POST /api/sso/v1/captcha/validate`. The dependency `verify_captcha` checks the
   payload before allowing the request to continue. 【F:app/apps/captcha/routes.py†L18-L21】【F:app/apps/captcha/dependencies.py†L1-L80】

Successful validation returns a confirmation message and allows the caller to
proceed with the gated operation (e.g., login). 【F:app/apps/captcha/routes.py†L18-L21】
