# Secrets Validator

Detect and prevent exposure of secrets, credentials, and sensitive data in code and configuration files.

## What This Skill Detects

### High Severity (Block Immediately)

| Pattern | Description | Example |
|---------|-------------|---------|
| AWS Keys | AWS Access Key ID | `AKIA[0-9A-Z]{16}` |
| AWS Secret | AWS Secret Access Key | `[0-9a-zA-Z/+]{40}` |
| GitHub Token | Personal Access Token | `ghp_[0-9a-zA-Z]{36}` |
| GitHub OAuth | OAuth Access Token | `gho_[0-9a-zA-Z]{36}` |
| Private Keys | RSA/DSA/EC private keys | `-----BEGIN.*PRIVATE KEY-----` |
| Database URLs | Connection strings with passwords | `postgres://user:pass@host` |
| API Keys | Generic API key patterns | `api[_-]?key.*['\"][0-9a-zA-Z]{20,}` |

### Medium Severity (Warn)

| Pattern | Description | Example |
|---------|-------------|---------|
| Slack Token | Slack bot/user tokens | `xox[baprs]-[0-9a-zA-Z-]+` |
| Stripe Key | Stripe API keys | `sk_live_[0-9a-zA-Z]{24}` |
| SendGrid | SendGrid API key | `SG\.[0-9A-Za-z-_]{22}\.[0-9A-Za-z-_]{43}` |
| Twilio | Account SID/Auth Token | `AC[0-9a-fA-F]{32}` |
| JWT | JSON Web Tokens | `eyJ[A-Za-z0-9-_]+\.eyJ[A-Za-z0-9-_]+` |

### Low Severity (Flag for Review)

| Pattern | Description |
|---------|-------------|
| `password=` | Hardcoded password assignment |
| `secret=` | Hardcoded secret assignment |
| `token=` | Hardcoded token assignment |
| IP addresses | Internal/private IP addresses |
| Email addresses | Potentially sensitive emails |

## Detection Patterns

### AWS Credentials

```python
# Patterns to detect
AWS_ACCESS_KEY = r'AKIA[0-9A-Z]{16}'
AWS_SECRET_KEY = r'(?<![A-Za-z0-9/+=])[A-Za-z0-9/+=]{40}(?![A-Za-z0-9/+=])'
AWS_ACCOUNT_ID = r'\b[0-9]{12}\b'
```

### GitHub Tokens

```python
GITHUB_PAT = r'ghp_[0-9a-zA-Z]{36}'
GITHUB_OAUTH = r'gho_[0-9a-zA-Z]{36}'
GITHUB_APP = r'ghu_[0-9a-zA-Z]{36}|ghs_[0-9a-zA-Z]{36}'
GITHUB_REFRESH = r'ghr_[0-9a-zA-Z]{36}'
```

### Database Connection Strings

```python
# PostgreSQL
POSTGRES_URL = r'postgres(?:ql)?://[^:]+:[^@]+@[^/]+/[^\s]+'

# MySQL
MYSQL_URL = r'mysql://[^:]+:[^@]+@[^/]+/[^\s]+'

# MongoDB
MONGO_URL = r'mongodb(?:\+srv)?://[^:]+:[^@]+@[^/]+/[^\s]+'

# Redis
REDIS_URL = r'redis://[^:]+:[^@]+@[^/]+(?:/\d+)?'
```

### Private Keys

```python
PRIVATE_KEY = r'-----BEGIN\s+(?:RSA|DSA|EC|OPENSSH|PGP)\s+PRIVATE\s+KEY-----'
```

### Generic Secrets

```python
# Environment variables with secrets
ENV_SECRET = r'(?:PASSWORD|SECRET|TOKEN|API_KEY|APIKEY|AUTH|CREDENTIAL)[_\s]*[=:]\s*[\'"]?[^\s\'"]{8,}[\'"]?'

# Hardcoded in code
CODE_SECRET = r'(?:password|secret|token|api_key|apikey|auth_token)\s*[=:]\s*[\'"][^\'"]{8,}[\'"]'
```

