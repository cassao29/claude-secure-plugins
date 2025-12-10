# Security Scanner

Scan infrastructure configuration files for security vulnerabilities, misconfigurations, and deviations from best practices.

## Scan Targets

### Docker Compose Files

Scan `docker-compose.yml`, `docker-compose.*.yml`, `compose.yml`

#### Critical Issues (Severity: HIGH)

| Issue | Detection | Fix |
|-------|-----------|-----|
| Public port binding | `ports: ["8080:8080"]` | Use `127.0.0.1:8080:8080` |
| Privileged container | `privileged: true` | Remove or use specific capabilities |
| Host network mode | `network_mode: host` | Use bridge network |
| Writable /etc or /root | Volume mounts to sensitive paths | Use read-only or avoid |
| Root user | No `user:` specified | Add `user: "1000:1000"` |
| Missing security_opt | No `security_opt` section | Add `no-new-privileges:true` |

#### Medium Issues (Severity: MEDIUM)

| Issue | Detection | Fix |
|-------|-----------|-----|
| No health check | Missing `healthcheck:` | Add health check |
| No resource limits | Missing `deploy.resources` | Add CPU/memory limits |
| No restart policy | Missing `restart:` | Add `unless-stopped` |
| Default bridge network | No custom networks | Define isolated networks |
| No logging limits | Missing `logging.options` | Add max-size, max-file |

#### Low Issues (Severity: LOW)

| Issue | Detection | Fix |
|-------|-----------|-----|
| Using `latest` tag | `image: app:latest` | Pin specific version |
| Deprecated version key | `version: "3.8"` | Remove (optional in modern Docker) |
| No container name | Missing `container_name` | Add for easier management |

### Kubernetes Manifests

Scan `*.yaml`, `*.yml` files with Kubernetes API objects

#### Critical Issues

| Issue | Detection | Fix |
|-------|-----------|-----|
| Running as root | `runAsUser: 0` or missing | Set `runAsNonRoot: true` |
| Privileged pod | `privileged: true` | Set `privileged: false` |
| Host PID/Network | `hostPID: true` | Remove |
| No resource limits | Missing `resources.limits` | Define limits |
| Writable root FS | `readOnlyRootFilesystem: false` | Set to `true` |
| All capabilities | `capabilities: add: [ALL]` | Drop ALL, add specific |

#### Medium Issues

| Issue | Detection | Fix |
|-------|-----------|-----|
| No network policy | Missing NetworkPolicy | Create restrictive policy |
| No pod security | Missing securityContext | Add security context |
| No liveness probe | Missing livenessProbe | Add health check |
| Service type LoadBalancer | `type: LoadBalancer` | Use ClusterIP + Ingress |

### Terraform Files

Scan `*.tf`, `*.tfvars` files

#### Critical Issues

| Issue | Detection | Fix |
|-------|-----------|-----|
| Open security group | `0.0.0.0/0` in ingress | Restrict to specific IPs |
| Public S3 bucket | `acl = "public-read"` | Use `private` |
| Unencrypted storage | Missing encryption config | Enable encryption |
| Hardcoded secrets | `password = "..."` | Use variables/secrets manager |
| No state encryption | Plain state file | Enable encryption |

#### Medium Issues

| Issue | Detection | Fix |
|-------|-----------|-----|
| Default VPC | Using default VPC | Create custom VPC |
| No VPC flow logs | Missing flow logs | Enable for audit |
| IMDSv1 enabled | Missing IMDSv2 requirement | Require IMDSv2 |
| No backup | Missing backup config | Enable automated backups |

## Scan Output Format

### Summary Report

```
╔══════════════════════════════════════════════════════════════╗
║              SECURITY SCAN REPORT                            ║
║              2025-12-10 14:30:00                             ║
╠══════════════════════════════════════════════════════════════╣
║  Files Scanned: 5                                            ║
║  Issues Found:  12                                           ║
║                                                              ║
║  ⛔ Critical: 3                                              ║
║  ⚠️  Medium:   5                                              ║
║  ℹ️  Low:      4                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### Detailed Findings

```
⛔ CRITICAL: Public Port Exposure
   File: docker-compose.yml
   Line: 15
   Service: web

   Found:
     ports:
       - "8080:8080"

   Risk: Port 8080 is exposed to all network interfaces (0.0.0.0).
         Attackers can access this service from the internet.

   Fix:
     ports:
       - "127.0.0.1:8080:8080"

   Reference: CIS Docker Benchmark 5.13
