# Own Your PR Reviewer

**A controllable, provider-flexible AI pull request reviewer for GitHub Actions.**

A self-controlled AI pull request review workflow for GitHub Actions, built with PR-Agent and LiteLLM.

It gives you the useful parts of a Copilot-style PR review loop: a PR overview, reviewed-file context, inline code suggestions, and a second-reviewer feeling while you are building. The difference is that you own the setup. Bring your own API key, pick your own provider, choose the model, choose the cost level, and decide when the bot should run.

This is not a full replacement for GitHub Copilot. It is a practical template for people who want a review partner they can tune and run themselves.

## Why This Exists

AI PR review became part of my development rhythm because it is genuinely useful. When I am moving fast, a quick second pass can catch boring mistakes, missing tests, edge cases, and review questions I should answer before asking another person to look.

The frustrating part is how fragile that workflow can feel when usage limits, rate limits, billing changes, or limited included usage get in the way. Gone are the days where you could always press "add AI review" and expect another useful pass. These days, you can be met with a limit right when you actually want feedback.

This template is my attempt to bring that workflow back under my own control:

- My own provider.
- My own API key.
- My own model choice.
- My own cost and quality tradeoff.
- My own GitHub Actions workflow.
- Cheap models for routine automation.
- Stronger models when I intentionally ask for deeper review.
- A pause switch for times when another reviewer, including GitHub Copilot, is already working.

The default setup uses DeepSeek because it is very inexpensive and works well as a starting point. The project is not DeepSeek-only. It is built around PR-Agent plus LiteLLM, so you can adapt it to OpenAI, Claude, Gemini, Kimi, Qwen, MiniMax, local OpenAI-compatible endpoints, or other LiteLLM-supported providers.

## What It Does

- Adds a lightweight PR overview when a PR is opened, reopened, or marked ready for review.
- Adds automatic inline-style code suggestions on PR updates.
- Gives reviewers changed-file context instead of only a generic summary.
- Supports manual slash commands like `/review`, `/improve`, `/describe`, and `/ask` when the PR comment starts with `/`.
- Uses a cheap automatic model for normal iteration.
- Lets manual commands use a stronger model when the PR deserves deeper attention.
- Can be paused with a repository variable when you do not want AI review running.
- Supports light, balanced, and deep review modes.
- Keeps provider switching visible and editable in the workflow file.

## Who This Is For

This template is for developers who want control first and lower cost second.

It should be useful for:

- Students who want review help without burning through expensive tools.
- Small teams that want predictable PR feedback.
- Side projects where cost matters but quality still matters.
- Open source maintainers who want optional AI review without forcing one ecosystem.
- Indie hackers who want fast iteration and manual deeper review when it counts.
- Developers frustrated by review limits during active build sessions.
- Teams that want provider flexibility instead of being locked into one model vendor.
- Serious projects that want cheap routine review plus premium manual review for risky changes.

## How It Works

The recommended workflow lives at `.github/workflows/pr_agent.yml`.

The review behavior lives in `.pr_agent.toml`.

The default flow is:

| Trigger | Tool behavior | Default model |
| --- | --- | --- |
| PR opened, reopened, or ready for review | Lightweight overview | `deepseek/deepseek-v4-flash` |
| PR opened or updated | Automatic code suggestions | `deepseek/deepseek-v4-flash` |
| PR comment with slash command | Manual review, improve, describe, or ask | `deepseek/deepseek-v4-pro` |

That split matters. Automatic review can run a lot, so it should be cheap and fast. Manual review should be available when you decide the PR needs a stronger pass.

## Quick Start

1. Copy the recommended files into your repository:
   - `.github/workflows/pr_agent.yml`
   - `.pr_agent.toml`
2. Add a provider API key as a repository secret.
   - For the default DeepSeek setup, add `DEEPSEEK_API_KEY`.
3. Optional: add repository variables.
   - `AI_PR_REVIEW_ENABLED=false` pauses the workflow.
   - `AI_PR_REVIEW_MODE=light|balanced|deep` changes review intensity.
4. Open a non-draft pull request.
5. Let GitHub Actions post the automatic overview and suggestions.
6. For a deeper pass, comment on the PR with a slash command.

See [docs/setup.md](docs/setup.md) for the full guide.

If the workflow fails, [docs/troubleshooting.md](docs/troubleshooting.md) includes a copy/paste prompt you can give to Codex or ChatGPT with your Actions log.

## Required Secrets

For the default DeepSeek setup:

| Secret | Used for |
| --- | --- |
| `DEEPSEEK_API_KEY` | API key for DeepSeek through LiteLLM |

`GITHUB_TOKEN` is provided automatically by GitHub Actions.

If you switch providers, the secret name and environment mapping may change. Check the current PR-Agent and LiteLLM docs for provider-specific environment variables.

## Enable Switch

Set this repository variable when you want to pause the reviewer without deleting the workflow:

| Variable | Value | Behavior |
| --- | --- | --- |
| `AI_PR_REVIEW_ENABLED` | unset | Enabled |
| `AI_PR_REVIEW_ENABLED` | `true` | Enabled |
| `AI_PR_REVIEW_ENABLED` | `false` | Disabled |

When disabled, the workflow skips automatic reviews and manual slash commands. I chose a full pause because the main use case is avoiding duplicate AI review when another reviewer is already working.

## Model Strategy

The default mode is `balanced` when `AI_PR_REVIEW_MODE` is not set.

