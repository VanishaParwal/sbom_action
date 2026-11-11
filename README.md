# AccuKnox SBOM Action

GitHub Action that downloads the [AccuKnox ASPM scanner](https://github.com/accuknox/aspm-scanner-cli) binary, generates a filesystem SBOM (CycloneDX by default) with license enrichment, and uploads it to AccuKnox.

Under the hood it runs:

```bash
accuknox-aspm-scanner tool install --type container   # Trivy, generates the SBOM
accuknox-aspm-scanner tool install --type syft        # used by --enrich-licenses
accuknox-aspm-scanner scan --project-name my-project container \
  --command "filesystem ." --generate-sbom --enrich-licenses
```

`--enrich-licenses` runs Syft *after* Trivy, so both tools must be installed.

## Usage

A ready-to-copy workflow is in [`examples/accuknox-sbom.yml`](examples/accuknox-sbom.yml); setup steps are in [`.github/workflows/README.md`](.github/workflows/README.md).

```yaml
name: AccuKnox SBOM
on:
  push:
    branches: [main]
  pull_request:

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: <owner>/sbom_action@v1
        with:
          project_name: my-project
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `project_name` | yes | | AccuKnox SBOM project name |
| `accuknox_token` | yes | | AccuKnox token |
| `accuknox_endpoint` | yes | | e.g. `cspm.demo.accuknox.com` |
| `accuknox_label` | yes | | AccuKnox label |
| `scan_path` | no | `.` | Directory to scan |
| `scanner_version` | no | `v0.16.4` | aspm-scanner-cli release tag |
| `enrich_licenses` | no | `true` | Add `--enrich-licenses` |
| `sbom_format` | no | `cyclonedx` | `cyclonedx` or `spdx` |
| `soft_fail` | no | `true` | Add `--softfail` |
| `upload_artifact` | no | `true` | Upload the SBOM as a workflow artifact |

## Outputs

| Output | Description |
|---|---|
| `sbom_file` | Path of the generated SBOM on the runner |

Supports Linux x86_64 runners only.
