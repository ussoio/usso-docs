# Service Accounts Overview

Service accounts (also called **agents**) are special identities designed for machine-to-machine authentication. Unlike regular user accounts, service accounts don't represent people—they represent applications, scripts, or automated services that need to access your API.

---

## Why Service Accounts?

Imagine you have:

- A **cron job** that needs to sync data every hour
- A **microservice** that needs to call your API
- A **CI/CD pipeline** that deploys your application
- A **third-party integration** that accesses your resources

These scenarios need authentication, but they can't use traditional username/password login. Service accounts solve this problem.

---

## Service Accounts vs Regular Users

| Feature | Regular User | Service Account |
|---------|-------------|-----------------|
| **Purpose** | Human access | Machine access |
| **Authentication** | Password, OAuth, OTP | API keys, JWT tokens |
| **Session** | Browser-based | Stateless |
| **MFA** | Supported | Not applicable |
| **Lifespan** | Long-term | Until revoked |
| **Permissions** | Role + workspace based | Scoped access |

---

## Types of Credentials

USSO provides two types of machine credentials:

### 1. **API Keys** (Simple)

Best for:
- External services
- Simple integrations
- Testing and development

**Pros:**
- Simple to use
- Easy to revoke
- No expiration

**Cons:**
- Stateful (stored in database)
- Less granular control

### 2. **Service Agent Tokens** (Advanced)

Best for:
- Internal microservices
- High-throughput services
- Complex permission requirements

**Pros:**
- Stateless JWT tokens
- Fine-grained scopes
- Full workspace access
- Role-based permissions

**Cons:**
- Slightly more complex setup

---

## Creating a Service Account

### Step 1: Create the Agent

=== "cURL"

    ```bash
    curl -X POST http://localhost:8000/api/sso/v1/agents \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "Data Sync Service",
        "description": "Syncs data with external systems",
        "scopes": [
          "read:users",
          "write:users",
          "read:content"
        ],
        "workspace_id": "ws_engineering"
      }'
    ```

=== "Python"

    ```python
    import requests

    response = requests.post(
        "http://localhost:8000/api/sso/v1/agents",
        headers={
            "Authorization": f"Bearer {admin_token}",
            "Content-Type": "application/json"
        },
        json={
            "name": "Data Sync Service",
            "description": "Syncs data with external systems",
            "scopes": [
                "read:users",
                "write:users",
                "read:content"
            ],
            "workspace_id": "ws_engineering"
        }
    )

    agent = response.json()
    print(f"Agent ID: {agent['id']}")
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('http://localhost:8000/api/sso/v1/agents', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${adminToken}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            name: 'Data Sync Service',
            description: 'Syncs data with external systems',
            scopes: [
                'read:users',
                'write:users',
                'read:content'
            ],
            workspace_id: 'ws_engineering'
        })
    });

    const agent = await response.json();
    console.log('Agent ID:', agent.id);
    ```

**Response:**

```json
{
  "id": "service:abc123",
  "name": "Data Sync Service",
  "description": "Syncs data with external systems",
  "tenant_id": "org_mycompany",
  "workspace_id": "ws_engineering",
  "scopes": [
    "read:users",
    "write:users",
    "read:content"
  ],
  "is_active": true,
  "created_at": "2025-10-04T10:00:00Z"
}
```

### Step 2: Generate API Key

=== "cURL"

    ```bash
    curl -X POST http://localhost:8000/api/sso/v1/apikeys \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "agent_id": "service:abc123",
        "name": "Production Key",
        "expires_in_days": 365
      }'
    ```

=== "Python"

    ```python
    response = requests.post(
        "http://localhost:8000/api/sso/v1/apikeys",
        headers={"Authorization": f"Bearer {admin_token}"},
        json={
            "agent_id": "service:abc123",
            "name": "Production Key",
            "expires_in_days": 365
        }
    )

    api_key_data = response.json()
    api_key = api_key_data["key"]
    
    # IMPORTANT: Save this key securely!
    # It won't be shown again
    print(f"API Key: {api_key}")
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('http://localhost:8000/api/sso/v1/apikeys', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${adminToken}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            agent_id: 'service:abc123',
            name: 'Production Key',
            expires_in_days: 365
        })
    });

    const apiKeyData = await response.json();
    console.log('API Key:', apiKeyData.key);
    // IMPORTANT: Save this key securely!
    ```

**Response:**

