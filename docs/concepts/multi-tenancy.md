# Multi-Tenancy

Multi-tenancy is a core feature of USSO that allows you to serve multiple organizations (tenants) from a single USSO instance, with complete data isolation and per-tenant configuration.

---

## What is Multi-Tenancy?

Think of USSO as an apartment building:
- The **building** is your USSO instance
- Each **apartment** is a tenant (organization)
- **Residents** in one apartment can't access another apartment
- Each apartment can have different **decorations** (configuration)

### Why Multi-Tenancy?

**For SaaS Applications:**
- Serve multiple companies from one deployment
- Reduce infrastructure costs
- Easier maintenance and updates
- Faster onboarding of new customers

**For Enterprise:**
- Separate departments or business units
- Isolate test/staging environments
- Manage multiple brands

---

## Tenant Isolation

USSO provides **complete isolation** between tenants:

### 1. **Data Isolation**

Every record includes a `tenant_id`:

```python
{
  "id": "user:abc123",
  "tenant_id": "org_company_a",  # ← Isolates this user
  "email": "user@company-a.com",
  "roles": ["editor"]
}
```

All queries automatically filter by tenant:

```python
# Fetch users - automatically scoped to tenant
users = await User.find(
    User.tenant_id == current_tenant.id
).to_list()
```

### 2. **Configuration Isolation**

Each tenant has its own settings:

```python
{
  "id": "org_company_a",
  "name": "Company A",
  "domains": ["company-a.com", "app.company-a.com"],
  "config": {
    "require_mfa": true,
    "session_timeout": 3600,
    "allowed_login_methods": ["password", "oauth"]
  },
  "branding": {
    "logo_url": "https://...",
    "primary_color": "#FF6B6B"
  }
}
```

### 3. **Key Isolation**

Each tenant has its own JWT signing keys:

```python
{
  "tenant_id": "org_company_a",
  "keys": [
    {
      "id": "key_abc123",
      "algorithm": "EdDSA",
      "public_key": "...",
      "private_key": "...",  # Encrypted
      "is_active": true
    }
  ]
}
```

**Benefits:**
- Tokens from one tenant can't access another
- Per-tenant key rotation
- Different security policies per tenant

---

## Tenant Identification

USSO identifies tenants in two ways:

### Option 1: Domain-Based (Recommended)

The tenant is determined by the domain in the request:

```bash
# Request to company-a.com
curl https://company-a.com/api/sso/v1/me
# → Resolves to org_company_a

# Request to company-b.com
curl https://company-b.com/api/sso/v1/me
# → Resolves to org_company_b
```

**Setup:**

1. Register domains with tenant:

```bash
curl -X POST http://localhost:8000/api/sso/v1/tenants \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Company A",
    "domains": ["company-a.com", "app.company-a.com"]
  }'
```

2. Configure DNS to point to your USSO instance

### Option 2: Header-Based

Pass tenant ID in a header:

