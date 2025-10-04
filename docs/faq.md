# Frequently Asked Questions (FAQ)

Common questions about USSO, answered.

---

## General Questions

### What is USSO?

USSO (Universal Single Sign-On) is an open-source, multi-tenant identity and authentication platform. It provides authentication, authorization, and user management for modern applications.

### Who should use USSO?

- **Startups** building SaaS applications
- **Developers** who need authentication without the complexity
- **Companies** wanting self-hosted identity management
- **Teams** building multi-tenant platforms

### Is USSO production-ready?

USSO is in active development (v0.x). The core features are stable and being used in production, but the API may change before v1.0. We recommend thorough testing before production use.

### How is USSO different from Auth0, Firebase, or Keycloak?

| Feature | USSO | Auth0 | Firebase | Keycloak |
|---------|------|-------|----------|----------|
| **Open Source** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Self-Hosted** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Multi-Tenant** | ✅ Native | ✅ Yes | ❌ Limited | ⚠️ Complex |
| **Modern Stack** | ✅ FastAPI | ⚠️ Proprietary | ⚠️ Firebase | ⚠️ Java |
| **Easy Setup** | ✅ Docker | ✅ SaaS | ✅ SaaS | ⚠️ Complex |

---

## Getting Started

### How do I install USSO?

```bash
git clone https://github.com/ussoio/usso.git
cd usso
cp sample.env .env
docker compose up -d
```

[Full installation guide →](getting-started/quickstart.md)

### What are the system requirements?

- **Docker** and **Docker Compose**
- **2GB RAM** minimum (4GB recommended)
- **10GB disk** space
- **MongoDB** 4.4+ (included in Docker setup)
- **Redis** 6.0+ (included in Docker setup)

### Can I use USSO without Docker?

Yes, but Docker is recommended. Manual installation requires:
- Python 3.10+
- MongoDB
- Redis
- Knowledge of Python deployment

### How long does setup take?

- **Docker setup**: 5-10 minutes
- **First user creation**: 2 minutes
- **Integration with your app**: 15-30 minutes

---

## Features

### What authentication methods does USSO support?

- ✅ Email/password
- ✅ Magic links (passwordless email)
- ✅ OTP (SMS and email)
- ✅ OAuth/OIDC (Google, GitHub, etc.)
- ✅ Passkeys (WebAuthn)
- ✅ QR code login
- 🚧 Telegram (coming soon)
- 🚧 TOTP (coming soon)

### Does USSO support multi-factor authentication (MFA)?

Yes! USSO supports:
- OTP via SMS/email
- Passkeys (WebAuthn)
- TOTP (planned)

### Can users have multiple login methods?

Yes! USSO supports **linked accounts**. A user can have:
- Email/password
- Google OAuth
- GitHub OAuth
- Phone number (for OTP)

All linked to one identity.

### Does USSO support social login?

Yes! Configure OAuth providers in your tenant settings:
- Google
- GitHub
- GitLab
- Custom OAuth 2.0 providers

### What is the difference between roles and scopes?

- **Roles** are bundles of permissions (e.g., "admin", "editor")
- **Scopes** are individual permissions (e.g., "read:users", "write:posts")

Roles contain scopes. [Learn more →](concepts/auth-vs-authz.md)

### What are workspaces?

Workspaces are **data isolation boundaries** within a tenant. Use them to:
- Separate teams
- Isolate projects
- Segment customers

[Learn more →](concepts/workspaces-roles.md)

---

## Multi-Tenancy

### What is a tenant?

A tenant is an **organization or company** using your application. Each tenant has:
- Its own users
- Separate data
- Independent configuration
- Custom branding

### How many tenants can USSO handle?

There's no hard limit. USSO can handle hundreds or thousands of tenants on a single instance.

### Can users belong to multiple tenants?

Not by default. Each user belongs to one tenant. However, you can implement cross-tenant access in your application logic.

