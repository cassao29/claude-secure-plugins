# Security Policy

## Our Commitment

This marketplace was created after a real security incident caused by insecure default configurations. We are committed to providing security-first plugins that protect users from common attack vectors.

## Security Principles

### 1. Secure by Default

All plugins in this marketplace follow the principle of secure defaults:

- **Port bindings**: Always `127.0.0.1:port:port`, never `port:port`
- **Privileges**: Always `no-new-privileges:true`
- **Network access**: Internal networks by default
- **Filesystem**: Read-only where applicable
- **Secrets**: Never hardcoded, always environment variables

### 2. Explicit Insecurity

Reducing security requires explicit user action:

```yaml
# User must explicitly request public exposure
ports:
  - "0.0.0.0:443:443"  # WITH WARNING COMMENT
```

### 3. Defense in Depth

Multiple layers of security:

1. Application-level security
2. Container-level restrictions
3. Network isolation
4. Resource limits
5. Monitoring and logging

## Audit Process

Every plugin undergoes:

### 1. Manual Code Review

- All code reviewed by maintainer
- Focus on security-critical sections
- Check for secret exposure
- Verify secure defaults

### 2. Automated Scanning

- Static analysis for common vulnerabilities
- Dependency vulnerability scanning
- Secret detection in all files

### 3. Behavioral Testing

- Test in isolated environment
- Verify network bindings
- Confirm security options applied
- Test with various inputs

## Reporting Vulnerabilities

If you find a security vulnerability:

1. **DO NOT** open a public issue
2. Email: security@example.com
3. Include:
   - Plugin name and version
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact

We will respond within 48 hours.

## Known Security Considerations

### Docker Compose Files

- Default Docker behavior binds to `0.0.0.0`
- Always prepend `127.0.0.1:` to port bindings
- Use internal networks for databases
- Enable health checks for all services

### Kubernetes Manifests

- Use NetworkPolicies to restrict traffic
- Enable PodSecurityPolicies
- Use non-root containers
- Limit resource requests/limits

### Terraform Configurations

- Security groups deny all by default
- Enable encryption at rest
- Use private subnets for databases
- Enable VPC flow logs

## Third-Party Dependencies

We minimize dependencies, but when used:

- All dependencies pinned to specific versions
- Regular vulnerability scanning
- Updates applied promptly for security fixes
- No dependencies with known critical vulnerabilities

## Changelog

### 2025-12-10

- Initial security policy
- Created after incident response
- Implemented secure defaults for all plugins
