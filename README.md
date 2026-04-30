# DataLinter-Action
GitHub Action for the DataLinter

Sample Pull Request DataLinter integration
```yml
name: DataLinter PULL_REQUEST run
env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
  PROJECT_DIR_NAME: "target_R_project"
on:
  pull_request:
    branches:
      - main
jobs:
  build:
    permissions:
      #for comment creation and comment update
      pull-requests: write 
      issues: write
    name: DataLinter Docker ${{ github.event_name }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout current project repository
        uses: actions/checkout@v6
        with:
          path: ${{ env.PROJECT_DIR_NAME }}
      - id: datalinter
        name: Run DataLinter - with my custom action
        uses: OxoaResearch/DataLinter-Action@v1
        with:
          data-path: 'test/data/imbalanced_data.csv'
          code-path: 'test/code/r_snippet_imbalanced.r'
          config-path: 'config/r_modelling_config.toml'
          log-level: 'warning'
```
