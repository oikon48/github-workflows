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

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  suggest-docs:
    permissions:
      contents: read
      pull-requests: write
      id-token: write  # Required for OIDC authentication
    uses: oikon48/github-workflows/.github/workflows/doc-update-suggestion.yml@main
    secrets: inherit
```

**Inputs:**

| Name | Description | Default |
|------|-------------|---------|
| `target_docs` | Target documentation files (comma-separated) | `README.md,CHANGELOG.md,CLAUDE.md` |
| `timeout_minutes` | Workflow timeout in minutes | `10` |

**Required Secrets:**

| Name | Description |
|------|-------------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code OAuth token (generate with `claude mcp add-oauth anthropic`) |

## Important: Permissions Requirements

### OIDC Authentication

`claude-code-action@v1` uses OIDC (OpenID Connect) for authentication. This requires `id-token: write` permission.

**Critical:** When using Reusable Workflows, permissions are NOT automatically inherited. You must explicitly set `id-token: write` in your caller workflow:

```yaml
jobs:
  your-job:
    permissions:
      contents: read
      pull-requests: write
      id-token: write  # ← Required! Not inherited from reusable workflow
    uses: oikon48/github-workflows/.github/workflows/doc-update-suggestion.yml@main
    secrets: inherit
```

Reference: [GitHub Blog - OIDC Token Permissions in Reusable Workflows](https://github.blog/changelog/2023-06-15-github-actions-securing-openid-connect-oidc-token-permissions-in-reusable-workflows/)

## Troubleshooting

### `startup_failure` Error

**Symptom:** Workflow fails immediately with `startup_failure` status.

**Cause:** Missing `id-token: write` permission. The OIDC token cannot be fetched.

**Internal error:** `Unable to get ACTIONS_ID_TOKEN_REQUEST_URL env variable`

**Solution:** Add `id-token: write` to your caller workflow's permissions block:

```yaml
jobs:
  suggest-docs:
    permissions:
      contents: read
      pull-requests: write
      id-token: write  # Add this line
    uses: oikon48/github-workflows/.github/workflows/doc-update-suggestion.yml@main
    secrets: inherit
```

### Workflow Not Triggering on New Workflow Files

**Symptom:** When you add a new workflow file in a PR, the workflow doesn't run or fails validation.

**Cause:** Security feature - workflow files must match the default branch (main) version.

**Solution:**
1. Merge the workflow file changes to main first
2. Then create a new PR with code changes to trigger the workflow

## Setup

1. Add `CLAUDE_CODE_OAUTH_TOKEN` to your repository secrets
   ```bash
   claude mcp add-oauth anthropic
   ```
2. Create a workflow file that calls the reusable workflow (see Usage above)
3. Customize `paths` trigger as needed for your project
4. **Ensure `permissions` block includes `id-token: write`**

## Best Practices

- Always use `secrets: inherit` to pass secrets to reusable workflows
- Set `concurrency` with `cancel-in-progress: true` to prevent duplicate runs
- Add `if: github.event.pull_request.user.login != 'github-actions[bot]'` to prevent bot loops
- Use `timeout-minutes` to limit resource consumption

## License

MIT
