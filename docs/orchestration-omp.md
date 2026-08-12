# Orchestration Port — OMP (oh-my-pi)

> Bespoke alternative of the model-orchestration system for the OMP harness.
> Canonical roster: /home/cheta/LIVING_DOCUMENTS/projects/model-orchestration/master.md

## Mechanism

- OMP subagents use the same markdown-frontmatter format as Claude Code agents
  (name, description, model, tools; see agent-development skill). The shared
  agent files in /home/cheta/code/agents/agents/ (which ~/.claude/agents
  symlinks to) are the canonical bodies.
- OMP model selection is per-agent via frontmatter `model:`; this harness
  resolves model ids through its own provider config (~/.omp/agent/config.yml)
  and the opencode zen/go endpoints.
- OMP is a low-memory harness — ideal for the headless delegation dual pattern.

## The 15 slots for OMP

Same roster as master.md §1. Agent files: reuse the shared
`*-agent.md` files; the OMP side binds models per harness config:

| Slot | OMP model | Reasoning |
|---|---|---|
| opus-max | claude-opus-5 | max |
| opus-5x-high | claude-opus-5 | xhigh |
| opus-5-medium | claude-opus-5 | medium |
| gpt-5-6-sol-kvk3-max | gpt-5.6-sol | max |
| gpt-sol-max | gpt-5.6-sol | xhigh |
| kimi-k3-max | kimi-k3 | max |
| fable-5-max | claude-fable-5 | max |
| glm-5-2-max | glm-5.2 | max |
| deepseek-v4-flash-31 | deepseek-v4-flash | xhigh |
| spark-gpt-v5-6 | gpt-5.3-codex-spark | high |
| gpt-5-6-luna-high | gpt-5.6-luna | high |
| sonnet-5-medium | claude-sonnet-5 | medium |
| gpt-5-6-medium | gpt-5.6-luna | medium |
| mimo-v2-5 | mimo-v2.5-free | low |
| haiku-4-5 | claude-haiku-4-5 | default |

## Orchestration pattern

- Orchestrator delegates via OMP's native subagent mechanism (Task tool) with
  the model preset in frontmatter — never overridden at invoke time.
- For bulk/quota-sensitive leaves, prefer headless `omp -p <prompt>` (see
  HEADLESS-DELEGATION.md) — OMP is explicitly a low-memory harness.
- Proxy substitution not required for OMP routing; claude-code-proxy still
  recovers failed API calls and per-minute quota expiry wherever OMP traffic
  points at it.

## Notes
- OMP reasoning/effort binding per harness version: verify the frontmatter key
  (`effort:` vs model-suffix) against the installed OMP before relying on it.
- Toggle: usable or not at user discretion, same as Claude Code.
