# CleanCloud Scan Action

GitHub Action for [CleanCloud](https://github.com/cleancloud-io/cleancloud) — a read-only cloud hygiene scanner for AWS and Azure that finds orphaned resources and enforces hygiene in CI.

## Usage

### AWS (OIDC)

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/CleanCloudCIReadOnly
    aws-region: us-east-1

- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    all-regions: 'true'
    fail-on-confidence: HIGH
    fail-on-cost: '100'
    output: json
    output-file: scan-results.json
```

### Azure (Workload Identity)

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

- uses: cleancloud-io/scan-action@v1
  with:
    provider: azure
    fail-on-confidence: HIGH
    fail-on-cost: '100'
    output: json
    output-file: scan-results.json
```

### Scan a specific region

**AWS:**
```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    region: us-east-1
    fail-on-confidence: HIGH
    fail-on-cost: '100'
    output: json
    output-file: scan-results.json
```

**Azure** (`region` is optional — omit to scan all locations):
```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: azure
    region: westeurope
    fail-on-confidence: HIGH
    fail-on-cost: '100'
    output: json
    output-file: scan-results.json
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `provider` | Yes | — | `aws` or `azure` |
| `all-regions` | AWS: one of `all-regions` or `region` required | `false` | Scan all active regions (AWS only) |
| `region` | AWS: one of `all-regions` or `region` required | — | Specific region (AWS) or location filter (Azure, optional) |
| `fail-on-confidence` | No | — | Fail if findings at or above this level: `LOW`, `MEDIUM`, or `HIGH` |
| `fail-on-cost` | No | — | Fail if estimated monthly waste exceeds this USD amount |
| `fail-on-findings` | No | `false` | Fail on any finding |
| `output` | No | `human` | Output format: `human`, `json`, `csv`, or `markdown` |
| `output-file` | No | — | Path to write output file (required for `json`/`csv`, optional for `markdown`) |
| `version` | No | latest | CleanCloud version to install (e.g. `1.7.2`) |

> **AWS note:** You must provide either `all-regions: 'true'` or a specific `region`. Omitting both will cause the scan to fail. For Azure, `region` is optional — omitting it scans all accessible locations.

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | No policy violations |
| `1` | Configuration error or unexpected failure |
| `2` | Policy violation — findings detected (when enforcement enabled) |
| `3` | Missing credentials or insufficient permissions |

## How it works

This action installs CleanCloud from PyPI and runs it directly on the runner. For Docker-based CI, use the [Docker image](https://hub.docker.com/r/getcleancloud/cleancloud) directly instead of this action.

## Authentication

CleanCloud is read-only — it never creates, modifies, or deletes resources. Set up authentication before calling this action:

- **AWS:** Use [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) with OIDC. See [AWS setup guide](https://github.com/cleancloud-io/cleancloud/blob/main/docs/aws.md).
- **Azure:** Use [`azure/login`](https://github.com/Azure/login) with Workload Identity Federation. See [Azure setup guide](https://github.com/cleancloud-io/cleancloud/blob/main/docs/azure.md).

## Versioning

This action installs the latest CleanCloud from PyPI by default. To pin a specific version:

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    version: '1.7.2'
```

## As featured in

- [Korben](https://korben.info/cleancloud-nettoyeur-cloud-aws-azure.html) 🇫🇷 — Major French tech publication
- [Last Week in AWS #457](https://www.lastweekinaws.com/newsletter/15259/) — Corey Quinn's weekly AWS newsletter

## Links

- [CleanCloud CLI](https://github.com/cleancloud-io/cleancloud)
- [CI/CD guide](https://github.com/cleancloud-io/cleancloud/blob/main/docs/ci.md)
- [Detection rules](https://github.com/cleancloud-io/cleancloud/blob/main/docs/rules.md)