## Files to Always Scan

### Configuration Files

```
docker-compose.yml
docker-compose.*.yml
.env
.env.*
*.env
config.yml
config.json
settings.py
settings.json
application.properties
application.yml
```

### Infrastructure Files

```
*.tf
*.tfvars
terraform.tfstate
*.yaml (Kubernetes)
*.yml (Ansible)
serverless.yml
```

### Code Files (High Risk Areas)

```
**/config/**
**/settings/**
**/credentials/**
**/.aws/**
**/.ssh/**
```

## Files to NEVER Commit

Create or validate `.gitignore` includes:

```gitignore
# Environment files
.env
.env.*
*.env
!.env.example

# Credentials
credentials.json
service-account.json
*-credentials.json
*.pem
*.key
*.p12
*.pfx

# AWS
.aws/credentials
.aws/config

# SSH
.ssh/
id_rsa*
id_dsa*
id_ed25519*

# Terraform
*.tfstate
*.tfstate.*
.terraform/

# IDE
.idea/
.vscode/settings.json
```

## Validation Process

When writing or editing files, this skill:

1. **Pre-Write Scan**
   - Check new content for secret patterns
   - Block if high-severity secrets detected
   - Warn for medium-severity patterns

2. **Suggest Alternatives**
   ```yaml
   # Instead of:
   DB_PASSWORD: supersecret123

   # Use:
   DB_PASSWORD: ${DB_PASSWORD}  # From environment
   ```

3. **Recommend .env Pattern**
   ```bash
   # Create .env (git-ignored)
   DB_PASSWORD=supersecret123

   # Create .env.example (committed)
   DB_PASSWORD=your_password_here
   ```

## Response When Secrets Detected

### High Severity - Block

```
⛔ SECRET DETECTED - BLOCKING WRITE

Found: AWS Secret Access Key
File: docker-compose.yml
Line: 15

The following line contains what appears to be an AWS Secret Key:
  AWS_SECRET_ACCESS_KEY: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

ACTION REQUIRED:
1. Remove the secret from the file
2. Use environment variables instead:
   AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
3. Store the actual value in .env (git-ignored)
```

### Medium Severity - Warn

```
⚠️ POTENTIAL SECRET DETECTED

Found: Possible API key
File: config.js
Line: 23

Consider:
1. Moving this to environment variables
2. Using a secrets manager (AWS Secrets Manager, HashiCorp Vault)
3. Adding this file to .gitignore

Proceed with caution.
```

## Integration with Git Hooks

Recommend creating `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# Pre-commit hook to scan for secrets

patterns=(
    'AKIA[0-9A-Z]{16}'
    'ghp_[0-9a-zA-Z]{36}'
    '-----BEGIN.*PRIVATE KEY-----'
    'password\s*=\s*['\''"][^'\''"]{8,}'
)

for pattern in "${patterns[@]}"; do
    if git diff --cached | grep -E "$pattern"; then
        echo "ERROR: Potential secret detected!"
        echo "Pattern: $pattern"
        exit 1
    fi
done
```

## Best Practices

### DO

1. Use environment variables for all secrets
2. Use `.env.example` for documentation (no real values)
3. Use secrets managers in production (AWS Secrets Manager, Vault)
4. Rotate secrets regularly
5. Use different secrets for dev/staging/production

### DON'T

1. Commit `.env` files
2. Hardcode secrets in code
3. Log secrets (even in debug mode)
4. Share secrets via chat/email
5. Use the same secret across environments

## Environment Variable Template

When generating configurations, always create `.env.example`:

```bash
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=myapp
DB_PASSWORD=change_me_in_production

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=change_me_in_production

# API Keys (get from respective services)
STRIPE_SECRET_KEY=sk_test_your_key_here
SENDGRID_API_KEY=your_api_key_here

# Application
SECRET_KEY=generate_a_random_32_char_string
JWT_SECRET=generate_another_random_string

# AWS (if needed)
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
```