```bash
curl -X GET http://localhost:8000/api/sso/v1/me \
  -H "X-Tenant-ID: org_company_a" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Use cases:**
- Single domain for all tenants
- Testing and development
- Mobile apps

---

## Creating Tenants

### Via API

=== "cURL"

    ```bash
    curl -X POST http://localhost:8000/api/sso/v1/tenants \
      -H "Authorization: Bearer SYSTEM_ADMIN_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "My Company",
        "slug": "my-company",
        "domains": ["mycompany.com"],
        "config": {
          "require_mfa": false,
          "session_timeout": 7200
        }
      }'
    ```

=== "Python"

    ```python
    import requests

    response = requests.post(
        "http://localhost:8000/api/sso/v1/tenants",
        headers={
            "Authorization": f"Bearer {admin_token}",
            "Content-Type": "application/json"
        },
        json={
            "name": "My Company",
            "slug": "my-company",
            "domains": ["mycompany.com"],
            "config": {
                "require_mfa": False,
                "session_timeout": 7200
            }
        }
    )

    tenant = response.json()
    print(f"Tenant ID: {tenant['id']}")
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('http://localhost:8000/api/sso/v1/tenants', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${adminToken}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            name: 'My Company',
            slug: 'my-company',
            domains: ['mycompany.com'],
            config: {
                require_mfa: false,
                session_timeout: 7200
            }
        })
    });

    const tenant = await response.json();
    console.log('Tenant ID:', tenant.id);
    ```

### Default Tenant

On first startup, USSO creates a default tenant:

```python
{
  "id": "org_default",
  "name": "Default Tenant",
  "domains": ["localhost", "127.0.0.1", "dev.usso.io"],
  "is_active": true
}
```

---

## Tenant Configuration

### Authentication Settings

```python
{
  "config": {
    # Login methods
    "allowed_login_methods": [
      "password",
      "magic_link",
      "oauth",
      "passkey"
    ],
    
    # Security
    "require_mfa": false,
    "password_min_length": 8,
    "password_require_special": true,
    
    # Sessions
    "session_timeout": 3600,  # 1 hour
    "refresh_token_ttl": 2592000,  # 30 days
    
    # Rate limiting
    "login_attempts_limit": 5,
    "login_attempts_window": 300  # 5 minutes
  }
}
```

### Messaging Configuration

Each tenant can have its own email/SMS providers:

```python
{
  "messaging": {
    "email": {
      "provider": "smtp",
      "smtp_host": "smtp.company-a.com",
      "smtp_port": 587,
      "smtp_username": "noreply@company-a.com",
      "from_address": "Company A <noreply@company-a.com>"
    },
    "sms": {
      "provider": "twilio",
      "account_sid": "...",
      "auth_token": "..."
    }
  }
}
```

### Branding

Customize the look and feel:

```python
{
  "branding": {
    "logo_url": "https://cdn.company-a.com/logo.png",
    "primary_color": "#FF6B6B",
    "secondary_color": "#4ECDC4",
    "company_name": "Company A Inc.",
    "support_email": "support@company-a.com"
  }
}
```

---

## Workspaces Within Tenants

Tenants can have multiple **workspaces** for further organization:

```
Tenant: Company A (org_company_a)
├── Workspace: Engineering (ws_engineering)
│   ├── User: alice@company-a.com (Developer)
│   └── User: bob@company-a.com (Team Lead)
├── Workspace: Marketing (ws_marketing)
│   ├── User: carol@company-a.com (Manager)
│   └── User: dave@company-a.com (Designer)
└── Workspace: Sales (ws_sales)
    └── User: eve@company-a.com (Sales Rep)
```

**Benefits:**
- Further data segmentation within a tenant
- Different permissions per workspace
- Users can belong to multiple workspaces

---

## Access Patterns

### Pattern 1: Shared SaaS

One USSO instance serving multiple companies:

```
USSO Instance
├── Tenant: Company A
│   ├── 1,000 users
│   └── 5 workspaces
├── Tenant: Company B
│   ├── 500 users
│   └── 3 workspaces
└── Tenant: Company C
    ├── 2,000 users
    └── 10 workspaces
```

**Routing:**
- `company-a.com` → Tenant A
- `company-b.com` → Tenant B
- `company-c.com` → Tenant C

### Pattern 2: Enterprise Departments

One company, multiple departments:

```
USSO Instance
└── Tenant: MegaCorp (org_megacorp)
    ├── Workspace: Engineering
    ├── Workspace: HR
    ├── Workspace: Finance
    └── Workspace: Operations
```

### Pattern 3: Multi-Brand

One company, multiple brands:

```
USSO Instance
├── Tenant: Brand A (org_brand_a)
├── Tenant: Brand B (org_brand_b)
└── Tenant: Brand C (org_brand_c)
```

---

## Implementation Example

### Tenant Middleware

```python
from fastapi import Request, HTTPException
from typing import Optional

class TenantMiddleware:
    async def __call__(self, request: Request, call_next):
        # Resolve tenant
        tenant = await self.resolve_tenant(request)
        
        if not tenant:
            raise HTTPException(status_code=400, detail="Tenant not found")
        
        # Store in request state
        request.state.tenant = tenant
        
        # Process request
        response = await call_next(request)
        return response
    
    async def resolve_tenant(self, request: Request) -> Optional[Tenant]:
        # Try header first
        tenant_id = request.headers.get("X-Tenant-ID")
        if tenant_id:
            return await Tenant.get(tenant_id)
        
        # Try domain
        host = request.headers.get("host", "").split(":")[0]
        return await Tenant.get_from_domain(host)
```

### Tenant-Scoped Queries

```python
from fastapi import Depends, Request

def get_current_tenant(request: Request) -> Tenant:
    """Get tenant from request state"""
    return request.state.tenant

@router.get("/users")
async def list_users(
    tenant: Tenant = Depends(get_current_tenant)
):
    # Automatically scoped to tenant
    users = await User.find(
        User.tenant_id == tenant.id
    ).to_list()
    
    return {"users": users}
