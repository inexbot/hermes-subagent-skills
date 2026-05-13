# Hermes Subagent Skills

A curated collection of Hermes Agent skills for subagent-driven development — from planning through execution, debugging, and code review.

## Skills Included

| # | Skill | Role | Dependency |
|---|-------|------|------------|
| 1 | **writing-plans** | Write bite-sized implementation plans | — (entry point) |
| 2 | **subagent-driven-development** | Execute plans via delegate_task with active orchestrator oversight | writing-plans |
| 3 | **test-driven-development** | Enforce RED-GREEN-REFACTOR, tests before code | — |
| 4 | **systematic-debugging** | 4-phase root cause debugging | — |
| 5 | **requesting-code-review** | Pre-commit review: security scan, quality gates, auto-fix | — |

## Workflow

```
writing-plans          →  subagent-driven-development
(create plan)             (execute plan with subagents,
                            main model reviews every result,
                            intervenes when stuck,
                            final integration review)
                              │
                              ├── test-driven-development
                              │   (subagents follow TDD)
                              │
                              ├── systematic-debugging
                              │   (diagnose root cause when
                              │    subagents hit bugs)
                              │
                              └── requesting-code-review
                                  (optional: auto-fix loop
                                   for pre-commit verification)
```

## Installation

On any machine running Hermes Agent:

```bash
# Clone the repo
git clone https://github.com/inexbot/hermes-subagent-skills.git /tmp/hermes-subagent-skills

# Copy skills into Hermes skills directory
cp -r /tmp/hermes-subagent-skills/skills/* ~/.hermes/skills/software-development/

# Verify
ls ~/.hermes/skills/software-development/subagent-driven-development/SKILL.md
ls ~/.hermes/skills/software-development/writing-plans/SKILL.md
ls ~/.hermes/skills/software-development/test-driven-development/SKILL.md
ls ~/.hermes/skills/software-development/systematic-debugging/SKILL.md
ls ~/.hermes/skills/software-development/requesting-code-review/SKILL.md

# Clean up
rm -rf /tmp/hermes-subagent-skills
```

Or keep the repo for updates:

```bash
git clone https://github.com/inexbot/hermes-subagent-skills.git ~/hermes-subagent-skills
ln -s ~/hermes-subagent-skills/skills/* ~/.hermes/skills/software-development/
# To update: cd ~/hermes-subagent-skills && git pull
```

## Subagent Configuration (Required for delegate_task)

Subagents spawned by `delegate_task` need a model and provider. If you don't configure this, subagents inherit the main model — which works but may be slow or expensive for worker tasks.

### Option A: Let subagents inherit the main model (zero config)

If your main model is already set up and you're fine with subagents using the same model, no action needed. Skip to Quick Start.

### Option B: Use the same provider, different model

If your main model uses a provider that also offers smaller/faster models (e.g. DeepSeek R1 for main, DeepSeek V3 for subagents):

```yaml
# ~/.hermes/config.yaml
delegation:
  model: deepseek-chat          # smaller/faster model for subagents
  provider: deepseek            # same provider as main (reads DEEPSEEK_API_KEY env var)
  max_concurrent_children: 2
```

⚠️ **Pitfall:** Built-in providers (deepseek, openai, anthropic, minimax-cn, etc.) read API keys from environment variables, NOT from `delegation.api_key` in config. If your env var is already set, this just works.

### Option C: Use a different provider entirely

For a different provider that stores its API key in config (not env var):

```yaml
# ~/.hermes/config.yaml
custom_providers:
  - name: minimax-sub
    base_url: https://api.minimax.chat/v1
    api_key: sk-your-key-here

delegation:
  model: minimax-M2.7
  provider: custom:minimax-sub    # must be "custom:<name>"
  max_concurrent_children: 2
```

⚠️ **Critical format note:** `custom_providers` must be a YAML **list** (each item starts with `- name:`), NOT a dict/map.

### After changing config: restart Hermes Gateway

```bash
# Linux with systemd
systemctl --user restart hermes-gateway

# macOS
launchctl kickstart -k gui/$(id -u)/com.hermes.gateway

# WSL (no systemd)
sudo hermes gateway stop --system
sudo hermes gateway start --system
```

Config changes do NOT take effect until the gateway restarts.

### Verify it works

```bash
# Check current delegation config
hermes config get delegation

# Test with a minimal subagent
# Tell Hermes: "delegate_task a subagent that just reports its model name"
```

### Common issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Subagent uses wrong model | Gateway not restarted after config change | Restart gateway |
| "401 Unauthorized" from subagent | Built-in provider but no env var set | Set the provider's env var, OR use custom_providers |
| api_key in delegation ignored | Built-in providers ignore `delegation.api_key` | Use `custom_providers` instead (see Option C) |
| custom_providers parse error | Wrong YAML format | Must be list (`- name:`), not dict |
| Subagent times out on first task | `max_concurrent_children` too high for provider | Reduce to 1 or 2 |

For detailed configuration reference, see `skills/subagent-driven-development/references/delegation-config.md`.

## Key Concepts

### subagent-driven-development v2.0.0

The core orchestrator skill. Main model is an **active supervisor** with three responsibilities:

1. **Progress Tracking** — detect when subagents get stuck, diagnose root causes, intervene with specific guidance
2. **Per-Task Review** — main model personally reviews every subagent result (spec compliance + code quality)
3. **Final Integration Review** — holistic code review, full test suite, requirements verification

### writing-plans v1.1.0

Creates bite-sized implementation plans (2-5 min per task). Every task includes:
- Exact file paths
- Complete code examples
- Exact commands with expected output
- Verification steps

### test-driven-development v1.1.0

Iron law: NO production code without a failing test first. RED → GREEN → REFACTOR cycle.

### systematic-debugging v1.1.0

Four-phase process: Root Cause → Pattern Analysis → Hypothesis → Implementation. No fixes without root cause investigation.

### requesting-code-review v2.0.0

Automated verification pipeline: static security scan → baseline tests/linting → independent reviewer subagent → auto-fix loop (max 2 cycles) → verified commit.

## Quick Start

Tell Hermes:

```
Load subagent-driven-development skill and implement [feature description]
```

Hermes will:
1. Load `writing-plans` to create a plan
2. Load `subagent-driven-development` to execute it
3. Pull in `test-driven-development` and `systematic-debugging` as needed

## License

MIT
