# Opus Refinement Report — Model Orchestration System

> Refinement pass executed headless with **Claude Opus 5 (xhigh)** against the
> first-pass build (2026-08-12). Findings verified against Claude Code 2.1.220
> and claude-code-proxy source; fixes applied in the same pass.

## 1. Mechanism — verified (was: assumed)

Claude Code subagents in `~/.claude/agents/*.md` control the delegated model
and reasoning, but the field is **`effort:`**, not `reasoningEffort:`
(`reasoningEffort` loads silently and does nothing). Full frontmatter surface
read by Claude Code: `name, description, tools, disallowedTools, model,
permissionMode, mcpServers, hooks, maxTurns, skills, initialPrompt, memory,
effort, background, isolation, color`.

- Effort values: `low | medium | high | xhigh | max` (no `default`).
- Effort is model-dependent: `xhigh` needs Fable 5 / Mythos 5 / Opus 5 / Opus
  4.8 / Opus 4.7 / Sonnet 5; `max` adds Opus 4.6 / Sonnet 4.6 / Mythos
  Preview; **Sonnet 4.5 and Haiku 4.5 support no effort parameter**.
- `model:` accepts `sonnet | opus | haiku | fable`, a full model ID, or
  `inherit`. Resolution order: `CLAUDE_CODE_SUBAGENT_MODEL` env → Agent-tool
  per-invocation `model` param → frontmatter → conversation model. A
  per-invocation `model` override silently defeats the slot roster — never
  pass one.
- `tools:` is a strict allowlist; delegation tool is `Agent` (not `Task`);
  todos are `TodoWrite`. `Workflow`, `AskUserQuestion`, `ScheduleWakeup` are
  stripped from subagents.

## 2. Routing — verified (proxy)

- Substitution is session-scoped: active only when `ANTHROPIC_BASE_URL` points
  at claude-code-proxy (`cproxy-init` → `http://127.0.0.1:8082/v1`). Bypassed,
  all 15 slots run their real Anthropic frontmatter model.
- Family-keyword mapping in `map_claude_model_to_openai`: `haiku`→SMALL,
  `sonnet`→MIDDLE, `opus`→BIG. **`fable` had no branch** — the four fable-named
  top slots would have passed through unchanged. FIXED in the refinement pass:
  `fable`→XBIG added (model_manager.py), unit-verified (fable→X, hybrid
  `fable/openai/gpt-5.6-sol`→`openai/gpt-5.6-sol`, passthrough intact).
- The 3 "native" Opus slots are native **only with the proxy bypassed**; with
  the proxy in path, `opus`→BIG substitutes them. Deployment nuance: keep the
  native slots out of proxy sessions.
- Per-slot effort does not survive the proxy; `<TIER>_MODEL_REASONING` applies
  per tier (XBIG=high, BIG=high, MIDDLE=xhigh, SMALL=medium).

## 3. Changes applied in this pass

| Artifact | Change |
|---|---|
| 15 model agents + orchestrator | `reasoningEffort:` → `effort:` (all); effort values normalized to valid set; `default` removed |
| haiku-4-5, mimo-v2-5 | effort line removed (frontmatter model claude-haiku-4-5 supports no effort) |
| deepseek-v4-flash-31 | effort raised to `max` (cheap enough; matches slot intent) |
| glm-5-2-max | model `claude-opus-4-5` → `claude-opus-4-8` (current opus id; BIG-tier match) |
| orchestrator | tools include `Agent, Skill, TodoWrite` (correct delegation surface); effort `medium` |
| master.md | section 3 "Mechanism (verified)" + section 3b "Routing policy" added; quota section retained |
| claude-code-proxy `src/core/model_manager.py` | `fable`→XBIG branch added (authorized proxy change) |

## 4. Per-harness orchestration — recommendations

- **Claude Code**: current build (15 subagents + orchestrator, `effort:` in
  frontmatter, workflows not teams, headless CLI delegation for bulk work).
  Full harness docs in master.md section 4.
- **Codex**: agents are defined in config (no per-role md files); map the same
  15 slots as config entries with model + effort preset. Proxy optional (Codex
  talks to its own backends), but keep the proxy for failure/quote recovery
  where Codex points at it. [draft — build pending]
- **OMP (oh-my-pi)**: subagent frontmatter with model selection (agent-
  development skill); presets per slot. [draft — build pending]
- **Hermes**: slot routing R1_primary … R12_delegation maps onto the same
  roster; model-scan feeds assignments. Proxy optional. [draft — build pending]

## 5. Open questions [verify]

- Per-minute RPM numbers per provider (Anthropic subscription tier, OpenAI,
  Z.ai, DeepSeek, Moonshot, OpenRouter free, Nous 50 RPM/500k TPM, NVIDIA):
  to be researched and recorded in master.md section 2.
- Whether Claude Code enforces `model:` alias allowlisting when
  ANTHROPIC_BASE_URL points at the proxy (custom model names).
- MiMo-V2.5 vs MiMo-V2.5-Pro: confirm which the Pareto graph intends.
- Proxy XBIG tier currently configured to kimi-k3 as primary — confirm the
  desired XBIG backend for the Sol-max slots (kimi-k3 vs gpt-5.6-sol).
