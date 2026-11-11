# Workflows

| File | Purpose |
|---|---|
| `test.yml` | Self-test: runs this action (`uses: ./`) against this repository on every push/PR to `main`. |

## Using the action in your own repository

1. Copy [`examples/accuknox-sbom.yml`](../../examples/accuknox-sbom.yml) into your repository as `.github/workflows/accuknox-sbom.yml`.
2. In your repository go to **Settings → Secrets and variables → Actions** and add:
   - `ACCUKNOX_TOKEN` – token from AccuKnox **Settings → Tokens**
   - `ACCUKNOX_ENDPOINT` – e.g. `cspm.demo.accuknox.com`
   - `ACCUKNOX_LABEL` – label from AccuKnox **Settings → Labels**
3. Change `project_name: my-project` to your SBOM project name.
4. Push to `main` (or open a PR, or run it manually from the **Actions** tab).

The workflow:

1. Downloads `accuknox-aspm-scanner` (Linux amd64) from the aspm-scanner-cli releases.
2. Runs `accuknox-aspm-scanner tool install --type container` and `--type syft`.
3. Runs
   ```bash
   accuknox-aspm-scanner scan --project-name my-project container \
     --command "filesystem ." --generate-sbom --enrich-licenses
   ```
   and uploads the SBOM to AccuKnox.
4. Saves the SBOM as the workflow artifact `sbom-<project_name>`.

Results appear in AccuKnox under **Issues → SBOM** for the project name you set.

> For Java/Node/etc. projects, uncomment the build step in the example so the SBOM includes resolved dependencies, not only the manifest.

To run the self-test in this repository, add the same three secrets here as well.