```json
{
  "id": "key_xyz789",
  "key": "usso_sk_1a2b3c4d5e6f7g8h9i0j",
  "name": "Production Key",
  "agent_id": "service:abc123",
  "expires_at": "2026-10-04T10:00:00Z",
  "created_at": "2025-10-04T10:00:00Z"
}
```

!!! warning "Save Your API Key"
    **The API key is only shown once!** Save it securely. If you lose it, you'll need to generate a new one.

---

## Using Service Accounts

### With API Keys

Simply include the API key in the `Authorization` header:

=== "cURL"

    ```bash
    curl -X GET http://localhost:8000/api/sso/v1/users \
      -H "Authorization: Bearer usso_sk_1a2b3c4d5e6f7g8h9i0j"
    ```

=== "Python"

    ```python
    import requests

    API_KEY = "usso_sk_1a2b3c4d5e6f7g8h9i0j"

    response = requests.get(
        "http://localhost:8000/api/sso/v1/users",
        headers={"Authorization": f"Bearer {API_KEY}"}
    )

    users = response.json()
    ```

=== "JavaScript"

    ```javascript
    const API_KEY = 'usso_sk_1a2b3c4d5e6f7g8h9i0j';

    const response = await fetch('http://localhost:8000/api/sso/v1/users', {
        headers: {
            'Authorization': `Bearer ${API_KEY}`
        }
    });

    const users = await response.json();
    ```

### With Agent JWT Tokens

For microservices, you can use agent JWT tokens:

```python
from usso.integrations.fastapi import get_authenticator
from usso.config import JWTConfig
from usso.schemas import UserData

# Configure for service accounts
config = JWTConfig(
    jwks_url="http://localhost:8000/.well-known/jwks.json",
    issuer="http://localhost:8000",
    audience="your-service"
)

authenticator = get_authenticator(config)

@app.get("/sync")
async def sync_data(agent: UserData = Depends(authenticator)):
    # Verify it's a service account
    if not agent.sub.startswith("service:"):
        raise HTTPException(status_code=403, detail="Service account required")
    
    # Check scopes
    if "read:users" not in agent.scopes:
        raise HTTPException(status_code=403, detail="Missing required scope")
    
    # Perform sync
    return {"status": "synced"}
```

---

## Managing Service Accounts

### List All Agents

=== "cURL"

    ```bash
    curl -X GET http://localhost:8000/api/sso/v1/agents \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
    ```

=== "Python"

    ```python
    response = requests.get(
        "http://localhost:8000/api/sso/v1/agents",
        headers={"Authorization": f"Bearer {admin_token}"}
    )

    agents = response.json()
    for agent in agents["items"]:
        print(f"{agent['name']}: {agent['id']}")
    ```

### Update Agent Scopes

=== "cURL"

    ```bash
    curl -X PATCH http://localhost:8000/api/sso/v1/agents/service:abc123 \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "scopes": [
          "read:users",
          "write:users",
          "delete:content"
        ]
      }'
    ```

=== "Python"

    ```python
    response = requests.patch(
        f"http://localhost:8000/api/sso/v1/agents/{agent_id}",
        headers={"Authorization": f"Bearer {admin_token}"},
        json={
            "scopes": [
                "read:users",
                "write:users",
                "delete:content"
            ]
        }
    )
    ```

### Deactivate Agent

=== "cURL"

    ```bash
    curl -X PATCH http://localhost:8000/api/sso/v1/agents/service:abc123 \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "is_active": false
      }'
    ```

### Delete Agent

=== "cURL"

    ```bash
    curl -X DELETE http://localhost:8000/api/sso/v1/agents/service:abc123 \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
    ```

---

## API Key Management

### List API Keys

=== "cURL"

    ```bash
    curl -X GET http://localhost:8000/api/sso/v1/apikeys \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
    ```

### Revoke API Key

=== "cURL"

    ```bash
    curl -X DELETE http://localhost:8000/api/sso/v1/apikeys/key_xyz789 \
      -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
    ```

=== "Python"

    ```python
    response = requests.delete(
        f"http://localhost:8000/api/sso/v1/apikeys/{key_id}",
        headers={"Authorization": f"Bearer {admin_token}"}
    )
    ```

!!! tip "Key Rotation"
    Regularly rotate API keys for security. Create a new key, update your services, then revoke the old key.

---

## Real-World Examples

### Example 1: Background Job

