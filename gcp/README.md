# Scrutora Scan on Google Cloud Build

Run offline **DPDPA / HIPAA / GDPR / PCI-DSS / RBI** compliance scanning in
[Cloud Build](https://cloud.google.com/build). Cloud Build executes each step as
a container, so the Scrutora scanner image *is* the build step — no plugin, no
Docker-in-Docker, no Marketplace listing, and no API key. Your code never leaves
the build.

## Quick start

Add a step to your `cloudbuild.yaml`:

```yaml
steps:
  - name: "ghcr.io/nirvahana/dpdp-scan:latest"
    args: ["scan", ".", "--no-ai",
           "--frameworks", "dpdpa,hipaa",
           "--fail-on", "high",
           "--sarif-output", "scrutora.sarif"]
```

The repo is mounted at `/workspace` (the step's working directory), so `.` scans
your whole checkout. A non-zero `--fail-on` severity fails the build and blocks
the pipeline.

A ready-to-use config with substitutions is in [`cloudbuild.yaml`](./cloudbuild.yaml):

```bash
gcloud builds submit --config cloudbuild.yaml \
  --substitutions=_FRAMEWORKS="dpdpa,hipaa,pcidss",_FAIL_ON="critical"
```

## Options

| Flag | Example | Meaning |
|------|---------|---------|
| `--frameworks` | `dpdpa,hipaa,gdpr,pcidss` | Compliance frameworks to check |
| `--fail-on` | `none` … `critical` | Fail the build at or above this severity |
| `--sarif-output` | `scrutora.sarif` | SARIF v2.1.0 report path (relative to `/workspace`) |

## Archiving the SARIF report

To keep the report, publish it to a GCS bucket via the build `artifacts` block
(uncomment it in `cloudbuild.yaml` and set your bucket):

```yaml
artifacts:
  objects:
    location: "gs://YOUR_BUCKET/scrutora/"
    paths: ["scrutora.sarif"]
```

The Cloud Build service account needs `storage.objects.create` on that bucket.

## Image mirrors

The same image is available on Docker Hub if you prefer:
`docker.io/nirvahana/scrutora-scan:latest`.

## Notes

- **Report-only mode:** set `--fail-on none` to surface findings without failing the build.
- **Private pools / VPC-SC:** the image runs fully offline, so it works in restricted build pools with no egress.

Learn more at [scrutora.com](https://scrutora.com) · [GitHub](https://github.com/Scrutora/scrutora-scan)
