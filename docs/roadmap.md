# Roadmap

USSO is actively developed with new features and improvements. This page outlines planned features, improvements, and the development timeline.

---

## 🎯 Current Status

USSO is currently in **active development** (v0.x). The core features are stable and production-ready, but the API may change before v1.0.

### Production Ready ✅

- Multi-tenant architecture
- Authentication (Password, OAuth, Magic Link, OTP, Passkeys)
- Authorization (RBAC, Scopes, Workspaces)
- Service accounts & API keys
- OAuth/OIDC provider
- Session management
- Token lifecycle

### In Progress 🚧

- Enhanced KYC integrations
- Advanced user management features
- Performance optimizations

---

## 📅 Development Timeline

### Q4 2025 - Core Enhancements

#### User Management

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Reset User Secret** | 📋 Planned | High | Allow admins to reset user passwords and credentials |
| **User Impersonation** | 📋 Planned | High | Support staff impersonating users for troubleshooting |
| **User Visibility Controls** | 📋 Planned | Medium | Control which users can see each other |
| **Multi-Account Linking** | 📋 Planned | Medium | Link multiple identities to single user |

#### Authentication

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **QR Code Login** | 🚧 In Progress | High | WhatsApp Web-style cross-device authentication |
| **TOTP Support** | 📋 Planned | High | Time-based OTP for MFA |
| **Telegram Authentication** | 📋 Planned | Medium | Login via Telegram bot |
| **WebFlow Authentication** | 📋 Planned | Low | Custom web-based authentication flows |

#### Authorization

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Role Scope Expiration** | 📋 Planned | Medium | Time-limited role assignments |
| **Device Permissions** | 📋 Planned | Medium | Per-device permission control |
| **Scope Management API** | 📋 Planned | High | Dynamic scope registration for applications |

---

### Q1 2026 - Enterprise Features

#### Tenant Management

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Tenant Registration Flow** | 📋 Planned | High | Self-service tenant registration |
| **Domain Ownership Verification** | 📋 Planned | High | Verify tenant owns claimed domains |
| **Tenant Deletion** | 📋 Planned | Medium | Safe tenant removal with data export |
| **Tenant Retrieval API** | 📋 Planned | High | Enhanced tenant information endpoints |
| **Tenant Update API** | 📋 Planned | High | Update tenant configuration |
| **Tenant List API** | 📋 Planned | Medium | Paginated tenant listing |

#### KYC (Know Your Customer)

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Phone National ID** | 📋 Planned | High | Verify users via national phone databases |
| **Birth Date Verification** | 📋 Planned | Medium | Age verification and birth date validation |
| **Sejam Integration** | 📋 Planned | Medium | Integration with Iranian stock market ID system |

#### Workspace Enhancements

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Workspace Permissions** | 📋 Planned | High | Fine-grained workspace-level permissions |
| **Workspace Templates** | 📋 Planned | Low | Pre-configured workspace setups |
| **Workspace Analytics** | 📋 Planned | Low | Usage metrics per workspace |

---

### Q2 2026 - Integration & Performance

#### Session Management

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Session Management UI** | 📋 Planned | Medium | User-facing session management interface |
| **Lightweight Refresh Tokens** | 📋 Planned | High | Optimize refresh token structure |
| **Session Analytics** | 📋 Planned | Low | Track session patterns and anomalies |

#### OAuth Enhancements

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **OAuth Session Cleanup** | 📋 Planned | Medium | Remove OAuth session management overhead |
| **Enhanced OAuth Provider** | 📋 Planned | High | Additional OAuth 2.0 features |

#### Registration

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Registration Webhooks** | 📋 Planned | High | Notify external systems on user registration |
| **Custom Registration Flow** | 📋 Planned | Medium | Configurable registration workflows |

#### Referral System

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Referral Analytics** | 📋 Planned | Medium | Track referral effectiveness |
| **Referral Rewards** | 📋 Planned | Low | Automated reward distribution |

---

### Q3 2026 - Alternative Implementations

