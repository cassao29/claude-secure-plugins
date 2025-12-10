# Claude Secure Plugins

[![Security Audited](https://img.shields.io/badge/security-audited-green.svg)](docs/SECURITY.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**Security-first Claude Code plugins marketplace with audited configurations.**

[🇧🇷 Leia em Português](README.pt-BR.md)

## Why This Marketplace?

After a security incident where insecure AI-generated Docker configurations led to server compromise, we created this marketplace focused on:

1. **Secure by Default** - All configurations follow least privilege principles
2. **Audited Code** - Every plugin is manually reviewed before publication
3. **Documented Practices** - We explain the "why" behind every security decision

## Security Principles

### Docker & Containers
- ✅ Ports ALWAYS bound to `127.0.0.1` by default
- ✅ `security_opt: [no-new-privileges:true]` required
- ✅ Non-root user when possible
- ✅ Read-only filesystem when applicable
- ✅ No `privileged: true` unless explicitly required

### Infrastructure
- ✅ Restrictive firewall by default (deny all, allow specific)
- ✅ Secrets never in configuration files
- ✅ TLS/HTTPS required for exposed services
- ✅ Rate limiting on APIs

### Code
- ✅ Input validation at all entry points
- ✅ No shell command execution with user input
- ✅ Dependencies verified and updated

## Installation

```bash
# Add the marketplace
claude plugin marketplace add cassao29/claude-secure-plugins

# Install a specific plugin
claude plugin install docker-compose-secure@claude-secure-plugins
```

## Available Plugins

### DevOps
| Plugin | Description | Status |
|--------|-------------|--------|
| `docker-compose-secure` | Docker Compose generator with security by default | ✅ Audited |
| `kubernetes-secure` | K8s manifests with PodSecurityContext | ✅ Audited |

### Security
| Plugin | Description | Status |
|--------|-------------|--------|
| `security-scanner` | Vulnerability scanner for code and configs | ✅ Audited |
| `secrets-validator` | Exposed secrets and credentials validator | ✅ Audited |

## Contributing

Read [CONTRIBUTING.md](docs/CONTRIBUTING.md) before submitting plugins.

**Contribution requirements:**
1. Complete source code (no obfuscation)
2. Security documentation
3. Automated tests
4. Pass security audit

## Security Audit

Every plugin undergoes:
1. Manual code review
2. Dependency verification (npm audit, pip audit)
3. SAST tool scanning
4. Behavior testing in isolated environment

See [docs/SECURITY.md](docs/SECURITY.md) for details.

## License

MIT - See [LICENSE](LICENSE)

## Authors

- **Cássio Santos** - Creator and maintainer

---

> ⚠️ **Warning**: This marketplace was created after a real security incident.
> Third-party plugins without auditing may expose your server to attacks.
> [Read the full report](docs/INCIDENT_REPORT.md)
