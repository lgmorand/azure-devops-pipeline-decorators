# A Gitleaks decorator

This example uses Gitleaks 8.30.1 without requiring a separate Marketplace task.
It downloads the official binary, verifies its release SHA-256 checksum, and scans
`$(Build.SourcesDirectory)` immediately after checkout.

## Package and install

From this directory, package the extension:

```powershell
npx --yes --package tfx-cli tfx extension create --manifest-globs vss-extension.json
```

Change `publisher` in `vss-extension.json` to your Azure DevOps Marketplace
publisher before packaging. Upload the resulting VSIX as a **private** extension,
share it with your test organization, and install it there. Pipeline decorators
must be private extensions.

Increment the manifest version when uploading an updated package.

## Test in Azure Pipelines

Create an Azure DevOps YAML pipeline using `azure-pipelines-test.yml` in this
directory. No explicit Gitleaks task is needed: the installed extension injects
`Run Gitleaks (Injected)` after checkout.

The pipeline must define `gitleaks.enabled: 'true'` in YAML before execution;
setting it from a script is too late. Omit the variable or set it to `'false'`
to leave a pipeline unchanged.

Use a Linux agent (x64 or ARM64) with Bash, curl, tar, sha256sum, and outbound
HTTPS access to GitHub releases. The example uses `ubuntu-latest`.
Enabling the decorator on another operating system or architecture fails the job.

The scan covers the current checked-out files, including all repositories under
`$(Build.SourcesDirectory)` for a multi-repository checkout, but not Git history.
Keep checkout enabled. Gitleaks can use a `.gitleaks.toml` configuration in the
sources directory. Findings return exit code 1 and prevent subsequent normal
steps from running; download and checksum failures also fail the job.
Secret values are redacted in logs. This example does not publish a report.

To exercise the failure path, use a disposable test repository with a synthetic,
non-functional secret matching a Gitleaks rule. Never use a real credential.