```

### Cross-Tenant Operations (Admin Only)

```python
@router.get("/admin/tenants/{tenant_id}/users")
async def list_tenant_users(
    tenant_id: str,
    admin: UserData = Depends(require_system_admin)
):
    """System admin can query any tenant"""
    users = await User.find(
        User.tenant_id == tenant_id
    ).to_list()
    
    return {"users": users}
```

---

## Security Considerations

### 1. **Prevent Tenant Leakage**

Always validate tenant ownership:

```python
@router.get("/workspaces/{workspace_id}")
async def get_workspace(
    workspace_id: str,
    tenant: Tenant = Depends(get_current_tenant)
):
    workspace = await Workspace.get(workspace_id)
    
    # Verify workspace belongs to tenant
    if workspace.tenant_id != tenant.id:
        raise HTTPException(status_code=404, detail="Workspace not found")
    
    return workspace
```

### 2. **Token Validation**

Verify token was issued by the correct tenant:

```python
def verify_token(token: str, tenant: Tenant) -> UserData:
    # Decode and verify with tenant's public key
    claims = tenant.verify_jwt(token)
    
    # Ensure tenant_id matches
    if claims.get("tenant_id") != tenant.id:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return UserData(**claims)
```

### 3. **Database Indexes**

Ensure efficient tenant-scoped queries:

```python
class User(BaseDocument):
    tenant_id: str
    email: str
    
    class Settings:
        indexes = [
            # Compound index for tenant + email
            IndexModel(
                [("tenant_id", ASCENDING), ("email", ASCENDING)],
                unique=True
            )
        ]
```

---

## Migration Between Tenants

Moving users between tenants (rare but possible):

```python
async def migrate_user_to_tenant(
    user_id: str,
    target_tenant_id: str
):
    """Migrate user to different tenant"""
    user = await User.get(user_id)
    
    # Update tenant_id
    user.tenant_id = target_tenant_id
    
    # Clear tenant-specific data
    user.roles = []
    user.workspaces = []
    
    # Save
    await user.save()
    
    # Revoke all sessions
    await AuthSession.find(
        AuthSession.user_id == user_id
    ).delete()
    
    return user
```

---

## Monitoring Per Tenant

Track metrics per tenant:

```python
@router.post("/auth/login")
async def login(
    data: LoginRequest,
    tenant: Tenant = Depends(get_current_tenant)
):
    # Track login attempt
    metrics.increment("login.attempt", tags={"tenant": tenant.id})
    
    try:
        user = await authenticate_user(data, tenant)
        metrics.increment("login.success", tags={"tenant": tenant.id})
        return user
    except AuthenticationError:
        metrics.increment("login.failure", tags={"tenant": tenant.id})
        raise
```

---

## Best Practices

### 1. **Use Domain-Based Routing**

More intuitive for users:
```
company-a.com → Tenant A
company-b.com → Tenant B
```

### 2. **Separate Branding**

Give each tenant its own look:
```python
tenant.branding = {
    "logo": "...",
    "colors": {...}
}
```

### 3. **Tenant-Level Configuration**

Allow tenants to customize behavior:
```python
tenant.config = {
    "require_mfa": True,
    "session_timeout": 7200
}
```

### 4. **Monitor Resource Usage**

Track per-tenant usage:
- Number of users
- API requests
- Storage used
- Active sessions

### 5. **Tenant Lifecycle**

Support full tenant lifecycle:
- **Onboarding** - Easy tenant creation
- **Configuration** - Self-service settings
- **Monitoring** - Usage dashboards
- **Offboarding** - Data export and deletion

---

## Troubleshooting

### Tenant Not Found

```python
# Check tenant exists
tenant = await Tenant.get_from_domain("company-a.com")
if not tenant:
    print("Tenant not found for domain")

# List all tenants
tenants = await Tenant.find().to_list()
for tenant in tenants:
    print(f"{tenant.name}: {tenant.domains}")
```

### Cross-Tenant Data Leakage

```python
# Always filter by tenant
users = await User.find(
    User.tenant_id == current_tenant.id  # ← Don't forget this!
).to_list()
```

### Token Validation Fails

```python
# Ensure token was issued for correct tenant
claims = decode_token(token)
if claims["tenant_id"] != current_tenant.id:
    raise ValueError("Token tenant mismatch")
```

---

## Next Steps

- **[Authentication vs Authorization](auth-vs-authz.md)** - Understanding access control
- **[Workspaces & Roles](workspaces-roles.md)** - Organizing users
- **[Tokens & Sessions](tokens-sessions.md)** - Token management

---

[← Back to Architecture](architecture.md){ .md-button }
[Next: Auth vs Authz →](auth-vs-authz.md){ .md-button .md-button--primary }

