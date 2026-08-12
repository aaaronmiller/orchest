# Orchestration Port — Codex CLI

> Bespoke alternative of the model-orchestration system for OpenAI Codex CLI.
> Canonical roster: /home/cheta/LIVING_DOCUMENTS/projects/model-orchestration/master.md
> Verified against codex-cli 0.146.0 on 2026-08-12.

## Mechanism (verified)

- **No config-file agents**: `codex exec --agent <name>` does not exist in
  0.146.0 (verified: "unexpected argument '--agent'"). The `[agent.*]` config
  sections some Codex versions advertise are not exposed here. Do not build the
  port on that assumption.
- Codex delegates via headless invocations: `codex exec -m <model> "prompt"`.
  Model is set with `-m/--model`; other config via `-c key=value` overrides.
- `~/.codex/agents` is a symlink to the shared agents repo
  (`/home/cheta/code/agents/agents`), so the 15 slot descriptions are available
  as reference on the Codex side too.

## The 15 slots for Codex

Same roster as master.md §1. The slot-to-model binding happens at invocation,
not in files:

| Slot | Codex model (`-m`) | Reasoning (config) |
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

- The orchestrator (Claude Code) delegates to Codex headlessly when the task is
  bulk, parallelizable, or quota-sensitive:
  `codex exec -m <model> -c reasoning_effort=<level> "<leaf prompt>"`.
- Reasoning preset at deployment via `-c`, so the orchestrator never spawns a
  suboptimal combination.
- Proxy not required for routing (Codex talks to its own backends). Keep
  claude-code-proxy in path only where Codex points at it; it still recovers
  failed API calls and per-minute quota expiry.
- Cost: Codex $20/mo subscription tier.

## Notes
- Codex reasoning override key verified via `-c` config surface (`-c model=...`
  documented in help); `reasoning_effort` uses Codex's config key.
- If a future Codex version adds `[agent.*]`, port the 15 slots as config
  entries with description + model + prompt; the roster table above is the
  source.
