# Model Strategy

This template is not only about which provider you use. It is also about how much attention the reviewer should spend.

The best default is usually:

1. Cheap automatic review for normal iteration.
2. Stronger manual review when the PR is important.
3. Clear rules for when to spend more.

## One-Word Mode Switching

The recommended workflow supports a repository variable:

```text
AI_PR_REVIEW_MODE=light|balanced|deep
```

If the variable is unset, the workflow behaves as `balanced`.

Use lowercase values: `AI_PR_REVIEW_ENABLED=false` and `AI_PR_REVIEW_MODE=light|balanced|deep` should be written exactly in lowercase.

This is implemented directly in GitHub Actions expressions. It is intentionally simple: the workflow only branches on `light` and `deep`, and anything else behaves close to balanced. That keeps the template from becoming fragile.

## Default Lanes

| Lane | How it runs | Good for | Cost profile |
| --- | --- | --- | --- |
| Light | Automatic overview and suggestions | Small PRs, normal iteration, quick feedback | Lowest |
| Balanced | Stronger model or stricter instructions | Medium PRs, changes with real logic, review before asking humans | Medium |
| Deep | Manual slash-command review | Auth, security, migrations, large refactors, important releases | Highest |

## What The Mode Changes

| Mode | Automatic overview | Automatic suggestions | Manual slash commands | Suggestion settings |
| --- | --- | --- | --- | --- |
| `light` | DeepSeek flash | DeepSeek flash | DeepSeek flash | Fewer suggestions, stricter threshold |
| `balanced` or unset | DeepSeek flash | DeepSeek flash | DeepSeek pro | Moderate suggestions |
| `deep` | DeepSeek flash | Skipped | DeepSeek pro | More room for manual suggestions |

Deep mode is manual-heavy on purpose. It does not turn every PR update into an expensive deep review. It keeps the visible PR overview, then waits for you to ask for `/review`, `/improve`, or `/ask`.

## Light Mode

Light mode is the default automatic lane.

Use it for:

- PR summaries.
- Obvious bug checks.
- Small code suggestions.
- Test gap reminders.
- Fast feedback while you are still iterating.

Default model:

```text
deepseek/deepseek-v4-flash
```

Typical settings:

| Setting | Light-mode idea |
| --- | --- |
| `config.model` | Cheap or fast model |
| `config.reasoning_effort` | Medium or provider-supported fast setting |
| `github_action_config.auto_describe` | `true` |
| `github_action_config.auto_improve` | `true` |
| `num_code_suggestions_per_chunk` | `2` in the recommended workflow |
| `suggestions_score_threshold` | `3` in the recommended workflow |

## Balanced Mode

Balanced mode is for PRs that deserve more than a cheap pass but do not need the most expensive model every time.

Use it for:

- Medium-size feature PRs.
- Data-flow changes.
- Important bug fixes.
- Refactors that touch shared code.
- PRs where the light pass was too shallow.

Ways to create a balanced lane:

- Use the default `AI_PR_REVIEW_MODE=balanced` behavior.
- Keep the model cheap but tighten `.pr_agent.toml` instructions.
- Run `/review` manually with a mid-tier model.
- Increase suggestion quality thresholds so the bot comments less often but more carefully.

Example model patterns:

```text
openai/<balanced-model>
anthropic/<sonnet-style-model>
gemini/<pro-or-balanced-model>
deepseek/deepseek-v4-pro
```

## Deep Mode

Deep mode is manual-heavy in the recommended workflow. The point is to spend extra tokens when a person decides the extra review is worth it.

Use it for:

- Authentication and permissions.
- Security-sensitive code.
- Billing, payments, or account logic.
- Database migrations.
- Large refactors.
- Release-critical changes.
- PRs where you want a second or third pass.

Default manual model:

```text
deepseek/deepseek-v4-pro
```

Manual commands:

```text
/review
/improve
/ask What could break in production?
```

If you want automatic deep review on every PR update, you can edit the workflow and remove the `vars.AI_PR_REVIEW_MODE != 'deep'` guard from the automatic suggestions step. I do not recommend that as the template default because it can surprise users with higher spend.

## Flash, Mini, Pro, and Reasoning Models

Provider names differ, but the pattern is usually similar:

| Model style | Best for | Tradeoff |
| --- | --- | --- |
| Flash, mini, lite, fast | Frequent automatic runs | Cheaper and faster, usually less deep |
| Balanced, sonnet-style, mid-tier | More serious everyday review | Better judgment, still cost-aware |
| Pro, large, opus-style, reasoning | Manual deeper review | More capable, usually slower and more expensive |

## What You Can Tune

You can tune review intensity without changing the whole project:

| Lever | What it changes |
| --- | --- |
| `config.model` | Which model reviews the PR |
| `config.reasoning_effort` | How hard supported models try to reason |
| PR-Agent command | Whether the bot describes, reviews, improves, or answers a question |
| `suggestions_score_threshold` | How selective suggestions should be |
| `num_code_suggestions_per_chunk` | How many suggestions the bot can produce per chunk |
| Automatic versus manual steps | Whether review spends happen on every PR update or only when requested |

Not every provider supports every option. If a setting is ignored by your model, keep the workflow-level strategy: cheap automatic review, stronger manual review.

## Avoid Wasting Credits

- Do not run expensive models on every commit by default.
- Keep draft PRs skipped until work is ready.
- Keep PRs smaller when possible.
- Use `/ask` for focused questions.
- Watch your provider usage dashboard after enabling the workflow.
- Tune `.pr_agent.toml` so the bot does not leave noisy comments.

The goal is not maximum AI usage. The goal is useful review at a price and intensity you control.
