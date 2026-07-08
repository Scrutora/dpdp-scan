# Scrutora Scan — GitHub Action

Free **DPDPA / HIPAA / GDPR / PCI-DSS compliance** code scanning in CI. Findings
cite the exact obligation (e.g. DPDPA §8(5)), not just a generic rule id, and
surface in the **Security → Code scanning** tab via SARIF. Runs offline — **no
API key, your code never leaves the runner.**

## Quick start

Add `.github/workflows/compliance.yml` to your repo:

```yaml
name: Compliance
on: [pull_request]
permissions:
  contents: read
  security-events: write   # required to upload SARIF to Code Scanning
jobs:
  scrutora-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - id: scan
        uses: Scrutora/scrutora-scan@v1
        with:
          frameworks: dpdpa,hipaa   # default: dpdpa
          fail-on: high             # none|low|medium|high|critical (default: high)
      - if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: ${{ steps.scan.outputs.sarif-file }}
```

Open a pull request (or push) and the scan runs automatically.

## Where results appear

1. **Security → Code scanning** tab — each finding as an alert with the file,
   line, severity, and the compliance obligation it maps to. Click an alert for
   the full message and remediation. *(Free on public repos; private repos need
   GitHub Advanced Security — see below.)*
2. **The Actions run log** — the full scan summary (counts by severity) under the
   `scrutora-scan` step, for any repo.
3. **The PR check** — with `fail-on` set (default `high`), the job fails when a
   finding at/above that severity is present, so it can block a merge. Use
   `fail-on: none` to report without failing.

## Inputs

| Input | Default | Description |
|---|---|---|
| `path` | `.` | Directory/file to scan |
| `frameworks` | `dpdpa` | Comma-separated: `dpdpa, hipaa, gdpr, pcidss, …` |
| `fail-on` | `high` | Fail the job at/above this severity; `none` = report-only |
| `sarif-file` | `scrutora.sarif` | Where to write SARIF v2.1.0 |

## Output

- `sarif-file` — path to the generated SARIF (pipe to `upload-sarif`).

## Private repositories

SARIF rendering in the Code Scanning tab is free for **public** repos; **private**
repos require GitHub Advanced Security. Without GHAS the job still runs and
`fail-on` still gates the build — read the summary in the Actions log instead.

## About

The free CI wedge for [Scrutora](https://scrutora.com) — the code → cloud →
consent compliance platform. This Action gives you DPDPA/HIPAA findings in your
pull requests; the Scrutora platform adds cross-module data lineage, RoPA, cloud
posture (CSPM), consent management and AI remediation.

More integrations (GitLab, CircleCI, Bitbucket, Azure Pipelines, Google Cloud
Build, pre-commit, Docker, VS Code): <https://scrutora.com/integrations>.
