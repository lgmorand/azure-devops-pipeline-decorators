# A Gitleaks decorator

This private Azure Pipelines extension injects Gitleaks after checkout when
`gitleaks.enabled` is set to `'true'` in the pipeline YAML.

It scans checked-out files, not Git history, on Linux x64 or ARM64 agents.
Detected secrets fail the job, and secret values are redacted in the logs.
Other pipelines are unchanged.
