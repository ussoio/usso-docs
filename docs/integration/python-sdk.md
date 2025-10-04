# Python SDK Integration

The [USSO Python SDK](https://pypi.org/project/usso/) provides a plug-and-play authentication solution for Python applications. It supports FastAPI, Django, and direct JWT verification.

---

## Installation

Install the USSO SDK via pip:

```bash
pip install usso
```

For specific framework support:

```bash
# For FastAPI applications
pip install "usso[fastapi]"

# For Django applications
pip install "usso[django]"

# Install all extras
pip install "usso[all]"
```

---

## Quick Start

### Basic Token Verification

```python
from usso import USSOAuth
from usso.config import JWTConfig

# Configure USSO
config = JWTConfig(
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    issuer="http://localhost:8000",
    audience="your-app"
)

auth = USSOAuth(config)

# Verify a token
try:
    user_data = auth.verify_token(access_token)
    print(f"User ID: {user_data.sub}")
    print(f"Roles: {user_data.roles}")
    print(f"Scopes: {user_data.scopes}")
except Exception as e:
    print(f"Token verification failed: {e}")
```

---

## FastAPI Integration

The SDK provides seamless FastAPI integration with dependency injection:

### Basic Setup

```python
from fastapi import FastAPI, Depends
from usso.integrations.fastapi import get_authenticator
from usso.config import JWTConfig
from usso.schemas import UserData

# Configure USSO
config = JWTConfig(
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    issuer="http://localhost:8000",
    audience="your-app"
)

# Create authenticator dependency
authenticator = get_authenticator(config)

app = FastAPI()

@app.get("/protected")
def protected_route(user: UserData = Depends(authenticator)):
    return {
        "message": "Access granted",
        "user_id": user.sub,
        "roles": user.roles,
        "scopes": user.scopes
    }
```

### Multiple Token Sources

The SDK automatically checks multiple sources for tokens:

1. **Authorization header**: `Authorization: Bearer <token>`
2. **Custom headers**: Configurable header name
3. **Cookies**: HTTP-only secure cookies

```python
from usso.config import JWTConfig, JWTHeaderConfig

config = JWTConfig(
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    issuer="http://localhost:8000",
    audience="your-app",
    # Configure token sources
    header=JWTHeaderConfig(
        type="Authorization",  # Header name
        cookie_name="access_token"  # Cookie name
    )
)
```

### Scope-Based Authorization

```python
from fastapi import FastAPI, Depends, HTTPException, status
from usso.integrations.fastapi import get_authenticator
from usso.schemas import UserData

authenticator = get_authenticator(config)
app = FastAPI()

def require_scope(required_scope: str):
    """Dependency factory for scope checking"""
    def scope_checker(user: UserData = Depends(authenticator)):
        if required_scope not in user.scopes:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Required scope '{required_scope}' not found"
            )
        return user
    return scope_checker

# Only users with 'write:posts' scope can access
@app.post("/posts")
def create_post(
    post_data: dict,
    user: UserData = Depends(require_scope("write:posts"))
):
    return {"message": "Post created", "author": user.sub}

# Only users with 'delete:posts' scope can access
@app.delete("/posts/{post_id}")
def delete_post(
    post_id: str,
    user: UserData = Depends(require_scope("delete:posts"))
):
    return {"message": f"Post {post_id} deleted"}
```

### Role-Based Authorization

```python
def require_role(required_role: str):
    """Dependency factory for role checking"""
    def role_checker(user: UserData = Depends(authenticator)):
        if required_role not in user.roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Required role '{required_role}' not found"
            )
        return user
    return role_checker

# Only admins can access
@app.get("/admin/users")
def list_all_users(admin: UserData = Depends(require_role("admin"))):
    # Return all users
    return {"users": [...]}

# Editors and admins can access
def require_any_role(*roles):
    def role_checker(user: UserData = Depends(authenticator)):
        if not any(role in user.roles for role in roles):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"One of roles {roles} required"
            )
        return user
    return role_checker

@app.post("/content")
def create_content(
    content: dict,
    user: UserData = Depends(require_any_role("editor", "admin"))
):
    return {"message": "Content created"}
```

### Workspace Isolation

```python
from fastapi import FastAPI, Depends
from usso.schemas import UserData
from sqlalchemy import select

@app.get("/projects")
def list_projects(
    user: UserData = Depends(authenticator),
    db: Session = Depends(get_db)
):
    # Filter by user's workspace
    projects = db.execute(
        select(Project).where(Project.workspace_id == user.workspace_id)
    ).scalars().all()
    
    return {"projects": projects}

@app.post("/projects")
def create_project(
    project_data: dict,
    user: UserData = Depends(authenticator),
    db: Session = Depends(get_db)
):
    # Automatically assign to user's workspace
    project = Project(
        **project_data,
        workspace_id=user.workspace_id,
        created_by=user.sub
    )
    db.add(project)
    db.commit()
    
    return {"project": project}
```

---

## Configuration Options

### JWTConfig

Complete configuration options:

```python
from usso.config import JWTConfig, JWTHeaderConfig
from usso.jwt.enums import Algorithm

config = JWTConfig(
    # Required: JWT validation
    issuer="http://localhost:8000",
    audience="your-app",
    
    # Option 1: Use JWKS URL (recommended)
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    
    # Option 2: Provide public key directly
    # key="-----BEGIN PUBLIC KEY-----\n...",
    # type=Algorithm.EdDSA,
    
    # Token extraction
    header=JWTHeaderConfig(
        type="Authorization",  # Header name
        prefix="Bearer",  # Token prefix
        cookie_name="access_token"  # Cookie name (optional)
    ),
    
    # Validation options
    verify_exp=True,  # Verify expiration
    verify_nbf=True,  # Verify not-before
    verify_aud=True,  # Verify audience
    verify_iss=True,  # Verify issuer
    
    # Clock skew tolerance (seconds)
    leeway=10
)
```

### Algorithm Support

USSO supports multiple JWT algorithms:

| Algorithm | Description | Use Case |
|-----------|-------------|----------|
| **EdDSA** | Ed25519 (default) | Recommended for performance |
| **RS256** | RSA 2048-bit | Legacy compatibility |
| **RS384** | RSA 3072-bit | Higher security |
| **RS512** | RSA 4096-bit | Maximum security |
| **ES256** | ECDSA P-256 | Mobile-friendly |
| **ES384** | ECDSA P-384 | Higher security |
| **HS256** | HMAC SHA-256 | Symmetric (not recommended) |

```python
from usso.jwt.enums import Algorithm

# For EdDSA (recommended)
config = JWTConfig(
    key="your-ed25519-public-key",
    type=Algorithm.EdDSA,
    issuer="...",
    audience="..."
)

# For RS256
config = JWTConfig(
    key="-----BEGIN PUBLIC KEY-----\n...",
    type=Algorithm.RS256,
    issuer="...",
    audience="..."
)
```

---

## Django Integration

### Middleware Setup

```python
# settings.py
MIDDLEWARE = [
    # ... other middleware
    'usso.integrations.django.middleware.USSOAuthMiddleware',
]

# USSO Configuration
USSO_CONFIG = {
    'JWKS_URL': 'http://localhost:8000/.well-known/jwks.json',
    'ISSUER': 'http://localhost:8000',
    'AUDIENCE': 'your-app',
    'VERIFY_EXP': True,
    'VERIFY_AUD': True,
}
```

### Using in Views

```python
from django.http import JsonResponse
from usso.integrations.django.decorators import require_auth, require_scope

@require_auth
def protected_view(request):
    user = request.usso_user
    return JsonResponse({
        'user_id': user.sub,
        'roles': user.roles
    })

@require_scope('write:posts')
def create_post(request):
    user = request.usso_user
    # Create post
    return JsonResponse({'message': 'Post created'})
```

### Class-Based Views

```python
from django.views import View
from django.http import JsonResponse
from usso.integrations.django.mixins import USSOAuthMixin

class ProtectedView(USSOAuthMixin, View):
    required_scopes = ['read:data']
    
    def get(self, request):
        user = request.usso_user
        return JsonResponse({
            'data': 'Protected data',
            'user': user.sub
        })
```

---

## Advanced Usage

### Custom User Data

Extend the UserData schema for custom claims:

```python
from usso.schemas import UserData as BaseUserData
from pydantic import Field

class CustomUserData(BaseUserData):
    """Extended user data with custom claims"""
    company_id: str | None = Field(None, description="Company ID")
    subscription_tier: str | None = Field(None, description="Subscription level")
    custom_permissions: list[str] = Field(default_factory=list)

# Use custom schema
auth = USSOAuth(config, user_schema=CustomUserData)

# Verify token
user = auth.verify_token(token)
print(f"Company: {user.company_id}")
print(f"Tier: {user.subscription_tier}")
```

### Token Caching

For high-traffic applications, cache JWKS:

```python
from usso.config import JWTConfig
from cachetools import TTLCache

# Cache JWKS for 1 hour
jwks_cache = TTLCache(maxsize=100, ttl=3600)

config = JWTConfig(
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    issuer="http://localhost:8000",
    audience="your-app",
    jwks_cache=jwks_cache
)
```

### Multiple Tenants

Support multiple tenants with different configurations:

```python
from fastapi import FastAPI, Depends, Request
from usso.integrations.fastapi import get_authenticator
from usso.config import JWTConfig

app = FastAPI()

# Tenant configurations
TENANT_CONFIGS = {
    "tenant1": JWTConfig(
        jwks_url="http://tenant1.usso.io/.well-known/jwks.json",
        issuer="http://tenant1.usso.io",
        audience="app1"
    ),
    "tenant2": JWTConfig(
        jwks_url="http://tenant2.usso.io/.well-known/jwks.json",
        issuer="http://tenant2.usso.io",
        audience="app2"
    ),
}

def get_tenant_authenticator(request: Request):
    """Get authenticator based on tenant"""
    # Extract tenant from subdomain or header
    tenant_id = request.headers.get("X-Tenant-ID")
    config = TENANT_CONFIGS.get(tenant_id)
    
    if not config:
        raise HTTPException(status_code=400, detail="Invalid tenant")
    
    return get_authenticator(config)

@app.get("/data")
def get_data(user: UserData = Depends(get_tenant_authenticator)):
    return {"tenant": user.tenant_id, "data": [...]}
```

### Async Token Verification

For async applications:

```python
import asyncio
from usso import USSOAuth

auth = USSOAuth(config)

async def verify_async(token: str):
    """Async token verification"""
    # Run verification in thread pool
    loop = asyncio.get_event_loop()
    user = await loop.run_in_executor(None, auth.verify_token, token)
    return user

# Usage in FastAPI
@app.get("/async-protected")
async def async_protected():
    token = request.headers.get("authorization", "").split(" ")[1]
    user = await verify_async(token)
    return {"user": user.sub}
```

---

## Error Handling

The SDK raises specific exceptions:

```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from usso.exceptions import (
    USSOException,
    TokenExpired,
    TokenInvalid,
    TokenNotFound,
    InsufficientScope
)

app = FastAPI()

@app.exception_handler(TokenExpired)
async def token_expired_handler(request: Request, exc: TokenExpired):
    return JSONResponse(
        status_code=status.HTTP_401_UNAUTHORIZED,
        content={"error": "Token expired", "detail": str(exc)}
    )

@app.exception_handler(TokenInvalid)
async def token_invalid_handler(request: Request, exc: TokenInvalid):
    return JSONResponse(
        status_code=status.HTTP_401_UNAUTHORIZED,
        content={"error": "Invalid token", "detail": str(exc)}
    )

@app.exception_handler(InsufficientScope)
async def insufficient_scope_handler(request: Request, exc: InsufficientScope):
    return JSONResponse(
        status_code=status.HTTP_403_FORBIDDEN,
        content={"error": "Insufficient permissions", "detail": str(exc)}
    )

@app.exception_handler(USSOException)
async def usso_exception_handler(request: Request, exc: USSOException):
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={"error": "Authentication error", "detail": str(exc)}
    )
```

---

## Testing

### Mocking Authentication

```python
import pytest
from fastapi.testclient import TestClient
from usso.schemas import UserData

@pytest.fixture
def mock_user():
    """Create mock user for testing"""
    return UserData(
        sub="user:test123",
        tenant_id="org_test",
        workspace_id="ws_test",
        roles=["editor"],
        scopes=["read:posts", "write:posts"],
        exp=9999999999  # Far future
    )

@pytest.fixture
def authenticated_client(app, mock_user):
    """Create test client with mocked auth"""
    def mock_authenticator():
        return mock_user
    
    app.dependency_overrides[authenticator] = mock_authenticator
    client = TestClient(app)
    yield client
    app.dependency_overrides.clear()

def test_protected_endpoint(authenticated_client):
    """Test protected endpoint with mocked user"""
    response = authenticated_client.get("/protected")
    assert response.status_code == 200
    assert response.json()["user_id"] == "user:test123"
```

### Integration Testing

```python
import pytest
from usso import USSOAuth
from usso.config import JWTConfig

@pytest.fixture
def usso_auth():
    """Create USSO auth instance"""
    config = JWTConfig(
        jwks_url="http://localhost:8000/.well-known/jwks.json",
        issuer="http://localhost:8000",
        audience="test-app"
    )
    return USSOAuth(config)

def test_token_verification(usso_auth):
    """Test actual token verification"""
    # Get real token from USSO
    token = login_and_get_token()
    
    # Verify token
    user = usso_auth.verify_token(token)
    
    assert user.sub.startswith("user:")
    assert len(user.roles) > 0
```

---

## Best Practices

### 1. Use Environment Variables

```python
import os
from usso.config import JWTConfig

config = JWTConfig(
    jwks_url=os.getenv("USSO_JWKS_URL"),
    issuer=os.getenv("USSO_ISSUER"),
    audience=os.getenv("USSO_AUDIENCE")
)
```

### 2. Cache JWKS

```python
from cachetools import TTLCache

jwks_cache = TTLCache(maxsize=100, ttl=3600)
config = JWTConfig(..., jwks_cache=jwks_cache)
```

### 3. Validate Scopes Early

```python
@app.post("/resource")
def create_resource(
    data: dict,
    user: UserData = Depends(require_scope("write:resource"))
):
    # Scope already validated by dependency
    pass
```

### 4. Log Authentication Events

```python
import logging

logger = logging.getLogger(__name__)

def authenticator_with_logging(user: UserData = Depends(authenticator)):
    logger.info(
        "User authenticated",
        user_id=user.sub,
        tenant=user.tenant_id,
        workspace=user.workspace_id
    )
    return user
```

---

## Troubleshooting

### JWKS URL Not Accessible

```python
# Verify JWKS URL is accessible
import requests

jwks_url = "http://localhost:8000/.well-known/jwks.json"
response = requests.get(jwks_url)
print(response.json())
```

### Token Validation Fails

```python
# Debug token validation
from usso import USSOAuth

auth = USSOAuth(config)

try:
    user = auth.verify_token(token)
except Exception as e:
    print(f"Validation error: {type(e).__name__}: {e}")
    
    # Check token claims
    import jwt
    claims = jwt.decode(token, options={"verify_signature": False})
    print(f"Token claims: {claims}")
```

---

## Complete Example

Here's a complete example application:

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from usso.integrations.fastapi import get_authenticator
from usso.config import JWTConfig
from usso.schemas import UserData
from pydantic import BaseModel
import os

# Configuration
config = JWTConfig(
    jwks_url=os.getenv("USSO_JWKS_URL"),
    issuer=os.getenv("USSO_ISSUER"),
    audience=os.getenv("USSO_AUDIENCE")
)

authenticator = get_authenticator(config)
security = HTTPBearer()

app = FastAPI(title="My API with USSO")

# Models
class Post(BaseModel):
    title: str
    content: str

# Helper functions
def require_scope(scope: str):
    def checker(user: UserData = Depends(authenticator)):
        if scope not in user.scopes:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Scope '{scope}' required"
            )
        return user
    return checker

