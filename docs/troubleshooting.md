# Troubleshooting

Most people should not have to debug PR-Agent from memory. When the workflow fails, the fastest path is often to copy the failed Actions log and ask an AI assistant to inspect the wiring.

## Paste This Into Codex Or ChatGPT

Copy this prompt, then paste in your workflow YAML and the relevant failed GitHub Actions log.

```text
I am debugging a GitHub Actions workflow that runs PR-Agent for AI pull request review.

Please inspect the workflow YAML and logs. Do not assume the provider config is correct.

Check:
- Whether the workflow file is valid YAML.
- Whether the event type should run this job.
- Whether the PR is a draft.
- Whether AI_PR_REVIEW_ENABLED is set to false.
- Whether AI_PR_REVIEW_MODE is light, balanced, deep, or unset.
- Whether the secret names in the workflow match the repository secrets.
- Whether the provider API key env mapping matches PR-Agent and LiteLLM docs.
- Whether the model strings are valid for the configured provider.
- Whether the failure is provider auth, LiteLLM model lookup, token limit, permissions, or PR-Agent behavior.
- Whether the workflow permissions include contents: read, issues: write, and pull-requests: write.
- Whether manual slash commands are being skipped because the job condition is false.

Explain the likely root cause, the exact lines to change, and one small test PR I can use to verify the fix.
```

Useful things to paste below the prompt:

- `.github/workflows/pr_agent.yml`
- `.pr_agent.toml`
- The failed GitHub Actions log
- The PR event type, such as `pull_request` or `issue_comment`
- Whether the PR was a draft
- Which provider and model string you tried
- The names of secrets and variables, without secret values

## The Action Does Not Run

Check:

- `AI_PR_REVIEW_ENABLED` is not set to `false`.
- The workflow file is at `.github/workflows/pr_agent.yml`.
- GitHub Actions are enabled.
- The PR is not a draft.
- The event is one of `opened`, `reopened`, `ready_for_review`, or `synchronize`.
- The workflow was merged into the branch GitHub uses for Actions.

## Authentication Fails

Check:

- The API key is valid.
- The repository secret exists.
- The secret name in GitHub matches the workflow.
- The workflow maps the secret to the environment variable expected by PR-Agent and LiteLLM.
- The provider account has billing or credits enabled if required.

Never print API keys in logs.

## The Model Is Not Found

Check:

- The model string is spelled correctly.
- The provider prefix is correct for LiteLLM.
- The provider supports that model for your account.
- `config.fallback_models` uses models from the same provider or another configured provider.

## The Review Is Too Noisy

Try:

- Set `AI_PR_REVIEW_MODE=light`.
- Raising `suggestions_score_threshold`.
- Lowering `num_code_suggestions_per_chunk`.
- Tightening the `extra_instructions` in `.pr_agent.toml`.
- Using `/ask` for targeted questions instead of full review passes.

## The Review Misses Important Things

Try:

- Running `/review` manually with the stronger model.
- Set `AI_PR_REVIEW_MODE=deep` and use manual slash commands.
- Asking a targeted `/ask` question.
- Adding project-specific review guidance to `.pr_agent.toml`.
- Keeping PRs smaller so the model has clearer context.

## The Action Costs Too Much

Try:

- Use a cheaper model for automatic runs.
- Set `AI_PR_REVIEW_MODE=light`.
- Set `AI_PR_REVIEW_ENABLED=false` when another AI reviewer is already working.
- Keep the stronger model only for manual slash commands.
- Skip draft PRs.
- Avoid repeated full reviews on tiny follow-up commits.
- Watch your provider dashboard while tuning the workflow.

## Large PRs Behave Poorly

Large diffs may be clipped or chunked. That can make feedback shallow or incomplete.

Try:

- Splitting large changes into smaller PRs.
- Asking focused `/ask` questions.
- Using a stronger long-context model manually.
- Increasing token settings only after checking provider cost and limits.
