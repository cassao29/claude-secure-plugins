# /security-scan

Scan infrastructure files for security vulnerabilities and misconfigurations.

## Usage

```bash
/security-scan [path] [options]
```

## Examples

```bash
# Scan current directory
/security-scan

# Scan specific file
/security-scan docker-compose.yml

# Scan with specific severity threshold
/security-scan --min-severity high

# Scan and output JSON
/security-scan --format json
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `--min-severity` | Minimum severity to report (low, medium, high, critical) | low |
| `--format` | Output format (text, json, markdown) | text |
| `--fix` | Attempt automatic fixes | false |
| `--ignore-file` | Path to ignore file | .security-scan-ignore |

## Output

### Text Format (Default)

```
Security Scan Results
=====================

⛔ CRITICAL: 2 issues
⚠️  HIGH: 3 issues
ℹ️  MEDIUM: 1 issue

[DC001] Public Port Binding (CRITICAL)
  File: docker-compose.yml:15
  ports: "8080:8080" exposes service to 0.0.0.0
  Fix: Use "127.0.0.1:8080:8080"

[DC003] Missing Security Options (HIGH)
  File: docker-compose.yml:10
  Service 'web' missing security_opt
  Fix: Add security_opt: [no-new-privileges:true]
```

### JSON Format

```json
{
  "scan_date": "2025-12-10T14:30:00Z",
  "files_scanned": 3,
  "issues": [
    {
      "id": "DC001",
      "severity": "critical",
      "file": "docker-compose.yml",
      "line": 15,
      "message": "Port exposed publicly",
      "fix": "Use 127.0.0.1:8080:8080"
    }
  ]
}
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | No issues found |
| 1 | Low/Medium issues found |
| 2 | High/Critical issues found |

## Integration

Use in CI/CD to fail builds on security issues:

```yaml
- name: Security Scan
  run: |
    /security-scan --min-severity high
```
