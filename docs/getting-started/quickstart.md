# Quick Start Guide

Get USSO up and running in under 10 minutes. This guide will walk you through installation, configuration, and your first API call.

---

## Prerequisites

Before you begin, make sure you have:

- **Docker** and **Docker Compose** installed ([Get Docker](https://docs.docker.com/get-docker/))
- Basic understanding of REST APIs
- A code editor

---

## Step 1: Install USSO

Clone the USSO repository:

```bash
git clone https://github.com/ussoio/usso.git
cd usso
```

---

## Step 2: Configure Environment

Copy the sample environment file and edit it:

```bash
cp sample.env .env
```

Edit `.env` with your favorite editor. At minimum, set these variables:

```bash
# Project basics
PROJECT_NAME=my-sso
DOMAIN=localhost

# Database
MONGO_URI=mongodb://localhost:27017/
REDIS_URI=redis://localhost:6379/

# Security (generate a strong password)
SYSTEM_PASSWORD=your-super-secret-password

# Optional: Email for magic links and OTP
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_SENDER=noreply@example.com
SMTP_USE_TLS=true
```

!!! tip "Generate Secure Passwords"
    Use a password manager or run: `openssl rand -base64 32`

---

## Step 3: Start USSO

Launch all services with Docker Compose:

```bash
docker compose up -d
```

This will start:

- **USSO API** on port 8000
- **MongoDB** for data storage
- **Redis** for sessions and caching

Check if services are running:

```bash
docker compose ps
```

---

## Step 4: Verify Installation

Open your browser and visit:

- **API Docs**: [http://localhost:8000/api/sso/v1/docs](http://localhost:8000/api/sso/v1/docs)
- **Alternative Docs**: [http://localhost:8000/api/sso/v1/redoc](http://localhost:8000/api/sso/v1/redoc)

You should see the FastAPI Swagger interface.

---

## Step 5: Create Your First User

Now let's create a user via the REST API:

=== "cURL"

```bash
curl -X POST http://localhost:8000/api/sso/v1/auth/login \
    -H "Content-Type: application/json" \
    -d '{
    "identifier": "admin@example.com",
    "secret": "SecurePassword123!",
    "register": true
    }'
```

=== "Python"

```python
import requests

response = requests.post(
    "http://localhost:8000/api/sso/v1/auth/login",
    json={
        "identifier": "admin@example.com",
        "secret": "SecurePassword123!",
        "register": true
    }
)

data = response.json()
access_token = data["access_token"]
print(f"Access token: {access_token}")
```

=== "JavaScript"

```javascript
const response = await fetch('http://localhost:8000/api/sso/v1/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        identifier: 'admin@example.com',
        secret: 'SecurePassword123!',
        register: true
    })
});

const data = await response.json();
console.log('Access token:', data.access_token);
```

The response will include:

```json
{
  "access_token": "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user": {
    "id": "user:abc123",
    "tenant_id": "org_default",
    "identifiers": [
      {
        "type": "email",
        "identifier": "admin@example.com",
        "verified": false
      }
    ]
  }
}
```

!!! success "Congratulations!"
    You've created your first user! The `access_token` can be used to authenticate API requests.

---

## Step 6: Access Protected Resources

Use the access token to call protected endpoints:

=== "cURL"

    ```bash
    # Get current user info
    curl -X GET http://localhost:8000/api/sso/v1/me \
      -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
    ```

=== "Python"

    ```python
    headers = {"Authorization": f"Bearer {access_token}"}
    response = requests.get(
        "http://localhost:8000/api/sso/v1/me",
        headers=headers
    )
    print(response.json())
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('http://localhost:8000/api/sso/v1/me', {
        headers: {
            'Authorization': `Bearer ${accessToken}`
        }
    });

    const user = await response.json();
    console.log('Current user:', user);
    ```

---

## Step 7: Integrate with Your App

Now that USSO is running, integrate it with your application using our SDKs:

### Option A: Python SDK

Install the USSO Python SDK:

```bash
pip install usso
```

Use it in your FastAPI app:

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

[Read full Python SDK guide →](../integration/python-sdk.md)

### Option B: JavaScript SDK

For Node.js or frontend apps, use direct API calls:

```javascript
// Verify token manually
async function verifyToken(token) {
    const response = await fetch('http://localhost:8000/api/sso/v1/auth/verify-token', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ token })
    });
    
    return await response.json();
}
```

[Read full JavaScript guide →](../integration/javascript-sdk.md)

---

## Common Operations

### Login (Existing User)

```bash
curl -X POST http://localhost:8000/api/sso/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "identifier": "admin@example.com",
    "secret": "SecurePassword123!"
  }'
```

### Refresh Token

```bash
curl -X POST http://localhost:8000/api/sso/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "YOUR_REFRESH_TOKEN"
  }'
```

### Logout

```bash
curl -X POST http://localhost:8000/api/sso/v1/auth/logout \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

---

## Next Steps

Now that USSO is running, explore these topics:

- **[Core Concepts](../concepts/architecture.md)** - Understand how USSO works
- **[Authentication Methods](../authentication/login-methods.md)** - Setup OAuth, magic links, and more
- **[Authorization](../authorization/overview.md)** - Configure roles and permissions
- **[Service Accounts](../service-accounts/overview.md)** - Create API keys for services
- **[Production Deployment](../deployment/production.md)** - Deploy to production

---

## Troubleshooting

### Port Already in Use

If port 8000 is already in use, edit `docker-compose.yml` to change the port:

```yaml
services:
  app:
    ports:
      - "8080:8000"  # Change host port to 8080
```

### Cannot Connect to MongoDB

Make sure MongoDB is running:

```bash
docker compose logs mongo
```

If needed, restart services:

```bash
docker compose restart
```

### Token Verification Fails

Ensure your JWKS URL is accessible:

```bash
curl http://localhost:8000/.well-known/jwks.json
```

---

## Getting Help

- **Documentation**: Browse this site for detailed guides
- **GitHub Issues**: [Report bugs or ask questions](https://github.com/ussoio/usso/issues)
- **Email**: support@usso.io

---

[← Back to Home](../index.md){ .md-button }
[Next: Core Concepts →](../concepts/architecture.md){ .md-button .md-button--primary }

