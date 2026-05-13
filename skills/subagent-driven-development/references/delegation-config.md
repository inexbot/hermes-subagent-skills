# Delegation Configuration Guide

## How Subagent Model Selection Works

The `delegation` section in `config.yaml` controls which model subagents use:

```yaml
delegation:
  model: ""          # empty = inherit parent model
  provider: ""       # empty = inherit parent provider
  base_url: ""       # empty = inherit parent base_url
  api_key: ""        # empty = inherit parent api_key
  max_concurrent_children: 3
  max_iterations: 50
  child_timeout_seconds: 600
```

When `model` and `provider` are empty, subagents use the same model as the parent agent.

## Using a Different Model for Subagents

### Option A: Built-in Provider (env var required)

For Hermes built-in providers (minimax-cn, kimi-coding, etc.), the API key must be set as an **environment variable** — the `api_key` field in delegation config is NOT used by built-in providers.

Built-in provider env var names (from config.yaml comment block):
- `minimax` → `MINIMAX_API_KEY`
- `minimax-cn` → `MINIMAX_CN_API_KEY`
- `kimi-coding` → `KIMI_API_KEY`
- `kimi-coding-cn` → `KIMI_CN_API_KEY`

Example:
```bash
export MINIMAX_CN_API_KEY="sk-xxx"
```
```yaml
delegation:
  model: minimax-M2.7
  provider: minimax-cn
```

**⚠️ Pitfall**: Setting `api_key` in the delegation config alongside a built-in provider has no effect — the env var is the only path. If you set only `api_key` in config, the provider will fail to authenticate silently, and the subagent will fall back to the parent model.

### Option B: Custom Provider (api_key in config)

To store the API key directly in config (no env var needed), use `custom_providers`:

```yaml
custom_providers:
  my-minimax:
    base_url: https://api.minimax.chat/v1
    api_key: sk-xxx

delegation:
  model: minimax-M2.7
  provider: custom:my-minimax
```

The provider name in delegation must be `custom:<name>` where `<name>` matches the key in `custom_providers`.

## Config Changes Require Gateway Restart

After modifying `config.yaml`, the Hermes Gateway must be restarted for changes to take effect:

```bash
systemctl --user restart hermes-gateway
```

Without a restart, subagents will continue using the previous configuration. Verify by dispatching a minimal test subagent that reports its model name.

## Concurrent Children Tuning

`max_concurrent_children` controls how many subagents can run in parallel. Reduce this if the subagent model provider has rate limits or concurrency restrictions. Value of 2 is a safe default for most providers.
