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

## Prerequisites

Add these secrets in your repository under **Settings → Secrets and variables → Actions**:

| Secret | Description | Example |
|---|---|---|
| `ACCUKNOX_TOKEN` | Token from AccuKnox **Settings → Tokens** | `eyJhbGc...` |
| `ACCUKNOX_ENDPOINT` | AccuKnox endpoint the SBOM is uploaded to | `cspm.demo.accuknox.com` |
| `ACCUKNOX_LABEL` | Label from AccuKnox **Settings → Labels** | `sbom` |

## Usage

Create `.github/workflows/accuknox-sbom.yml` in your repository:

```yaml
name: AccuKnox SBOM Scan Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  sbom-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run AccuKnox SBOM Scan
        uses: VanishaParwal/sbom_action@v1
        with:
          # SBOM project name in AccuKnox
          project_name: "my-project"

          # AccuKnox credentials
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}

          # Optional
          scan_path: "."             # Directory to scan ('filesystem <scan_path>')
          sbom_format: "cyclonedx"   # cyclonedx or spdx
          soft_fail: true            # Continue the pipeline on findings
          upload_artifact: true      # Upload the SBOM as a workflow artifact
          scanner_version: "v0.16.4" # aspm-scanner-cli release tag
```

### Java / Maven projects

Build first so the SBOM contains the resolved dependencies, not just what is declared in `pom.xml`:

```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"
          cache: maven

      - name: Build with Maven
        run: ./mvnw -B -DskipTests package

      - name: Run AccuKnox SBOM Scan
        uses: VanishaParwal/sbom_action@v1
        with:
          project_name: "my-project"
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
```

The same workflow is in [`examples/accuknox-sbom.yml`](examples/accuknox-sbom.yml).

## Viewing results

- **AccuKnox:** the SBOM appears under the SBOM project set in `project_name`, tagged with your label.
- **GitHub:** download the `sbom-<project_name>` artifact from the workflow run summary.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `project_name` | yes | | AccuKnox SBOM project name |
| `accuknox_token` | yes | | AccuKnox token |
| `accuknox_endpoint` | yes | | e.g. `cspm.demo.accuknox.com` |
| `accuknox_label` | yes | | AccuKnox label |
| `scan_path` | no | `.` | Directory to scan |
| `scanner_version` | no | `v0.16.4` | aspm-scanner-cli release tag |
| `sbom_format` | no | `cyclonedx` | `cyclonedx` or `spdx` |
| `soft_fail` | no | `true` | Add `--softfail` |
| `upload_artifact` | no | `true` | Upload the SBOM as a workflow artifact |

## Outputs

| Output | Description |
|---|---|
| `sbom_file` | Path of the generated SBOM on the runner |

Supports Linux x86_64 runners only.
