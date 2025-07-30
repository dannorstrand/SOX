# GitHub SOX Report Automation - Workflow Hand-off

## Overview
This document outlines the operational details of the GitHub Actions workflow located at `.github/workflows/fetch-permissions.yml`. This workflow automates the generation of SOX compliance reports for specified GitHub organizations.

## Workflow: `fetch-permissions.yml`

### Purpose
The workflow runs the .NET console application to audit repository permissions, members, and teams based on a provided configuration. It generates an Excel (XLSX) report and logs, which are then uploaded as a workflow artifact.

### Triggers
The workflow can be initiated in three ways:
1.  **Scheduled:** Runs automatically on a daily schedule at 00:00 UTC (`0 0 * * *`).
2.  **Manual (`workflow_dispatch`):** Can be triggered manually from the GitHub Actions UI.
3.  **Push:** Runs automatically on pushes to the `automation` branch. [Confirm branch name or update if needed]

### Manual Inputs
When running the workflow manually, you can provide the following inputs to override the default behavior:

*   **`config_files`**:
    *   **Description**: A comma-separated list of YAML configuration files to process (e.g., `config/org1.yml,config/org2.yml`).
    *   **Usage**: If you need to run the audit for specific pre-defined configurations. If left blank, the workflow will process all `.yml` files in the `config/` directory.

*   **`org`**:
    *   **Description**: A specific GitHub organization name to process.
    *   **Usage**: Use this for a one-off audit of an organization. **This will override and ignore the `config_files` input.**

*   **`repos`**:
    *   **Description**: A comma-separated list of repository names to audit within the specified `org`.
    *   **Usage**: Must be used in conjunction with the `org` input to narrow the audit scope to specific repositories.

*   **`repos_from_file`**:
    *   **Description**: The path to a file within the repository that contains a list of repositories to audit (one per line).
    *   **Usage**: Must be used in conjunction with the `org` input. This is an alternative to the `repos` input for longer lists of repositories.

### Configuration
The workflow relies on a YAML configuration structure to define the audit targets. While manual inputs can dynamically generate a configuration, the standard approach is to use files in the `config/` directory.

**Example `config/my-org.yml`:**
```yaml
ghesToken: "${{ secrets.GHES_TOKEN }}"
restUrl: "https://github.azc.ext.hp.com/api/v3" # Base URL for the REST API
outputFile: "permissions.xlsx"
errorLogFile: "error_log.txt"
email: "recipient@example.com" # [Confirm recipient email address]

organizations:
  - name: "my-github-org"
    # To audit all repos in the org, omit 'repos' and 'repos_from_file'.
    # To audit specific repos, use one of the following:
    repos: "repo-one,repo-two"
    # repos_from_file: "config/my-org-repos.txt"
```

### Required Secrets
The workflow requires one secret to be configured in the repository's settings (`Settings > Secrets and variables > Actions`).

*   **`GHES_TOKEN`**:
    *   **Description**: A GitHub Personal Access Token (PAT) for a service account with the necessary permissions to read organization and repository data.
    *   **Permissions Required**: `repo`, `read:org`, `admin:org`.
    *   **Setup Guide**: [Link to internal Service Account creation documentation, e.g., Confluence page]

### Outputs (Artifacts)
After a successful run, the workflow uploads a single artifact named `permissions-results`. This zipped artifact contains:
*   One or more Excel (`.xlsx`) report files.
*   One or more text (`.txt`) log files.

You can download this artifact from the summary page of the workflow run.