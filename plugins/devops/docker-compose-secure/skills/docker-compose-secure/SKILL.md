# Docker Compose Secure Generator

Generate Docker Compose configurations with **security hardening by default**.

## Security Principles

This skill enforces security best practices that prevent common attack vectors:

### 1. Port Binding - CRITICAL

**ALWAYS bind ports to localhost (127.0.0.1) by default.**

```yaml
# ✅ CORRECT - Secure by default
ports:
  - "127.0.0.1:8080:8080"
  - "127.0.0.1:5432:5432"

# ❌ NEVER DO THIS - Exposes to entire internet
ports:
  - "8080:8080"        # Binds to 0.0.0.0!
  - "5432:5432"        # Database exposed publicly!
```

**Why this matters:** A port binding like `"3000:3000"` exposes the service to the entire internet (0.0.0.0). Attackers actively scan for exposed development servers, databases, and admin panels. This exact vulnerability led to server compromise with cryptominer malware.

**Only use `0.0.0.0` when:**
- The service is explicitly designed for public access
- A reverse proxy (nginx, traefik) handles TLS termination
- Firewall rules restrict access appropriately
- The user explicitly requests public exposure

### 2. Security Options - REQUIRED

**ALWAYS include security_opt in every service:**

```yaml
services:
  app:
    security_opt:
      - no-new-privileges:true
```

**Why:** Prevents privilege escalation attacks. A compromised process cannot gain additional privileges.

### 3. User Configuration - RECOMMENDED

**Run as non-root when possible:**

```yaml
services:
  app:
    user: "1000:1000"
    # OR use the built-in node user for Node.js images
    user: "node"
```

### 4. Read-Only Filesystem - WHEN APPLICABLE

```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

### 5. Resource Limits - RECOMMENDED

**Prevent resource exhaustion attacks:**

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 128M
```

### 6. Network Isolation

**Create isolated networks, don't use default bridge:**

```yaml
networks:
  backend:
    driver: bridge
    internal: true  # No external access
  frontend:
    driver: bridge

services:
  db:
    networks:
      - backend  # Only accessible from backend network

  app:
    networks:
      - backend
      - frontend
```

### 7. Health Checks - REQUIRED

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 8. Environment Variables - SECURE HANDLING

**NEVER hardcode secrets in docker-compose.yml:**

```yaml
# ✅ CORRECT - Use environment files
services:
  app:
    env_file:
      - .env  # Git-ignored file with secrets

# ✅ CORRECT - Use Docker secrets (Swarm mode)
services:
  app:
    secrets:
      - db_password

secrets:
  db_password:
    external: true

# ❌ NEVER - Hardcoded secrets
services:
  app:
    environment:
      - DB_PASSWORD=supersecret123  # Will be in git history!
```

### 9. Logging Configuration

**Prevent log file exhaustion:**

```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 10. No Privileged Mode

**NEVER use privileged mode unless absolutely necessary:**

```yaml
# ❌ AVOID - Full host access
services:
  app:
    privileged: true  # DANGEROUS!

# ✅ PREFER - Specific capabilities only
services:
  app:
    cap_add:
      - NET_ADMIN  # Only if needed
    cap_drop:
      - ALL  # Drop all capabilities first
```

## Complete Secure Template

```yaml
services:
  app:
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    user: "1000:1000"
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    ports:
      - "127.0.0.1:8080:8080"
    networks:
      - app-network
    volumes:
      - app-data:/app/data:rw
    tmpfs:
      - /tmp
      - /var/run
    env_file:
      - .env
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  app-network:
    driver: bridge

volumes:
  app-data:
```

## Common Service Templates

### PostgreSQL (Secure)

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "127.0.0.1:5432:5432"
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 1G

volumes:
  postgres-data:

networks:
  backend:
    internal: true
```

### Redis (Secure)

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    ports:
      - "127.0.0.1:6379:6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 256M

volumes:
  redis-data:

networks:
  backend:
    internal: true
```

### Node.js Application (Secure)

```yaml
services:
  node-app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: node-app
    restart: unless-stopped
    user: "node"
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    ports:
      - "127.0.0.1:3000:3000"
    tmpfs:
      - /tmp
    env_file:
      - .env
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - app-network
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 512M

networks:
  app-network:
    driver: bridge
```

### Nginx Reverse Proxy (Secure)

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    ports:
      # Only nginx should be publicly accessible
      - "0.0.0.0:80:80"
      - "0.0.0.0:443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - frontend
      - backend
    depends_on:
      - app
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 128M

networks:
  frontend:
    driver: bridge
  backend:
    internal: true
```

## Validation Checklist

Before generating any Docker Compose file, verify:

- [ ] All ports use `127.0.0.1:` prefix (unless explicitly public-facing like nginx)
- [ ] `security_opt: [no-new-privileges:true]` is present
- [ ] No hardcoded passwords or secrets
- [ ] `restart: unless-stopped` for production services
- [ ] Health checks defined for all services
- [ ] Resource limits defined
- [ ] Logging limits configured
- [ ] No `privileged: true` unless absolutely necessary
- [ ] Sensitive services on internal networks

## Warning About Public Exposure

If a user explicitly requests public port exposure, include this warning:

```yaml
# ⚠️ WARNING: This port is publicly accessible (0.0.0.0)
# Ensure you have:
# - Firewall rules configured (UFW, iptables)
# - TLS/HTTPS enabled
# - Authentication required
# - Rate limiting in place
ports:
  - "0.0.0.0:443:443"  # INTENTIONALLY PUBLIC
```

## References

- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