### How is tenant data isolated?

All data includes a `tenant_id` field. USSO automatically filters queries by tenant, ensuring complete isolation.

### Can tenants have custom domains?

Yes! Each tenant can have multiple domains:

```python
{
  "tenant_id": "org_company_a",
  "domains": ["company-a.com", "app.company-a.com"]
}
```

---

## Security

### How does USSO store passwords?

Passwords are hashed using **Argon2id**, the winner of the Password Hashing Competition. Argon2id is:
- Memory-hard (resistant to GPU attacks)
- Slow by design (resistant to brute force)
- Industry best practice

### Are tokens secure?

Yes! USSO uses:
- **EdDSA (Ed25519)** by default (modern, fast, secure)
- **Short-lived access tokens** (1 hour default)
- **Revocable refresh tokens**
- **Signature verification** via JWKS

### Can I use USSO in production?

USSO is being used in production, but is still in v0.x (pre-1.0). We recommend:
- Thorough testing
- Monitoring and logging
- Regular backups
- Staying updated with releases

### How do I report security vulnerabilities?

Email security@usso.io with details. Please **do not** open public GitHub issues for security vulnerabilities.

### Does USSO support HTTPS?

USSO runs on HTTP by default. For production, use a reverse proxy (Traefik, Nginx, etc.) for HTTPS termination.

---

## Integration

### How do I integrate USSO with my app?

1. Install the SDK: `pip install usso`
2. Configure JWKS URL
3. Add authentication dependency
4. Check scopes for authorization

[Full integration guide →](integration/python-sdk.md)

### Does USSO have SDKs?