```python
# background_job.py
import requests
import os

API_KEY = os.getenv("USSO_API_KEY")
BASE_URL = "http://localhost:8000/api/sso/v1"

def sync_users():
    """Sync users from external system"""
    headers = {"Authorization": f"Bearer {API_KEY}"}
    
    # Fetch users from USSO
    response = requests.get(
        f"{BASE_URL}/users",
        headers=headers
    )
    users = response.json()
    
    # Sync to external system
    for user in users["items"]:
        external_system.sync_user(user)
    
    print(f"Synced {len(users['items'])} users")

if __name__ == "__main__":
    sync_users()
```

### Example 2: Microservice Authentication

```python
# microservice/main.py
from fastapi import FastAPI, Depends, HTTPException
from usso.integrations.fastapi import get_authenticator
from usso.config import JWTConfig
from usso.schemas import UserData

app = FastAPI()

config = JWTConfig(
    jwks_url="http://usso:8000/.well-known/jwks.json",
    issuer="http://usso:8000",
    audience="internal-api"
)

authenticator = get_authenticator(config)

def require_service_account(user: UserData = Depends(authenticator)):
    """Ensure caller is a service account"""
    if not user.sub.startswith("service:"):
        raise HTTPException(
            status_code=403,
            detail="Service account required"
        )
    return user

@app.post("/internal/process")
async def process_data(
    data: dict,
    agent: UserData = Depends(require_service_account)
):
    # Only service accounts can call this
    return {"status": "processed", "agent": agent.sub}
```

### Example 3: CI/CD Integration

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Deploy to production
        env:
          USSO_API_KEY: ${{ secrets.USSO_API_KEY }}
        run: |
          # Use service account to trigger deployment
          curl -X POST https://api.example.com/deploy \
            -H "Authorization: Bearer $USSO_API_KEY" \
            -H "Content-Type: application/json" \
            -d '{"version": "${{ github.sha }}"}'
```

---

## Security Best Practices

### 1. Principle of Least Privilege

Only grant the minimum scopes needed:

```python
# ✅ Good: Specific scopes
scopes = ["read:users", "write:users"]

# ❌ Bad: Too broad
scopes = ["admin:*"]
```

### 2. Store Keys Securely

Never commit API keys to version control:

```bash
# Use environment variables
export USSO_API_KEY="usso_sk_..."

# Or use secret managers
aws secretsmanager get-secret-value --secret-id usso-api-key
```

### 3. Rotate Keys Regularly

Set up a rotation schedule:

```python
# rotation_script.py
def rotate_api_key(agent_id: str):
    # Create new key
    new_key = create_api_key(agent_id, name="Auto-rotated")
    
    # Update services with new key
    update_services(new_key)
    
    # Wait for propagation
    time.sleep(300)  # 5 minutes
    
    # Revoke old key
    revoke_old_keys(agent_id)
```

### 4. Monitor Usage

Track service account activity:

```python
@app.middleware("http")
async def log_service_account_usage(request: Request, call_next):
    if request.headers.get("authorization", "").startswith("Bearer usso_sk_"):
        logger.info(
            "Service account request",
            path=request.url.path,
            method=request.method,
            ip=request.client.host
        )
    response = await call_next(request)
    return response
```

### 5. Set Expiration

Always set expiration for API keys:

```python
# Create key that expires in 90 days
api_key = create_api_key(
    agent_id="service:abc123",
    expires_in_days=90
)
```

---

## Troubleshooting

### API Key Not Working

1. Verify key format: `usso_sk_...`
2. Check if key is expired:
```bash
curl -X GET http://localhost:8000/api/sso/v1/apikeys/key_id \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

3. Verify agent is active:
```bash
curl -X GET http://localhost:8000/api/sso/v1/agents/service:abc123 \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

### Permission Denied

Check agent scopes:

```bash
curl -X GET http://localhost:8000/api/sso/v1/agents/service:abc123 \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

Verify the agent has the required scope for the action.

---

## Next Steps

- **[Creating Agents](creating-agents.md)** - Detailed guide on agent creation
- **[API Keys](api-keys.md)** - Complete API key documentation
- **[Best Practices](best-practices.md)** - Security and operational guidelines
- **[Authorization](../authorization/overview.md)** - Understanding scopes and permissions

---

[← Back to User Management](../user-management/users.md){ .md-button }
[Next: Creating Agents →](creating-agents.md){ .md-button .md-button--primary }

