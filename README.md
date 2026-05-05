# DataLinter GitHub Action

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub tag (latest)](https://img.shields.io/github/v/tag/OxoaResearch/datalinter-github-action)](https://github.com/OxoaResearch/datalinter-github-action/releases/tag/v2)

A GitHub Action to run **[DataLinter](https://github.com/zgornel/DataLinter)**—a contextual data and code linter for machine-learning and statistical-modelling workflows—inside a reproducible Docker container. Results are automatically posted as a comment on pull requests.

## Table of Contents

- [What is DataLinter?](#what-is-datalinter)
- [Features](#features)
- [Quick Start](#quick-start)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [Workflow Examples](#workflow-examples)
- [How It Works](#how-it-works)
- [DataLinter Configuration](#datalinter-configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## What is DataLinter?

[DataLinter](https://github.com/zgornel/DataLinter) detects potential issues in data files and associated code (R, Python, etc.) used in ML/statistical pipelines.

## Features

- Reproducible execution via a pre-built Docker image (`ghcr.io/zgornel/datalinter-compiled:latest`).
- Automatic mounting of data, code, and configuration directories.
- Full output capture (stdout + stderr) wrapped in a Markdown code block.
- **Automatic PR commenting** (edits previous comment if it exists).
- Composite action—works on any runner that supports Docker.
- Flexible logging levels.

## Quick Start

Add the following workflow to `.github/workflows/datalinter.yml`:

```yaml
name: DataLinter

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true   # Required by the embedded PR comment action on newer runners
  PROJECT_DIR_NAME: "target-project"

on:
  pull_request:
    branches: [ main, master ]
  push:
    branches: [ main, master ]

jobs:
  datalinter:
    name: Run DataLinter
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write   # Needed for automatic PR comments
      # issues: write        # Optional; not currently used

    steps:
      - name: Checkout repository
        uses: actions/checkout@v6
        with:
          path: ${{ env.PROJECT_DIR_NAME }}

      - name: Run DataLinter
        id: datalinter
        uses: zgornel/datalinter-github-action@v2
        with:
          target-project-checkout-dir: ${{ env.PROJECT_DIR_NAME }}
          data-path: "data/my_data.csv"
          code-path: "code/my_analysis.r"
          config-path: "config/datalinter.toml"
          log-level: "warning"
```

### Workflow Examples
 - *Pull-request*, recommended for reviews – see Quick Start above.
 - *Push*-only i.e. logs only, no comment, use
```yml
on:
  push:
    branches: [ main ]
# ... rest of job identical, but remove `pull-requests: write` if desired
```

## Inputs

`datalinter-github-action` has the following inputs:
  - `target-project-checkout-dir` (**required**),  the name of the directory where the repository will be checked out
  - `data-path` (**required**), path to the data relative to `target-project-checkout-dir`
  - `code-path` (**required**), path to code, relative to `target-project-checkout-dir`
  - `config-path` (**required**), path to the `datalinter` `.toml` file configuration, relative to `target-project-checkout-dir`
  - `log-level` (**optional**, default is `'debug'`), logging level; can be `'debug'`, `'info'`, `'warning'` and `'error'`.


## Outputs
Complete DataLinter output (stdout + stderr) formatted as a Markdown code block. You can reference it with `${{ steps.<id>.outputs.result }}` if you need further processing.


### How it works
  1. The action splits the supplied paths into directory and filename components.
  2. It pulls `ghcr.io/zgornel/datalinter-compiled:latest`
  3. Three Docker volumes are mounted:
     - Code directory → `/tmp`
     - Data directory → `/_data`
     - Config directory → `/_config`
  
  4. `datalinter` binary is executed inside the container.
  5. All output is captured and stored in the `result` output.
  6. On `pull_request` events the output is automatically posted (or edited) as a PR comment using `thollander/actions-comment-pull-request@v3`.


## DataLinter Configuration
Create a `datalinter.toml` file in your repository. Full documentation and example configurations are available in the [DataLinter](https://github.com/zgornel/DataLinter) repository.


## Troubleshooting
  - **“No such file or directory”** → Verify the three paths exist relative to the checkout directory.
  - **Docker pull fails** → GitHub runners have Docker; rate-limiting on GHCR is rare but can be mitigated with a personal access token if needed.
  - **PR comment not appearing** → Ensure the job has `pull-requests: write` permission and the workflow is triggered on a PR (not a push from a fork without approval).
  - **Output too verbose** → Set log-level: `'warning'` or `'error'`.
  - **Spaces/special characters in paths** → The action quotes paths internally; it should handle them.


## Reporting Bugs

Please [file an issue](https://github.com/OxoaResearch/datalinter-github-action/issues/new) to report a bug or request a feature.


## License

This project is licensed under the MIT License. See LICENSE for details.
