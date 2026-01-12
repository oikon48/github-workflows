# GitHub Workflows

Reusable GitHub Actions workflows for documentation and CI/CD automation.

## Available Workflows

### doc-update-suggestion

Analyzes PR code changes and suggests documentation updates using Claude Code.

**Usage:**

```yaml
name: Doc Update Suggestion

on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - "src/**"

jobs:
  suggest-docs:
    uses: oikon48/github-workflows/.github/workflows/doc-update-suggestion.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**Inputs:**

| Name | Description | Default |
|------|-------------|---------|
| `target_docs` | Target documentation files (comma-separated) | `README.md,CHANGELOG.md,CLAUDE.md` |
| `timeout_minutes` | Workflow timeout in minutes | `10` |

**Required Secrets:**

| Name | Description |
|------|-------------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code OAuth token (generate with `claude setup-token`) |

## Setup

1. Add `CLAUDE_CODE_OAUTH_TOKEN` to your repository secrets
2. Create a workflow file that calls the reusable workflow
3. Customize `paths` trigger as needed for your project

## License

MIT
