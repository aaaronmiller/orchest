# Model Orchestration — Master File

> Living document. Canonical roster, routing policy, quota data, and per-harness
> orchestration patterns for the multi-model delegation system.
> Generated 2026-08-12. Sources: ardf-pareto-living-document-from-scratch
> (pareto-frontier-cheatsheet.md, data/unified-model-evidence.csv),
> claude-code-proxy .env, model-scan configs.

## 1. The 15 slots (Claude Code agents in ~/.claude/agents)

`frontmatter model` is what the .md file declares. It is also the only thing the
proxy sees, and the proxy routes on the family keyword inside it. `effort` is the
`effort:` frontmatter field (see section 3 for the mechanism).

| # | Agent file | Intended model | IQ (ARDF) | $/task | frontmatter model | effort | Tier reached |
|---|---|---|---|---|---|---|---|
| 1 | opus-max-agent | Claude Opus 5 (max) | 61.13 | 2.34 | claude-opus-5 | max | native, or BIG via proxy |
| 2 | opus-5x-high-agent | Claude Opus 5 (xhigh) | 61.90 | 1.80 | claude-opus-5 | xhigh | native, or BIG via proxy |
| 3 | opus-5-medium-agent | Claude Opus 5 (medium) | 59.00 | 0.72 | claude-opus-5 | medium | native, or BIG via proxy |
| 4 | gpt-5-6-sol-kvk3-max-agent | GPT-5.6 Sol (max) → Kimi K3 | 60.65 | 1.23 | claude-fable-5 | max | proxy → XBIG (fable→XBIG branch added 2026-08-12) |
| 5 | gpt-sol-max-agent | GPT-5.6 Sol (xhigh) | 59.16 | 0.81 | claude-fable-5 | xhigh | proxy → XBIG (fable→XBIG branch added 2026-08-12) |
| 6 | kimi-k3-max-agent | Kimi K3 (max) | 59.87 | 0.84 | claude-fable-5 | max | proxy → XBIG (fable→XBIG branch added 2026-08-12) |
| 7 | fable-5-max-agent | Claude Fable 5 (max) | 60.02 | 3.14 | claude-fable-5 | max | proxy → XBIG (fable→XBIG branch added 2026-08-12) |
| 8 | glm-5-2-max-agent | GLM-5.2 (max) | 51.39 | 0.014744 | claude-opus-4-8 | max | BIG (glm-5.2) |
| 9 | deepseek-v4-flash-31-agent | DeepSeek V4 Flash 0731 (max) | 52.00 | 0.03 | claude-sonnet-5 | max | MIDDLE (deepseek-v4-flash-free) |
| 10 | spark-gpt-v5-6-agent | GPT-5.3 Codex Spark | ~50 [verify] | 0.03 [verify] | claude-sonnet-5 | high | MIDDLE |
| 11 | gpt-5-6-luna-high-agent | GPT-5.6 Luna (high) | 47.00 | 0.02 | claude-sonnet-5 | high | MIDDLE |
| 12 | sonnet-5-medium-agent | Claude Sonnet 5 (medium) | ~50 [verify] | 0.09 [verify] | claude-sonnet-5 | medium | MIDDLE |
| 13 | gpt-5-6-medium-agent | GPT-5.6 Luna (medium) | 39.00 | 0.01 | claude-sonnet-5 | medium | MIDDLE |
| 14 | mimo-v2-5-agent | MiMo-V2.5 (Xiaomi) | 41.96 | 0.011033 | claude-haiku-4-5 | none | SMALL |
| 15 | haiku-4-5-agent | Claude Haiku 4.5 | ~40 [verify] | 0.002 [verify] | claude-haiku-4-5 | none | SMALL |

Slots 14 and 15 declare no `effort` field because Claude Haiku 4.5 has no effort
parameter. Slots 10, 12 and 15 have no row in the ARDF cheatsheet, so their IQ
and price figures are estimates, not calibrated evidence.

Backend collapse: the 15 slots reach at most 4 distinct backends, one per tier.
Slots 1-3 and 8 all land on BIG; slots 9-13 all land on MIDDLE; 14-15 on SMALL;
4-7 land nowhere. Slot identity is a routing label, not a distinct model.

Pareto note (corrected against pareto-frontier-cheatsheet.md): **2** Anthropic
configs sit on the strict frontier — Opus 5 medium and Opus 5 xhigh. Opus 5 max
is near-frontier, dominated by xhigh at the same AA score (63) and lower task
cost, and the cheatsheet hard-prunes it from general default routing. GPT-5.5
(high) is **on** the strict frontier at 58.55/$0.70097; it is not slotted here
because no provider route is configured, not because it was pruned. Fable 5 max
is price-heavy (155.3% premium over Sol max) — prefer cheaper frontier-equivalent
slots.

## 2. Quota posture (researched 2026-08-12; sources in parentheses)

Subscriptions: Claude Code $20/mo, Codex $20/mo, Antigravity $20/mo, OpenCode
Go $10/mo. Everything else free tier. Proxy cascades recover 429/per-minute
quota expiry; no permanent free tier exists on several providers.

