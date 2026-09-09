# GuardBee Security Scan Action

[![GuardBee](https://img.shields.io/badge/GuardBee-Security-green)](https://guardbee.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Run GuardBee security scans in your GitHub Actions workflow — secret detection, dependency CVEs, SSL inspection, and DNS checks — with automatic SARIF upload to the GitHub Security tab.

---

## Quick Start

```yaml
- uses: GuardBee/guardbee-security-scan-action@v1
  with:
    fail-on: high
```

---

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `secret-scan` | Run secret scanner | `true` |
| `dep-audit` | Run dependency CVE audit | `true` |
| `ssl-inspect` | Run SSL/TLS inspection | `false` |
| `dns-check` | Run DNS intelligence check | `false` |
| `domain` | Domain for ssl-inspect and dns-check | `''` |
| `fail-on` | Severity threshold: `critical` \| `high` \| `medium` \| `low` | `high` |
| `upload-sarif` | Upload SARIF to GitHub Security tab | `true` |
| `working-directory` | Directory to scan | `.` |

## Outputs

| Output | Description |
|--------|-------------|
| `secret-scan-exit-code` | `0`=clean, `1`=findings, `2`=error |
| `dep-audit-exit-code` | `0`=clean, `1`=findings, `2`=error |

---

## Usage Examples

### Minimal — secrets + dependencies on every push

```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: GuardBee/guardbee-security-scan-action@v1
```

### Full scan with SSL and DNS

```yaml
name: Full Security Scan
on:
  schedule:
    - cron: '0 3 * * 1'   # every Monday at 03:00 UTC
  workflow_dispatch:

jobs:
  security:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: GuardBee/guardbee-security-scan-action@v1
        with:
          ssl-inspect: true
          dns-check: true
          domain: example.com
          fail-on: medium
```

### Low noise — only critical findings fail the build

```yaml
- uses: GuardBee/guardbee-security-scan-action@v1
  with:
    fail-on: critical
    upload-sarif: true
```

### Scan a subdirectory

```yaml
- uses: GuardBee/guardbee-security-scan-action@v1
  with:
    working-directory: backend/
```

### Continue on findings, inspect exit codes

```yaml
- uses: GuardBee/guardbee-security-scan-action@v1
  id: scan
  continue-on-error: true

- name: Check results
  run: |
    echo "Secret scan: ${{ steps.scan.outputs.secret-scan-exit-code }}"
    echo "Dep audit:   ${{ steps.scan.outputs.dep-audit-exit-code }}"
```

---

## SARIF / GitHub Security Tab

When `upload-sarif: true` (the default), scan results are uploaded to the **Security** tab of your repository and annotated directly on pull request diffs.

The action requires the `security-events: write` permission:

```yaml
permissions:
  security-events: write
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | No findings above the `fail-on` threshold |
| `1` | One or more findings at or above the threshold |
| `2` | Scan error (tool not found, invalid config, etc.) |

---

## License

MIT — [GuardBee](https://guardbee.ai)
