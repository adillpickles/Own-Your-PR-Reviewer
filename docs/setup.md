# Setup Guide

This guide gets Own Your PR Reviewer running in another repository.

The recommended starting point is the main workflow in `.github/workflows/pr_agent.yml`. The files in `examples/` are references for customization, not extra required files.

## 1. Copy The Required Files

Copy these files into the target repository:

```text
.github/workflows/pr_agent.yml
.pr_agent.toml
```

If the `.github/workflows` folder does not exist yet, create it.

## 2. Add An API Key Secret

For the default DeepSeek setup, add this repository secret in GitHub:

```text
DEEPSEEK_API_KEY
```

In GitHub, go to:

```text
Settings -> Secrets and variables -> Actions -> New repository secret
```

Paste your provider API key as the value.

Do not put API keys directly in the workflow or config file.

If you switch away from DeepSeek, use the secret and environment mapping required by PR-Agent and LiteLLM for your provider.

## 3. Optional: Add Control Variables

The workflow works without these variables. If they are unset, the reviewer is enabled and uses balanced mode.

Add repository variables in GitHub:

```text
Settings -> Secrets and variables -> Actions -> Variables -> New repository variable
```

### Pause Or Resume The Reviewer

| Variable | Value | Behavior |
| --- | --- | --- |
| `AI_PR_REVIEW_ENABLED` | unset | Enabled |
| `AI_PR_REVIEW_ENABLED` | `true` | Enabled |
| `AI_PR_REVIEW_ENABLED` | `false` | Disabled |

When this is `false`, the whole job is skipped. That includes automatic PR runs and manual slash commands.

This is useful when:

- Another reviewer is already working.
- You want to pause spend for a while.
- You are testing workflow changes.
- You want to keep the file in place without deleting it.

### Choose Review Intensity

| Variable | Value | Behavior |
| --- | --- | --- |
| `AI_PR_REVIEW_MODE` | unset | Balanced mode |
| `AI_PR_REVIEW_MODE` | `light` | Cheapest normal iteration |
| `AI_PR_REVIEW_MODE` | `balanced` | Default mixed strategy |
| `AI_PR_REVIEW_MODE` | `deep` | Automatic overview only, deeper review through manual commands |

Use lowercase values. `AI_PR_REVIEW_ENABLED=false` and `AI_PR_REVIEW_MODE=light|balanced|deep` should be written exactly in lowercase.
Unknown values fall back close to balanced behavior because the workflow only branches on `light` and `deep`.

## 4. Open A Pull Request

Open a normal, non-draft PR.

The workflow runs on:

- `opened`
- `reopened`
- `ready_for_review`
- `synchronize`

Draft PRs are skipped by default to avoid spending credits too early.

That draft skip applies to automatic `pull_request` runs. Manual slash commands are triggered by
`issue_comment` events, and this template keeps that path simple by running when a PR comment starts
with `/`. Do not use slash commands on draft PRs unless you intentionally want the manual pass.

## 5. Let The Automatic Review Run

On a new PR, the recommended workflow runs:

- A lightweight PR overview.
- Automatic code suggestions through PR-Agent's improve flow.

The default balanced mode uses a cheap automatic model and a stronger manual model.

In deep mode, automatic suggestions are skipped. The overview still runs so the PR has context, but deeper review waits for a manual slash command.

## 6. Use Slash Commands Manually

Comment on the PR with one command on the first line. The comment must start with `/`:

```text
/describe
```

```text
/review
```

```text
/improve
```

```text
/ask What parts of this change need more tests?
```

Manual commands use the stronger model configured in the workflow. This gives you control over when deeper review is worth the extra cost.

If `AI_PR_REVIEW_ENABLED=false`, slash commands are skipped too.

## 7. Understand The Pinned Action Version

The workflow uses PR-Agent like this:

```yaml
uses: Codium-ai/pr-agent@0e37fc84fcc8207561e64eef8f7f634fb57e8447
```

That long value is a specific commit. Pinning to a commit makes the template more stable because the action will not change unexpectedly when PR-Agent releases updates.

You can update it later on purpose by choosing a newer PR-Agent version or commit, testing it on a small PR, and then keeping that newer reference.

Using a floating reference can be convenient, but it is more likely to change behavior without warning.

## 8. Common Checks

If nothing happens:

- Confirm `AI_PR_REVIEW_ENABLED` is not set to `false`.
- Confirm the workflow file is at `.github/workflows/pr_agent.yml`.
- Confirm the PR is not a draft.
- Confirm Actions are enabled for the repository.
- Confirm the repository secret exists.
- Confirm the secret name in GitHub matches the workflow.
- Open the failed Actions run and read the PR-Agent logs.

If provider authentication fails:

- Recreate the API key.
- Re-save the repository secret.
- Check the provider-specific environment variable expected by PR-Agent and LiteLLM.

If reviews are too expensive:

- Use a cheaper automatic model.
- Use `AI_PR_REVIEW_MODE=light`.
- Use `AI_PR_REVIEW_MODE=deep` when you want automatic summaries but manual-only deeper review.
- Keep stronger models for manual slash commands.
- Skip draft PRs.
- Ask targeted `/ask` questions instead of full reviews.
