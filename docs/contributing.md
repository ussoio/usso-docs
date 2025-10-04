# Contributing to USSO

Thank you for your interest in contributing to USSO! This guide will help you get started.

---

## Ways to Contribute

You don't need to be a Python expert to contribute! Here are ways to help:

### 🐛 Report Bugs

Found a bug? [Open an issue](https://github.com/ussoio/usso/issues/new) with:
- Clear description
- Steps to reproduce
- Expected vs actual behavior
- Environment details (OS, Python version, etc.)

### 💡 Suggest Features

Have an idea? [Start a discussion](https://github.com/ussoio/usso/discussions/new) with:
- Problem description
- Proposed solution
- Use case examples
- Any alternatives considered

### 📖 Improve Documentation

Documentation improvements are always welcome:
- Fix typos
- Add examples
- Clarify confusing sections
- Translate to other languages

### 🔧 Submit Code

Ready to code? Check our [good first issues](https://github.com/ussoio/usso/labels/good%20first%20issue).

---

## Development Setup

### Prerequisites

- Python 3.10+
- Docker and Docker Compose
- Git
- Basic understanding of FastAPI

### Setup Steps

1. **Fork the repository**

   Click "Fork" on [GitHub](https://github.com/ussoio/usso)

2. **Clone your fork**

   ```bash
   git clone https://github.com/YOUR_USERNAME/usso.git
   cd usso
   ```

3. **Set up environment**

   ```bash
   cp sample.env .env
   # Edit .env with your settings
   ```

4. **Start services**

   ```bash
   docker compose up -d
   ```

5. **Install development dependencies**

   ```bash
   cd app
   pip install -r requirements.txt
   pip install -r requirements-dev.txt  # If exists
   ```

6. **Run tests**

   ```bash
   pytest
   ```

---

## Project Structure

```
usso/
├── app/
│   ├── apps/              # Application modules
│   │   ├── identity/      # User management
│   │   ├── access_control/ # Authorization
│   │   ├── tenant/        # Multi-tenancy
│   │   └── ...
│   ├── server/            # Server config
│   ├── utils/             # Utilities
│   └── tests/             # Test suite
├── docs/                  # Documentation
├── docker-compose.yml     # Docker setup
└── README.md
```

---

## Coding Guidelines

### Python Style

We follow **PEP 8** with some modifications:

- **Line length**: 88 characters (Black default)
- **Quotes**: Double quotes for strings
- **Type hints**: Required for function signatures
- **Docstrings**: Google style

```python
def create_user(email: str, password: str) -> User:
    """Create a new user account.
    
    Args:
        email: User's email address
        password: Plain text password (will be hashed)
    
    Returns:
        Created user object
    
    Raises:
        ValueError: If email is invalid
    """
    ...
```

### Code Formatting

We use **Black** for formatting:

```bash
black app/
```

### Linting

We use **Ruff** for linting:

```bash
ruff check app/
```

### Type Checking

We use **mypy** for type checking:

```bash
mypy app/
```

---

## Testing

### Running Tests

```bash
# All tests
pytest

# Specific test file
pytest tests/test_auth.py

# Specific test
pytest tests/test_auth.py::test_login

# With coverage
pytest --cov=app --cov-report=html
```

### Writing Tests

Place tests in `app/tests/`:

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_login_success():
    """Test successful login"""
    response = client.post("/api/sso/v1/auth/login", json={
        "identifier": "test@example.com",
        "secret": "password123"
    })
    
    assert response.status_code == 200
    assert "access_token" in response.json()

def test_login_invalid_password():
    """Test login with wrong password"""
    response = client.post("/api/sso/v1/auth/login", json={
        "identifier": "test@example.com",
        "secret": "wrongpassword"
    })
    
    assert response.status_code == 401
```

### Test Coverage

We aim for **90%+ test coverage**. Check coverage:

```bash
pytest --cov=app --cov-report=term-missing
```

---

## Git Workflow

### Branching Strategy

- `main` - Stable production code
- `develop` - Integration branch
- `feature/*` - New features
- `bugfix/*` - Bug fixes
- `hotfix/*` - Emergency fixes

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructure
- `test`: Tests
- `chore`: Maintenance

**Examples:**

```bash
feat(auth): add passkey authentication

Implements WebAuthn-based passwordless authentication.

Closes #123
```

```bash
fix(tokens): handle expired refresh tokens correctly

Previously expired refresh tokens caused 500 errors.
Now returns 401 with proper error message.
```

### Pull Request Process

1. **Create a branch**

   ```bash
   git checkout -b feature/my-feature
   ```

2. **Make changes**

   Write code, add tests, update docs

3. **Commit**

   ```bash
   git add .
   git commit -m "feat(scope): description"
   ```

4. **Push**

   ```bash
   git push origin feature/my-feature
   ```

5. **Open Pull Request**

   Go to GitHub and click "New Pull Request"

6. **Fill PR template**

   - Description of changes
   - Related issue number
   - Testing done
   - Screenshots (if UI changes)

7. **Address review comments**

   Make requested changes and push

8. **Merge**

   Once approved, we'll merge your PR!

---

## Pull Request Checklist

Before submitting, ensure:

- [ ] Code follows style guide
- [ ] Tests added and passing
- [ ] Documentation updated
- [ ] Commit messages are clear
- [ ] Branch is up to date with `main`
- [ ] No merge conflicts
- [ ] PR description is complete

---

## Code Review

### What We Look For

- **Correctness**: Does it work as intended?
- **Tests**: Are there sufficient tests?
- **Style**: Does it follow our guidelines?
- **Documentation**: Are docs updated?
- **Security**: Any security implications?
- **Performance**: Any performance impact?

### Review Process

1. **Automated checks**: CI runs tests and linting
2. **Code review**: Maintainer reviews your code
3. **Feedback**: You address comments
4. **Approval**: Maintainer approves
5. **Merge**: We merge your changes!

---

## Documentation

### Writing Documentation

Documentation is written in **Markdown** using **MkDocs**.

1. **Edit docs**

   ```bash
   cd docs/
   # Edit .md files
   ```

2. **Preview locally**

   ```bash
   pip install mkdocs mkdocs-material
   mkdocs serve
   ```

   Visit `http://localhost:8000`

3. **Build**

   ```bash
   mkdocs build
   ```

### Documentation Style

- Use clear, simple language
- Include code examples
- Add diagrams where helpful
- Test all code examples

---

## Security

### Reporting Vulnerabilities

**DO NOT** open public issues for security vulnerabilities!

Email: **security@usso.io**

Include:
- Description of vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

We'll respond within 48 hours.

---

## Community

### Code of Conduct

Be respectful, inclusive, and professional. See our [Code of Conduct](https://github.com/ussoio/usso/blob/main/CODE_OF_CONDUCT.md).

### Getting Help

- **GitHub Discussions**: Questions and ideas
- **GitHub Issues**: Bugs and features
- **Email**: support@usso.io

---

## Recognition

Contributors are recognized in:
- GitHub contributors page
- Release notes
- Documentation credits

---

## License

By contributing, you agree that your contributions will be licensed under the **Apache 2.0 License**.

---

## Quick Links

- [GitHub Repository](https://github.com/ussoio/usso)
- [Issue Tracker](https://github.com/ussoio/usso/issues)
- [Discussions](https://github.com/ussoio/usso/discussions)
- [Roadmap](roadmap.md)

---

## Thank You! 🎉

Every contribution, no matter how small, helps make USSO better. We appreciate your support!

---

[← Back to FAQ](faq.md){ .md-button }
[Back to Home →](index.md){ .md-button .md-button--primary }

