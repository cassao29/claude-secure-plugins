# /docker-compose-secure

Generate a secure Docker Compose configuration for your application.

## Usage

```
/docker-compose-secure [service-type] [options]
```

## Examples

```bash
# Generate a full-stack secure configuration
/docker-compose-secure fullstack

# Generate for specific services
/docker-compose-secure postgres redis node

# Generate with public nginx (only nginx exposed)
/docker-compose-secure fullstack --with-nginx
```

## What This Command Does

1. **Analyzes your project** to determine required services
2. **Generates docker-compose.yml** with security hardening:
   - All ports bound to `127.0.0.1`
   - `security_opt: [no-new-privileges:true]`
   - Health checks for all services
   - Resource limits defined
   - Proper network isolation
3. **Creates .env.example** with required environment variables
4. **Outputs security checklist** for review

## Security Features (Always Applied)

| Feature | Default | Configurable |
|---------|---------|--------------|
| Localhost port binding | `127.0.0.1:*` | Yes, use `--public` flag |
| no-new-privileges | `true` | No |
| Health checks | Enabled | No |
| Resource limits | Yes | Yes |
| Logging limits | Yes | Yes |
| Network isolation | Yes | Yes |

## Options

| Option | Description |
|--------|-------------|
| `--public` | Allow specific services to be publicly accessible (requires confirmation) |
| `--dev` | Development mode (relaxed limits, no read_only) |
| `--prod` | Production mode (strict limits, read_only where possible) |
| `--with-nginx` | Add nginx reverse proxy as public entry point |

## Output

The command generates:

1. `docker-compose.yml` - Main configuration (secure by default)
2. `.env.example` - Template for environment variables
3. `docker-compose.override.yml` - Optional overrides for development
