# Contributing to Claude Secure Plugins

Thank you for your interest in contributing! This marketplace prioritizes security over features.

## Requirements for Contributions

### 1. Security First

Your plugin MUST follow these security principles:

#### Docker Compose Plugins

```yaml
# REQUIRED: Localhost binding
ports:
  - "127.0.0.1:8080:8080"  # ✅ Correct
  - "8080:8080"             # ❌ Rejected

# REQUIRED: Security options
security_opt:
  - no-new-privileges:true

# REQUIRED: Health checks
healthcheck:
  test: ["CMD", "..."]
  interval: 30s
  timeout: 10s
  retries: 3
```

#### Infrastructure Plugins

- Security groups: Deny all by default
- Encryption: At rest and in transit
- Secrets: Never hardcoded
- Logging: Enabled for audit

### 2. Complete Source Code

- No obfuscated code
- No minified code without source
- No binary blobs
- All dependencies declared

### 3. Documentation Required

Every plugin must include:

```
plugins/category/plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   └── plugin-name/
│       └── SKILL.md         # Skill instructions
├── commands/
│   └── command-name.md      # Command documentation
├── README.md                # Overview and usage
└── SECURITY.md              # Security considerations
```

### 4. Security Documentation

Your `SECURITY.md` must document:

1. What security measures are applied by default
2. What can be configured (and risks of changing)
3. What external access is required (if any)
4. Known limitations or risks

## Submission Process

### 1. Fork and Clone

```bash
git clone https://github.com/YOUR_USERNAME/claude-secure-plugins
cd claude-secure-plugins
```

### 2. Create Plugin Structure

```bash
mkdir -p plugins/category/your-plugin/{.claude-plugin,skills/your-plugin,commands}
```

### 3. Implement Plugin

Follow the templates in `templates/` directory.

### 4. Self-Review Checklist

Before submitting, verify:

- [ ] All ports use `127.0.0.1:` prefix
- [ ] `security_opt: [no-new-privileges:true]` present
- [ ] No hardcoded secrets or passwords
- [ ] Health checks defined
- [ ] Resource limits defined
- [ ] README.md complete
- [ ] SECURITY.md included
- [ ] Tested in isolated environment

### 5. Submit Pull Request

1. Create feature branch: `git checkout -b plugin/your-plugin-name`
2. Commit changes: `git commit -m "Add your-plugin-name plugin"`
3. Push: `git push origin plugin/your-plugin-name`
4. Open PR with:
   - Description of plugin purpose
   - Security measures implemented
   - Testing performed

## Review Process

1. **Automated Checks**
   - Secret scanning
   - Security pattern validation
   - Documentation completeness

2. **Manual Review**
   - Code review by maintainer
   - Security assessment
   - Testing in isolated environment

3. **Approval**
   - Must pass all checks
   - Must address all review comments
   - Must have maintainer approval

## What We Accept

✅ Plugins that:
- Improve security posture
- Follow secure defaults
- Have clear documentation
- Solve real problems

## What We Reject

❌ Plugins that:
- Expose services publicly by default
- Require disabling security features
- Have undocumented behavior
- Include obfuscated code
- Have known vulnerabilities

## Questions?

Open an issue with the `question` label.
