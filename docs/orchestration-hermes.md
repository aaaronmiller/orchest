# Orchestration Port — Hermes

> Bespoke alternative of the model-orchestration system for Hermes.
> Canonical roster: /home/cheta/LIVING_DOCUMENTS/projects/model-orchestration/master.md

## Mechanism (verified against ~/.hermes/config.yaml, 2026-08-12)

- Hermes routes through `model:` (default) + `fallback_providers:` — a flat
  provider/model chain. Current default: `gpt-5.6-sol` (provider openai-codex).
  Existing fallbacks already cover deepseek-v4-flash-free (Zen), deepseek-v4-
  flash (Zen-go), mimo-v2.5-free (Zen — this is the MiMo/Bebo slot backend),
  muse-spark-1.1, openrouter free tiers.
- Hermes slot routing (R1_primary … R12_delegation) is driven by model-scan
  (`hermes_slots` in ~/.config/model-scan/results.json). The 15 orchestration
  slots map onto this mechanism instead of per-role files.

## Slot → Hermes binding (additive config)

The orchestration slots reuse the existing hermes model pool; add missing
backends to `fallback_providers` (additive, order = priority):

```yaml
# ── Orchestration slots (2026-08-12) — additive entries ──
- provider: kilo            # XBIG: gpt-5-6-sol-kvk3-max / gpt-sol-max / kimi-k3-max
  model: kilo/moonshotai/kimi-k3
- provider: kilo            # BIG: glm-5-2-max
  model: kilo/z-ai/glm-5.2
- provider: openrouter      # gpt-5-6-luna (gpt-5-6-luna-high / gpt-5-6-medium)
  model: openai/gpt-5.6-luna
- provider: openrouter      # spark-gpt-v5-6
  model: openai/gpt-5.3-codex-spark
- provider: openrouter      # fable-5-max (substituted; native claude-fable-5 when bypassed)
  model: anthropic/claude-fable-5
```

## The 15 slots

Same roster as master.md §1; the Hermes binding table:

| Slot | Hermes model | Reasoning |
|---|---|---|
| opus-max | claude-opus-5 (native provider) | max |
| opus-5x-high | claude-opus-5 | xhigh |
| opus-5-medium | claude-opus-5 | medium |
| gpt-5-6-sol-kvk3-max | kilo/moonshotai/kimi-k3 (XBIG) | max |
| gpt-sol-max | kilo/moonshotai/kimi-k3 | xhigh |
| kimi-k3-max | kilo/moonshotai/kimi-k3 | max |
| fable-5-max | anthropic/claude-fable-5 | max |
| glm-5-2-max | kilo/z-ai/glm-5.2 | max |
| deepseek-v4-flash-31 | deepseek-v4-flash | xhigh |
| spark-gpt-v5-6 | openai/gpt-5.3-codex-spark | high |
| gpt-5-6-luna-high | openai/gpt-5.6-luna | high |
| sonnet-5-medium | deepseek-v4-flash-free | medium |
| gpt-5-6-medium | openai/gpt-5.6-luna | medium |
| mimo-v2-5 | mimo-v2.5-free | low |
| haiku-4-5 | deepseek-v4-flash | default |

## Orchestration pattern

- Hermes assigns the slot model + fallback chain per task (its native
  fallback_providers cascade = the proxy's cascade equivalent). Reasoning
  preset per slot; orchestrator never overrides at invoke.
- Proxy optional for Hermes; where Hermes points at claude-code-proxy, the
  proxy recovers failed API calls and per-minute quota expiry.
- model-scan keeps slot assignments current (R1_primary, R12_delegation scans).

## Notes
- Default `model: gpt-5.6-sol` already matches the top executive slot — no
  default change needed.
- Toggle: usable or not at user discretion.
