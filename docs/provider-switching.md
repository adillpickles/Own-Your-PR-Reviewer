# Provider Switching

This template starts with DeepSeek, but it is not a DeepSeek project. It is a PR-Agent workflow that can call models through LiteLLM.

Switching providers usually means changing:

1. The model strings in `.github/workflows/pr_agent.yml`.
2. The API key secret in GitHub.
3. The workflow environment mapping for that provider.
4. Any provider-specific PR-Agent or LiteLLM settings.

Model names and environment variables change often. The tables below use placeholders unless the repo already has a working default. Before publishing your own final workflow, verify exact model strings and env vars with current PR-Agent and LiteLLM provider docs.

## Provider Table

| Provider or family | Typical reason to use it | Cheap or fast lane example | Strong or deep lane example | Model string pattern | Secret or env mapping pattern | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| DeepSeek | Very low cost default for frequent review | `deepseek/deepseek-v4-flash` | `deepseek/deepseek-v4-pro` | `deepseek/<model>` | `DEEPSEEK.KEY: ${{ secrets.DEEPSEEK_API_KEY }}` | This repo's default working setup. |
| OpenAI / GPT | Strong general coding behavior and broad ecosystem support | `openai/<fast-model>` | `openai/<strong-model>` | `openai/<model>` | Replace with OpenAI mapping expected by PR-Agent/LiteLLM | Verify current model names and key env vars. |
| Anthropic Claude | Strong review style and long-context reasoning | `anthropic/<sonnet-or-fast-model>` | `anthropic/<sonnet-or-opus-model>` | `anthropic/<model>` | Replace with Anthropic mapping expected by PR-Agent/LiteLLM | Good candidate for manual deep review. |
| Google Gemini | Fast models, large context options, Google ecosystem fit | `gemini/<flash-model>` | `gemini/<pro-model>` | `gemini/<model>` | Replace with Gemini mapping expected by PR-Agent/LiteLLM | Check whether your chosen model supports the options you set. |
| Kimi / Moonshot | Long-context oriented models and competitive pricing | `moonshot/<kimi-fast-model>` | `moonshot/<kimi-strong-model>` | `moonshot/<model>` | Replace with Moonshot/Kimi mapping expected by PR-Agent/LiteLLM | Naming may appear as Moonshot in LiteLLM. |
| Qwen | Coding-focused open model family with hosted options | `qwen/<fast-coding-model>` | `qwen/<strong-coding-model>` | `qwen/<model>` or provider-specific pattern | Replace with Qwen provider mapping expected by PR-Agent/LiteLLM | Provider prefix can depend on the route you use. |
| MiniMax | Alternative hosted model provider | `minimax/<fast-model>` | `minimax/<strong-model>` | `minimax/<model>` | Replace with MiniMax mapping expected by PR-Agent/LiteLLM | Verify current LiteLLM support and model names. |
| Local or self-hosted OpenAI-compatible endpoint | Maximum control, private infrastructure, local experiments | `openai/<local-fast-model>` | `openai/<local-strong-model>` | Often `openai/<model>` with custom API base | Provider-specific API base plus key mapping | Usually needs extra base URL configuration. Test with a tiny PR first. |
| Other LiteLLM providers | Fit your own cost, region, or policy needs | `provider/<fast-model>` | `provider/<strong-model>` | `provider/<model>` | Provider-specific mapping | Keep placeholders until you verify docs. |

## What To Change In The Workflow

The default DeepSeek block looks like this:

```yaml
DEEPSEEK.KEY: ${{ secrets.DEEPSEEK_API_KEY }}
config.fallback_models: '["deepseek/deepseek-v4-flash"]'
```

And the automatic model looks like this:

```yaml
config.model: "deepseek/deepseek-v4-flash"
```

Manual slash commands use:

```yaml
config.model: "deepseek/deepseek-v4-pro"
config.fallback_models: '["deepseek/deepseek-v4-pro"]'
```

For another provider, replace those with the provider's model strings and secret mapping.

## Suggested Switching Process

1. Pick a cheap or fast model for automatic review.
2. Pick a stronger model for manual slash commands.
3. Add the provider API key as a GitHub Actions secret.
4. Replace the workflow env mapping with the provider mapping from PR-Agent/LiteLLM docs.
5. Update `config.model` in the automatic steps.
6. Update `config.model` and `config.fallback_models` in the manual step.
7. Open a tiny test PR.
8. Check the Actions logs for auth, model-name, and token-limit errors.

## Example Provider-Flexible Shape

This is intentionally not a copy/paste final config because providers differ:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Replace with the provider-specific mapping required by PR-Agent and LiteLLM.
  PROVIDER.KEY: ${{ secrets.YOUR_PROVIDER_API_KEY }}

  config.fallback_models: '["provider/cheap-or-fast-model"]'

steps:
  - name: Automatic cheap review
    uses: Codium-ai/pr-agent@0e37fc84fcc8207561e64eef8f7f634fb57e8447
    env:
      config.model: "provider/cheap-or-fast-model"

  - name: Manual stronger review
    uses: Codium-ai/pr-agent@0e37fc84fcc8207561e64eef8f7f634fb57e8447
    env:
      config.model: "provider/stronger-review-model"
```

The real workflow should use the actual provider key mapping, not the placeholder.