# Routes
@app.get("/")
def read_root():
    return {"message": "Welcome to My API"}

@app.get("/me")
def get_current_user(user: UserData = Depends(authenticator)):
    return {
        "user_id": user.sub,
        "tenant": user.tenant_id,
        "workspace": user.workspace_id,
        "roles": user.roles,
        "scopes": user.scopes
    }

@app.get("/posts")
def list_posts(user: UserData = Depends(require_scope("read:posts"))):
    # Filter by workspace
    posts = db.get_posts(workspace_id=user.workspace_id)
    return {"posts": posts}

@app.post("/posts", status_code=status.HTTP_201_CREATED)
def create_post(
    post: Post,
    user: UserData = Depends(require_scope("write:posts"))
):
    # Create post in user's workspace
    new_post = db.create_post(
        **post.dict(),
        workspace_id=user.workspace_id,
        author_id=user.sub
    )
    return {"post": new_post}

@app.delete("/posts/{post_id}")
def delete_post(
    post_id: str,
    user: UserData = Depends(require_scope("delete:posts"))
):
    db.delete_post(post_id, workspace_id=user.workspace_id)
    return {"message": "Post deleted"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8001)
```

---

## Next Steps

- **[JavaScript SDK](javascript-sdk.md)** - Frontend integration
- **[REST API](rest-api.md)** - Direct API usage
- **[Authorization](../authorization/overview.md)** - Permission management

---

[← Back to OAuth Provider](../oauth-provider/overview.md){ .md-button }
[Next: JavaScript SDK →](javascript-sdk.md){ .md-button .md-button--primary }