| Provider / tier | Limit | Source |
|---|---|---|
| Claude Code subscription | No RPM/ITPM — 5-hour rolling window + weekly cap; shared buckets with claude.ai/Cowork; Pro/Max limits doubled 2026-05-06 | claude.com / anthropic.com |
| Claude API (if used directly) | Tier-based RPM/ITPM/OTPM; Tier 1 ITPM raised to 500k (2026-05) | claude.com |
| OpenRouter free (`:free`) | **20 RPM** all free models; **50 req/day** (1,000/day after $10+ lifetime credits); 429 on exceed | openrouter.ai |
| DeepSeek (direct API) | No RPM/TPM — **concurrency**: V4-Pro 500, V4-Flash 2,500 concurrent; no permanent free tier | deepseek.com |
| Z.ai GLM (Lite/Pro/Max) | Dynamic, tier-based; community-reports of undocumented low concurrency + 429s even in-quota; verify at z.ai/manage-apikey/rate-limits | z.ai |
| Moonshot Kimi | **No free tier**; min ~$1 top-up; spend-based tiers (0-5) | moonshot docs |
| NVIDIA NIM free | ~**40 RPM** prototyping; ~1,000 inference credits/mo; no free upgrades | build.nvidia.com |
| Nous (local .env comment) | 50 RPM / 500,000 TPM | claude-code-proxy/.env |
| Antigravity free tier | [verify] — covered by $20 sub | — |
| OpenCode Zen free | [verify] — free models share account-level caps | opencode.ai |

## 3. Mechanism (verified against Claude Code 2.1.220)

Frontmatter fields Claude Code actually reads: `name`, `description`, `tools`,
`disallowedTools`, `model`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`,
`skills`, `initialPrompt`, `memory`, `effort`, `background`, `isolation`,
`color`.

- **Effort is `effort:`, not `reasoningEffort:`.** `reasoningEffort` is not a
  field; unknown keys load without error and do nothing. Values: `low`, `medium`,
  `high`, `xhigh`, `max`. Default is inherited from the session. There is no
  `default` value.
- Effort is model-dependent. `xhigh` needs Fable 5, Mythos 5, Opus 5, Opus 4.8,
  Opus 4.7 or Sonnet 5. `max` adds Opus 4.6, Sonnet 4.6 and Mythos Preview.
  Sonnet 4.5 and Haiku 4.5 support no effort parameter at all.
- `model:` accepts `sonnet`, `opus`, `haiku`, `fable`, a full model ID, or
  `inherit`. Resolution order: `CLAUDE_CODE_SUBAGENT_MODEL` env var, then the
  Agent tool's per-invocation `model` parameter, then frontmatter, then the main
  conversation's model. A per-invocation `model` override silently defeats the
  slot roster — never pass one.
- `tools:` is a strict allowlist of exact tool names. Delegation is `Agent`, not
  `Task`; todos are `TodoWrite`, not `Todo`. `Workflow`, `AskUserQuestion` and
  `ScheduleWakeup` are stripped from every subagent regardless of the list.
  Background subagents (the default) keep only a reduced built-in set.

## 3b. Routing policy

- Substitution is **session-scoped, not per-agent**. It applies only when
  `ANTHROPIC_BASE_URL` points at the proxy (`cproxy-init` →
  `http://127.0.0.1:8082/v1`). Bypassed, all 15 slots run their real Anthropic
  frontmatter model.
- With the proxy in path, `map_claude_model_to_openai`
  (`src/core/model_manager.py`) matches the family keyword in the requested
  model name: `haiku`→SMALL, `sonnet`→MIDDLE, `opus`→BIG, `fable`→XBIG (branch
  added 2026-08-12, unit-verified), anything else passes through unchanged.
- The `opus` rule also catches the three "native" Opus slots. They are native
  only with the proxy bypassed.
- Per-slot effort does not survive the proxy. `_get_tier_reasoning_override`
  applies `<TIER>_MODEL_REASONING` after mapping: XBIG=high, BIG=high,
  MIDDLE=xhigh, SMALL=medium. Four reasoning levels total, one per tier.
- The proxy runs the cascade on failure and trips circuit breakers on hard
  failures.
- Orchestrator never changes an agent's effort; it selects the agent.
- Heavy bursts → substituted slots; native Opus reserved for high-value work.
- **XBIG primary pinned 2026-08-12**: kimi-k3 (strict frontier, $0.84) — not
  gpt-5.6-sol ($1.23); the sol-max intent rides the KVK3 chain via the slot map.

## 4. Per-harness patterns

### Claude Code (primary)
15 subagent .md files in ~/.claude/agents + orchestrator-agent.md. model +
reasoningEffort in frontmatter. Workflows (not teams). Headless CLI delegation
to low-memory harnesses for bulk/quota-sensitive work.

### Codex
Port doc: /home/cheta/code/agents/orchestration-codex.md. No config-file agents
in codex-cli 0.146 (--agent flag absent, verified 2026-08-12); delegate via
`codex exec -m <model>` with reasoning preset by `-c` at deployment. Shared
agents repo symlinked at ~/.codex/agents.

### OMP (oh-my-pi)
Port doc: /home/cheta/code/agents/orchestration-omp.md. Same markdown-frontmatter
agents; model preset per slot; headless `omp -p` for bulk (see HEADLESS-DELEGATION.md).
Subagent frontmatter with model selection (see agent-development skill).
Reasoning levels preset per slot. Native subagent mechanism.

### Hermes
Port doc: /home/cheta/code/agents/orchestration-hermes.md. Default model gpt-5.6-sol
already matches the top slot; 4 additive fallback_providers entries applied to
~/.hermes/config.yaml (2026-08-12, yaml parse verified); slot routing via
model-scan R1..R12.
Hermes slot routing (R1_primary … R12_delegation) maps onto the same roster;
model-scan feeds slot assignments. Proxy optional.

## 5. Toggle
System under test: usable or not at user discretion. Plain harness behavior
when orchestration is not invoked.

## 6. Gaps / next
- Per-minute RPM numbers per provider: verify from provider docs (web) and
  record here.
- Codex/OMP/Hermes ports built (2026-08-12): orchestration-{codex,omp,hermes}.md in /home/cheta/code/agents/ + HEADLESS-DELEGATION.md; hermes fallback entries applied.
- Headless CLI delegation (jcode/ante/omp): pattern defined, testing pending.
