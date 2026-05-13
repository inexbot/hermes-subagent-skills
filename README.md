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