- ✅ **Python SDK**: [pip install usso](https://pypi.org/project/usso/)
- 🚧 **JavaScript/TypeScript SDK**: Planned
- 🚧 **Go, Java, PHP SDKs**: Planned

### Can I use USSO with any programming language?

Yes! USSO is a REST API with JWT tokens. Any language that can:
- Make HTTP requests
- Verify JWT signatures

Can integrate with USSO.

### Does USSO work with mobile apps?

Yes! Use the OAuth authorization code flow or direct API calls.

### Can I use USSO as an OAuth provider?

Yes! USSO supports OAuth 2.0 and OpenID Connect, so you can offer "Sign in with Your App" to third parties.

[OAuth Provider guide →](oauth-provider/overview.md)

---

## Deployment

### How do I deploy USSO to production?

1. **Use Docker** for consistent deployment
2. **Use external MongoDB** (not Docker container)
3. **Use external Redis** (not Docker container)
4. **Set up reverse proxy** (Nginx, Traefik) for HTTPS
5. **Configure monitoring** (Sentry, Datadog, etc.)
6. **Set up backups** for MongoDB

[Production deployment guide →](deployment/production.md)

### Can I use a managed MongoDB?

Yes! USSO works with:
- MongoDB Atlas
- AWS DocumentDB
- Azure Cosmos DB
- Any MongoDB-compatible database

### Can I scale USSO horizontally?

Yes! USSO is **stateless** and can be scaled horizontally:
- Run multiple USSO instances
- Use a load balancer
- Share MongoDB and Redis

### What are the performance characteristics?

Typical performance per instance:
- **10,000+** concurrent users
- **1,000+** requests per second
- **< 10ms** token verification
- **< 100ms** login request

### How much does it cost to run USSO?

USSO is **open source and free**. Infrastructure costs depend on your deployment:

**Small (< 1,000 users):**
- 1 server ($10-20/month)
- MongoDB Atlas free tier
- Redis Cloud free tier
- **Total: $10-20/month**

**Medium (< 10,000 users):**
- 2-3 servers ($50-100/month)
- Managed MongoDB ($50/month)
- Managed Redis ($20/month)
- **Total: $120-170/month**

---

## Development

### Is USSO open source?

Yes! USSO is licensed under **Apache 2.0**.

### Can I contribute to USSO?

Yes! We welcome contributions:
- Bug reports
- Feature requests
- Code contributions
- Documentation improvements

[Contributing guide →](contributing.md)

### How do I request a feature?

Open a discussion on [GitHub Discussions](https://github.com/ussoio/usso/discussions) with:
- Problem description
- Proposed solution
- Use case examples

### Where can I get help?

- **Documentation**: This site
- **GitHub Issues**: Bug reports and questions
- **GitHub Discussions**: Feature requests and discussions
- **Email**: support@usso.io

---

## Roadmap

### When will v1.0 be released?

Target: **Q3 2026**

Before v1.0, we want to achieve:
- Stable API
- 90%+ test coverage
- Complete documentation
- Security audit

### What features are planned?

See our [roadmap](roadmap.md) for details. Highlights:
- User impersonation
- TOTP/Telegram auth
- Enhanced KYC
- Rust implementation
- More SDKs

### Can I influence the roadmap?

Yes! Vote on features in [GitHub Discussions](https://github.com/ussoio/usso/discussions).

---

## Troubleshooting

### Why can't I login?

Common issues:
1. **Wrong password**: Reset via email
2. **Account not activated**: Check activation email
3. **Account disabled**: Contact admin
4. **Rate limited**: Wait a few minutes

### Why is token verification failing?

Common issues:
1. **Wrong JWKS URL**: Check configuration
2. **Token expired**: Refresh token
3. **Wrong audience**: Token for different app
4. **Clock skew**: Sync server clocks

### Why am I getting 403 Forbidden?

**403** means you're authenticated but not authorized. Check:
1. Do you have the required scope?
2. Are you in the correct workspace?
3. Is your role active?

### How do I reset the admin password?

Set `SYSTEM_PASSWORD` in `.env` file and restart USSO.

### Where are logs stored?

Logs are in:
- **Console output**: Docker logs
- **File**: `app/logs/app.log`

View logs:

```bash
docker compose logs -f app
```

---

## Migration

### Can I migrate from Auth0/Firebase?

Yes! We're building migration tools (planned Q1 2026). Current workaround:
1. Export users from Auth0/Firebase
2. Import via USSO API
3. Users reset passwords on first login

### Can I export my data from USSO?

Yes! All data is in MongoDB. You can:
- Export via `mongodump`
- Query via MongoDB client
- Use USSO APIs

### Is there a migration guide?

Not yet. Migration guides are planned for Q1 2026.

---

## Pricing

### Is USSO free?

**Self-hosted**: Yes, forever free and open source.

**Managed service**: Coming 2026 with usage-based pricing.

### Do you offer support?

- **Community support**: Free via GitHub
- **Enterprise support**: Coming 2026

---

## Comparison

### USSO vs Auth0

| Feature | USSO | Auth0 |
|---------|------|-------|
| **Cost** | Free (self-hosted) | $0-$150+/month |
| **Open Source** | ✅ Yes | ❌ No |
| **Self-Hosted** | ✅ Yes | ❌ No |
| **Data Control** | ✅ Full | ❌ Limited |
| **Customization** | ✅ Unlimited | ⚠️ Limited |

### USSO vs Keycloak

| Feature | USSO | Keycloak |
|---------|------|----------|
| **Setup** | ✅ Easy | ⚠️ Complex |
| **Technology** | FastAPI (Python) | Java |
| **Multi-Tenancy** | ✅ Native | ⚠️ Realms |
| **Modern** | ✅ Yes | ⚠️ Older |

---

## Still have questions?

- **GitHub**: [Open an issue](https://github.com/ussoio/usso/issues)
- **Discussions**: [Ask the community](https://github.com/ussoio/usso/discussions)
- **Email**: support@usso.io

---

[← Back to Home](index.md){ .md-button }
[View Contributing →](contributing.md){ .md-button .md-button--primary }