| Mode | Use it for | What changes |
| --- | --- | --- |
| `light` | Cheapest normal iteration | Cheap automatic model, fewer suggestions, stricter suggestion threshold, cheap manual model |
| `balanced` | Default for most projects | Cheap automatic model, stronger manual model, moderate suggestion volume |
| `deep` | Risky PRs and intentional deeper review | Automatic overview stays on, automatic suggestions are skipped, manual commands use the stronger model |

You can tune:

- `config.model`
- `config.reasoning_effort`, when supported by the provider or model
- PR-Agent commands like `/review`, `/improve`, `/describe`, and `/ask`
- `num_code_suggestions_per_chunk`
- `suggestions_score_threshold`
- Whether review runs automatically or only after manual comments

See [docs/model-strategy.md](docs/model-strategy.md).

Model quality and cost move fast. Top-tier GPT, Claude, Gemini, and similar models may produce
better semantic summaries and broader reasoning, but they can cost much more per PR. Cheaper models
can still catch real issues and are often good enough for a frequent first pass. The best setup is
usually to test models on your own PRs, keep automatic review cheap, and save stronger models for
manual passes when the change is risky.

## Provider Flexibility

This project is provider-flexible because PR-Agent can call models through LiteLLM.

The default workflow starts with:

```text
deepseek/deepseek-v4-flash
deepseek/deepseek-v4-pro
```

But the model strings can be changed to patterns like:

```text
openai/<fast-model>
anthropic/<sonnet-or-opus-model>
gemini/<flash-or-pro-model>
moonshot/<kimi-model>
qwen/<qwen-coding-model>
minimax/<minimax-model>
openai/<local-or-openai-compatible-model>
```

Model names and provider env vars change often, so the docs use placeholders unless the repo already has a working default. See [docs/provider-switching.md](docs/provider-switching.md).

## Slash Commands

Comment on a pull request with one of these commands:

| Command | Use it for |
| --- | --- |
| `/describe` | Generate or refresh a PR overview |
| `/review` | Ask for a broader PR review |
| `/improve` | Ask for code suggestions |
| `/ask ...` | Ask a specific question about the PR |

Examples:

```text
/review
```

```text
/ask What edge cases should I test before merging this?
```

The workflow only wakes up for PR comments that begin with `/`, so normal review discussion should
not spend AI credits. Automatic runs skip draft PRs. Manual slash commands are evaluated from
`issue_comment` events, which do not include the full draft-state check in the template job
condition, so avoid using slash commands on draft PRs unless you intentionally want a review pass.

## Examples

The main workflow in `.github/workflows/pr_agent.yml` is the recommended starting point.

The `examples/` folder is there for reference:

| File | Purpose |
| --- | --- |
| `examples/recommended-full-workflow.yml` | A copy of the full recommended workflow with extra orientation comments |
| `examples/deepseek-default.yml` | A smaller DeepSeek-first example |
| `examples/provider-flexible.yml` | A provider-switching skeleton with placeholders |

You do not need to copy every example. Start with the main workflow and `.pr_agent.toml`, then use examples when you want to customize.

## Pinned PR-Agent Version

The workflow pins PR-Agent to a specific commit:

```yaml
uses: Codium-ai/pr-agent@0e37fc84fcc8207561e64eef8f7f634fb57e8447
```

That is intentional. Pinning keeps the template stable and avoids surprise behavior changes if PR-Agent updates. You can update the pinned commit later when you are ready to test a newer version.

Using a floating reference like a branch or latest-style tag can be easier, but it is less stable for a reusable public template.

## Cost-Control Tips

- Keep automatic runs on a cheap or fast model.
- Use stronger models only for manual commands.
- Set `AI_PR_REVIEW_ENABLED=false` when another AI reviewer is already running.
- Use `AI_PR_REVIEW_MODE=deep` when you want manual deeper review without automatic suggestion spend.
- Skip draft PRs until work is ready.
- Keep PRs small enough that review context stays useful.
- Use `/ask` for targeted questions instead of asking for a full review every time.
- Watch provider dashboards when you first enable the workflow.

## Limitations

- AI review can miss bugs.
- AI review can be confidently wrong.
- Large diffs may be clipped or chunked.
- Provider behavior, model names, and environment variables can change.
- Generated suggestions still need human judgment before merging.

Treat this as a useful reviewer, not as a merge gate by itself.

## FAQ

### Is this a GitHub Copilot replacement?

No. It covers a similar PR review habit, but it does not replace the full Copilot product. Think of it as your own configurable review workflow.

### Why DeepSeek by default?

Cost. DeepSeek is a good default when you want frequent automatic feedback without worrying too much about every review attempt. You can switch providers.

### Can I use OpenAI, Claude, Gemini, Kimi, Qwen, or MiniMax?

Yes, as long as PR-Agent and LiteLLM support the provider and model you choose. Update the model strings and provider secret mapping.

### Should automatic reviews use the strongest model?

Usually no. Automatic reviews can run many times. Put the cheaper model on routine automation and save stronger models for intentional manual review.

### Can this run only on manual commands?

Mostly. Set `AI_PR_REVIEW_MODE=deep` to keep the automatic overview but skip automatic suggestions, then use slash commands when you want deeper review. For a strict manual-only setup, remove or disable the automatic overview step too.

### How do I pause it when Copilot is already reviewing?

Set the repository variable `AI_PR_REVIEW_ENABLED=false`. Set it back to `true` or delete the variable when you want this workflow to run again.

### Can I change the review tone?

Yes. Edit `.pr_agent.toml`, especially the `extra_instructions` fields.

## License

MIT. See [LICENSE](LICENSE).
