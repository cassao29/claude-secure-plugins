# Docker Compose Secure Generator

Generate Docker Compose configurations with **security hardening by default**.

## Why This Plugin?

Standard Docker Compose generators create configurations like:

```yaml
ports:
  - "8080:8080"  # Exposed to 0.0.0.0 (entire internet!)
```

This plugin generates:

```yaml
ports:
  - "127.0.0.1:8080:8080"  # Only accessible from localhost
security_opt:
  - no-new-privileges:true
```

## The Problem This Solves

On December 10, 2025, a server was compromised because Docker ports were exposed publicly. Attackers installed a cryptominer and used the server for DDoS attacks. The root cause: `"3001:3000"` instead of `"127.0.0.1:3001:3000"`.

This plugin ensures that never happens again.

## Installation

```bash
claude plugin install docker-compose-secure@claude-secure-plugins
```

## Usage

```bash
/docker-compose-secure
```

Or invoke the skill by asking:
> "Generate a secure docker-compose for a Node.js app with PostgreSQL and Redis"

## Security Features

| Feature | Status | Description |
|---------|--------|-------------|
| Localhost binding | ✅ Default | All ports bound to 127.0.0.1 |
| no-new-privileges | ✅ Always | Prevents privilege escalation |
| Health checks | ✅ Default | All services have health checks |
| Resource limits | ✅ Default | CPU and memory limits defined |
| Non-root user | ✅ Recommended | Uses non-root when possible |
| Read-only FS | ✅ When applicable | Prevents filesystem modifications |
| Network isolation | ✅ Default | Internal networks for databases |
| Log limits | ✅ Default | Prevents log exhaustion |

## License

MIT