#### Technology Diversification

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **SQLite Support** | 📋 Planned | Medium | Option to use SQLite instead of MongoDB |
| **Rust Implementation** | 🔬 Research | Low | High-performance Rust rewrite for critical paths |
| **PostgreSQL Support** | 📋 Planned | Low | Native PostgreSQL support |

#### Security Enhancements

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Password Reset Flow** | 🚧 In Progress | High | Secure password reset via email/SMS |
| **Breach Detection** | 📋 Planned | Medium | Check passwords against known breaches |
| **Hardware Security Module** | 📋 Planned | Low | HSM integration for key storage |

---

## 🚀 Requested Features

These features are requested by the community. Vote for features on [GitHub Discussions](https://github.com/ussoio/usso/discussions).

### High Demand

- [ ] GraphQL API (21 votes)
- [ ] WebAuthn biometric auth (18 votes)
- [ ] Social login providers (15 votes)
- [ ] LDAP/Active Directory sync (12 votes)
- [ ] Admin dashboard UI (11 votes)

### Under Consideration

- [ ] SAML 2.0 support
- [ ] Push notification auth
- [ ] Passwordless SMS auth
- [ ] Custom email templates
- [ ] Audit log export

---

## 🎨 UI & Developer Experience

| Feature | Status | Priority | Timeline |
|---------|--------|----------|----------|
| **Admin Dashboard** | 📋 Planned | High | Q1 2026 |
| **User Portal** | 📋 Planned | Medium | Q2 2026 |
| **Developer Console** | 📋 Planned | Medium | Q2 2026 |
| **API Playground** | 📋 Planned | Low | Q3 2026 |
| **Postman Collection** | ✅ Available | - | Now |

---

## 📊 Performance & Scalability

### Current Performance Targets

- ✅ Support 10,000 users per tenant
- ✅ Handle 1,000 requests/second
- ✅ Token verification < 10ms

### Future Targets (Q4 2026)

- 📋 Support 1M+ users per tenant
- 📋 Handle 10,000 requests/second
- 📋 Token verification < 1ms (with Rust)
- 📋 Horizontal scaling support

---

## 🔐 Security Roadmap

| Feature | Status | Timeline |
|---------|--------|----------|
| **SOC 2 Compliance** | 📋 Planned | Q2 2026 |
| **GDPR Tools** | 📋 Planned | Q1 2026 |
| **Penetration Testing** | 🚧 Ongoing | Q4 2025 |
| **Bug Bounty Program** | 📋 Planned | Q3 2026 |
| **Security Audit** | 📋 Planned | Q2 2026 |

---

## 📱 SDK & Client Libraries

### Existing

- ✅ Python SDK ([PyPI](https://pypi.org/project/usso/))
- ✅ JavaScript/TypeScript examples

### Planned

| SDK | Status | Timeline |
|-----|--------|----------|
| **JavaScript/TypeScript SDK** | 📋 Planned | Q1 2026 |
| **Go SDK** | 📋 Planned | Q2 2026 |
| **Java SDK** | 📋 Planned | Q2 2026 |
| **PHP SDK** | 📋 Planned | Q3 2026 |
| **Ruby SDK** | 📋 Planned | Q3 2026 |
| **Swift SDK** | 📋 Planned | Q4 2026 |
| **Kotlin SDK** | 📋 Planned | Q4 2026 |

---

## 🌍 Internationalization

| Feature | Status | Timeline |
|---------|--------|----------|
| **Multi-language Support** | 📋 Planned | Q2 2026 |
| **RTL Support** | 📋 Planned | Q2 2026 |
| **Localized Templates** | 📋 Planned | Q3 2026 |

---

## 🔄 Migration & Import

| Feature | Status | Priority | Timeline |
|---------|--------|----------|----------|
| **Auth0 Import** | 📋 Planned | High | Q1 2026 |
| **Firebase Import** | 📋 Planned | High | Q1 2026 |
| **Keycloak Import** | 📋 Planned | Medium | Q2 2026 |
| **Generic CSV Import** | 📋 Planned | High | Q1 2026 |

---

## 💰 Pricing & Hosting

### Self-Hosted (Always Free)

- ✅ All features
- ✅ Unlimited users
- ✅ Community support

### Managed Service (Coming 2026)

- 📋 Hosted infrastructure
- 📋 Automatic updates
- 📋 Priority support
- 📋 SLA guarantees
- 📋 Enhanced monitoring

---

## 🤝 How to Contribute

We welcome contributions! Here's how you can help:

### 1. Vote on Features

Visit [GitHub Discussions](https://github.com/ussoio/usso/discussions) to vote on features.

### 2. Propose Features

Open a discussion with your feature idea:
- Describe the use case
- Explain the expected behavior
- Provide examples

### 3. Contribute Code

1. Check the [Issues](https://github.com/ussoio/usso/issues) for "good first issue" labels
2. Fork the repository
3. Implement the feature
4. Submit a pull request

### 4. Improve Documentation

- Fix typos
- Add examples
- Translate documentation
- Write tutorials

### 5. Report Bugs

Help us improve by reporting bugs:
- Provide minimal reproduction steps
- Include error logs
- Describe expected vs actual behavior

---

## 📮 Feature Requests

Have a feature request? Here's how to submit it:

1. **Search existing requests** - Check if someone already requested it
2. **Open a discussion** - Use GitHub Discussions
3. **Describe the problem** - What problem does it solve?
4. **Propose a solution** - How should it work?
5. **Provide examples** - Show example use cases

---

## 🗓️ Release Schedule

USSO follows semantic versioning (SemVer):

- **Major releases** (1.0, 2.0) - Breaking changes
- **Minor releases** (0.1, 0.2) - New features
- **Patch releases** (0.1.1, 0.1.2) - Bug fixes

### Release Frequency

- **Patch releases**: Every 2 weeks
- **Minor releases**: Every 1-2 months
- **Major releases**: When API is stable

---

## 📚 Documentation Roadmap

| Topic | Status | Timeline |
|-------|--------|----------|
| **Video Tutorials** | 📋 Planned | Q1 2026 |
| **Architecture Guide** | ✅ Available | Now |
| **Migration Guide** | 📋 Planned | Q1 2026 |
| **API Reference** | ✅ Available | Now |
| **Best Practices** | 🚧 In Progress | Q4 2025 |
| **Case Studies** | 📋 Planned | Q2 2026 |

---

## 🎯 Version 1.0 Goals

Before releasing v1.0, we want to achieve:

- [ ] Stable API (no breaking changes)
- [ ] 90%+ test coverage
- [ ] Complete documentation
- [ ] Production deployments at scale
- [ ] Security audit completion
- [ ] Performance benchmarks met
- [ ] Community-driven development

**Target:** Q3 2026

---

## 📊 Key Metrics

We track these metrics to measure progress:

| Metric | Current | Target (v1.0) |
|--------|---------|---------------|
| Test Coverage | 75% | 90% |
| API Stability | Beta | Stable |
| Documentation Coverage | 60% | 95% |
| Response Time | < 50ms | < 10ms |
| Community Size | 100+ | 1,000+ |

---

## 🌟 Long-Term Vision (2027+)

- Universal authentication standard for startups
- 100,000+ active deployments
- Comprehensive app marketplace
- Enterprise-grade features
- Multi-region support
- Advanced AI-powered security

---

## ❓ Questions?

- **GitHub**: [Open an issue](https://github.com/ussoio/usso/issues)
- **Email**: support@usso.io
- **Discussions**: [Join the conversation](https://github.com/ussoio/usso/discussions)

---

**Last Updated:** October 4, 2025

[← Back to Home](index.md){ .md-button }
[View Contributing Guide →](contributing.md){ .md-button .md-button--primary }

---

## Legend

- ✅ **Available** - Feature is live and ready to use
- 🚧 **In Progress** - Currently being developed
- 📋 **Planned** - Scheduled for future development
- 🔬 **Research** - Exploring feasibility
- ❌ **Deprecated** - No longer supported

