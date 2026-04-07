# CleanCloud Scan Action

GitHub Action for [CleanCloud](https://github.com/cleancloud-io/cleancloud) — a read-only cloud hygiene scanner for AWS, Azure, and GCP that finds orphaned resources, detects idle AI/ML waste ($500–$23K/month per endpoint), and enforces policy-as-code in CI.

## Usage

### AWS — single account (OIDC)

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

### AWS — all accounts in an Organization

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/CleanCloudCIReadOnly
    aws-region: us-east-1

- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    org: 'true'
    all-regions: 'true'
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### AWS — specific accounts (inline)

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    accounts: '111111111111,222222222222,333333333333'
    all-regions: 'true'
    output: json
    output-file: scan-results.json
```

### AWS — specific accounts (config file)

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    multi-account: .cleancloud/accounts.yaml
    all-regions: 'true'
    output: json
    output-file: scan-results.json
```

### Azure — single subscription (Workload Identity)

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

### Azure — all subscriptions under a Management Group

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

- uses: cleancloud-io/scan-action@v1
  with:
    provider: azure
    management-group: my-management-group-id
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### Azure — specific subscriptions

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: azure
    subscription: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx, yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy'
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### GCP — all projects (Workload Identity Federation)

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

- uses: cleancloud-io/scan-action@v1
  with:
    provider: gcp
    all-projects: 'true'
    fail-on-confidence: HIGH
    fail-on-cost: '100'
    output: json
    output-file: scan-results.json
```

### GCP — single project

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

- uses: cleancloud-io/scan-action@v1
  with:
    provider: gcp
    project: my-gcp-project-id
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### GCP — specific projects (comma-separated)

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: gcp
    project: 'project-id-1, project-id-2, project-id-3'
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### AI/ML waste detection — AWS (SageMaker)

Detect idle SageMaker endpoints with zero invocations. GPU-backed endpoints flagged HIGH risk ($500–$23K/month).

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    all-regions: 'true'
    category: ai
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### AI/ML waste detection — Azure (AML compute clusters)

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: azure
    category: ai
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### AI/ML waste detection — GCP (Vertex AI endpoints)

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: gcp
    all-projects: 'true'
    category: ai
    fail-on-confidence: HIGH
    output: json
    output-file: scan-results.json
```

### Hygiene + AI/ML together

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    org: 'true'
    all-regions: 'true'
    category: all
    fail-on-confidence: HIGH
    fail-on-cost: '500'
    output: json
    output-file: scan-results.json
```

### With policy-as-code config

Commit a `cleancloud.yaml` to your repo — exceptions, thresholds, and tag filtering are picked up automatically:

```yaml
- uses: actions/checkout@v4   # required so cleancloud.yaml is available

- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    all-regions: 'true'
    # config auto-detected from cleancloud.yaml in repo root
    # or pass explicitly:
    # config: configs/prod.yaml
    output: json
    output-file: scan-results.json
```

`cleancloud.yaml` (committed to repo root):
```yaml
defaults:
  confidence: MEDIUM
  min_cost: 10
exceptions:
  - rule_id: aws.ec2.instance.stopped
    resource_id: i-0abc1234567890def
    reason: "Bastion host — started on demand"
    expires_at: "2026-12-31"
thresholds:
  fail_on_confidence: HIGH
  fail_on_cost: 500
```

See [policy config reference](https://github.com/cleancloud-io/cleancloud/blob/main/docs/configuration.md) for full options.

### Scan a specific region

**AWS:**
```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    region: us-east-1
    fail-on-confidence: HIGH
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
    output: json
    output-file: scan-results.json
```

## Inputs

### General

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `provider` | Yes | — | `aws`, `azure`, or `gcp` |
| `category` | No | `hygiene` | `hygiene` (default), `ai` (SageMaker / AML / Vertex AI — all clouds), or `all` |
| `region` | No | — | Specific region (AWS) or location filter (Azure, optional) |
| `fail-on-confidence` | No | — | Fail if findings at or above this level: `LOW`, `MEDIUM`, or `HIGH` |
| `fail-on-cost` | No | — | Fail if estimated monthly waste exceeds this USD amount |
| `fail-on-findings` | No | `false` | Fail on any finding |
| `output` | No | `human` | Output format: `human`, `json`, `csv`, or `markdown` |
| `output-file` | No | — | Path to write output file (required for `json`/`csv`, optional for `markdown`) |
| `artifact-name` | No | — | Upload the output file as a GitHub Actions artifact with this name. Leave empty to skip upload. |
| `config` | No | — | Path to `cleancloud.yaml` config file. Auto-detected from repo root if omitted (requires `actions/checkout` first). |
| `explain` | No | `false` | Print suppression reason for each filtered finding. Useful for debugging policy config. |
| `skip` | No | — | Comma-separated rule IDs to skip. Example: `aws.ec2.ami.old,aws.resource.untagged` |
| `ignore-tag` | No | — | Ignore findings by tag. Comma-separated `key` or `key:value` pairs. Example: `env:dev,temporary` |
| `version` | No | latest | CleanCloud version to install (e.g. `1.7.2`) |

### AWS

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `all-regions` | One of `all-regions` or `region` required | `false` | Scan all active regions |
| `org` | No | `false` | Auto-discover and scan all accounts in your AWS Organization. Requires `organizations:ListAccounts` on the hub role. |
| `accounts` | No | — | Comma-separated AWS account IDs to scan. Example: `111111111111,222222222222` |
| `multi-account` | No | — | Path to accounts config file. Example: `.cleancloud/accounts.yaml` |
| `role-name` | No | `CleanCloudReadOnlyRole` | IAM role name to assume in each spoke account |
| `external-id` | No | — | External ID for cross-account role assumption, if required by the spoke trust policy |
| `concurrency` | No | `3` | Number of accounts to scan in parallel. Keep low to avoid API throttling. |
| `timeout` | No | `3600` | Total scan timeout in seconds across all accounts |
| `per-account-regions` | No | `false` | Detect active regions per account instead of once on the hub. Slower but accurate if accounts use different regions. |

> `org`, `accounts`, and `multi-account` are mutually exclusive — use only one.

### Azure

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `subscription` | No | — | Comma-separated subscription IDs to scan. Omit to scan all accessible subscriptions. |
| `management-group` | No | — | Management Group ID — auto-discovers all subscriptions underneath. |

> `subscription` and `management-group` are mutually exclusive — use only one.

### GCP

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `project` | No | — | Comma-separated GCP project IDs to scan. Omit to use the default project from credentials. |
| `all-projects` | No | `false` | Scan all accessible GCP projects. Requires `roles/browser` on the service account. |

> `project` and `all-projects` are mutually exclusive — use only one.

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
- **GCP:** Use [`google-github-actions/auth`](https://github.com/google-github-actions/auth) with Workload Identity Federation. See [GCP setup guide](https://github.com/cleancloud-io/cleancloud/blob/main/docs/gcp.md).

## Versioning

This action installs the latest CleanCloud from PyPI by default. To pin a specific version:

```yaml
- uses: cleancloud-io/scan-action@v1
  with:
    provider: aws
    version: '1.9.0'
```

## As featured in

- [Korben](https://korben.info/cleancloud-nettoyeur-cloud-aws-azure.html) 🇫🇷 — Major French tech publication
- [Last Week in AWS #457](https://www.lastweekinaws.com/newsletter/15259/) — Corey Quinn's weekly AWS newsletter

## Links

- [CleanCloud CLI](https://github.com/cleancloud-io/cleancloud)
- [CI/CD guide](https://github.com/cleancloud-io/cleancloud/blob/main/docs/ci.md)
- [Detection rules](https://github.com/cleancloud-io/cleancloud/blob/main/docs/rules.md)