```

## Scan Rules

### Docker Compose Rules

```yaml
rules:
  - id: DC001
    name: public-port-binding
    severity: critical
    pattern: 'ports:\s*-\s*"?\d+:\d+"?'
    exclude: '127\.0\.0\.1:|localhost:'
    message: "Port exposed publicly without localhost binding"
    fix: "Prepend 127.0.0.1: to port binding"

  - id: DC002
    name: privileged-container
    severity: critical
    pattern: 'privileged:\s*true'
    message: "Container running in privileged mode"
    fix: "Remove privileged: true or use specific capabilities"

  - id: DC003
    name: missing-security-opt
    severity: high
    pattern: 'services:'
    require: 'security_opt:'
    message: "Service missing security_opt configuration"
    fix: "Add security_opt: [no-new-privileges:true]"

  - id: DC004
    name: no-health-check
    severity: medium
    pattern: 'services:'
    require: 'healthcheck:'
    message: "Service missing health check"
    fix: "Add healthcheck with test, interval, timeout"

  - id: DC005
    name: no-resource-limits
    severity: medium
    pattern: 'services:'
    require: 'deploy:\s*resources:\s*limits:'
    message: "Service missing resource limits"
    fix: "Add deploy.resources.limits for CPU and memory"

  - id: DC006
    name: host-network
    severity: critical
    pattern: 'network_mode:\s*host'
    message: "Container using host network mode"
    fix: "Use bridge network with explicit port bindings"

  - id: DC007
    name: latest-tag
    severity: low
    pattern: 'image:\s*\S+:latest'
    message: "Using latest tag which is mutable"
    fix: "Pin to specific version tag"
```

### Kubernetes Rules

```yaml
rules:
  - id: K8S001
    name: run-as-root
    severity: critical
    pattern: 'runAsUser:\s*0'
    message: "Container configured to run as root"
    fix: "Set runAsNonRoot: true or specify non-root user"

  - id: K8S002
    name: privileged-pod
    severity: critical
    pattern: 'privileged:\s*true'
    message: "Pod running in privileged mode"
    fix: "Set privileged: false"

  - id: K8S003
    name: no-security-context
    severity: high
    pattern: 'containers:'
    require: 'securityContext:'
    message: "Container missing security context"
    fix: "Add securityContext with appropriate restrictions"

  - id: K8S004
    name: capabilities-all
    severity: critical
    pattern: 'capabilities:\s*add:\s*\[\s*ALL'
    message: "Container granted all capabilities"
    fix: "Drop ALL capabilities and add only required ones"

  - id: K8S005
    name: host-pid
    severity: critical
    pattern: 'hostPID:\s*true'
    message: "Pod sharing host PID namespace"
    fix: "Remove hostPID: true"
```

### Terraform Rules

```yaml
rules:
  - id: TF001
    name: open-security-group
    severity: critical
    pattern: 'cidr_blocks\s*=\s*\["0\.0\.0\.0/0"\]'
    context: 'ingress'
    message: "Security group allows traffic from anywhere"
    fix: "Restrict to specific IP ranges"

  - id: TF002
    name: public-s3-bucket
    severity: critical
    pattern: 'acl\s*=\s*"public'
    message: "S3 bucket configured for public access"
    fix: "Use acl = \"private\" and bucket policies"

  - id: TF003
    name: unencrypted-ebs
    severity: high
    pattern: 'aws_ebs_volume'
    require: 'encrypted\s*=\s*true'
    message: "EBS volume not encrypted"
    fix: "Add encrypted = true"

  - id: TF004
    name: hardcoded-credentials
    severity: critical
    pattern: '(password|secret|key)\s*=\s*"[^"$]+'
    message: "Hardcoded credentials in Terraform"
    fix: "Use variables or secrets manager"
```

## Integration

### As Pre-Write Hook

This skill can be integrated as a validation hook:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "security-scan --file $FILE_PATH"
      }]
    }]
  }
}
```

### As CI/CD Step

```yaml
# GitHub Actions
- name: Security Scan
  run: |
    claude plugin run security-scanner --path .
    if [ $? -ne 0 ]; then
      echo "Security issues found!"
      exit 1
    fi
```

## Remediation Guidance

When issues are found, the scanner provides:

1. **What**: Clear description of the issue
2. **Why**: Risk explanation with real-world impact
3. **Where**: Exact file and line number
4. **How**: Specific fix with code example
5. **Reference**: Link to security benchmark or standard

## False Positive Handling

Some patterns may be intentional. Mark as accepted:

```yaml
# security-scan: ignore DC001
ports:
  - "0.0.0.0:443:443"  # Intentionally public (reverse proxy)
```

Or create `.security-scan-ignore`:

```yaml
ignore:
  - rule: DC001
    file: nginx/docker-compose.yml
    reason: "Nginx is intentionally public-facing"
```

## Compliance Mapping

| Rule | CIS Docker | CIS Kubernetes | OWASP |
|------|------------|----------------|-------|
| DC001 | 5.13 | - | A05 |
| DC002 | 5.4 | - | A01 |
| DC003 | 5.25 | - | A05 |
| K8S001 | - | 5.2.6 | A01 |
| K8S002 | - | 5.2.1 | A01 |
| TF001 | - | - | A05 |
