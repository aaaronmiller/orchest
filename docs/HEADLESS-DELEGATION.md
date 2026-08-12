# Headless CLI Delegation Pattern (dual mode)

> Companion to the model-orchestration system. When to delegate via headless
> CLI sessions instead of in-session subagents, and how.

## Why

- **Quota avoidance**: in-session subagents draw from the orchestrator's
  per-minute model quotas. A headless CLI run with its own model selection
  (e.g. `deepseek-v4-flash` via opencode zen/go) spends a different budget and
  can keep working while the main session idles or parallelizes.
- **Low-memory harnesses**: jcode, ante, omp are explicitly low-memory; driving
  them headless keeps the main session light.
- **Explicit reasoning at deployment**: the headless invocation pins the model
  AND reasoning level at spawn time — the orchestrator cannot accidentally
  produce a suboptimal model/reasoning combination.

## When to use (vs in-session subagent)

| Use headless CLI | Use in-session subagent |
|---|---|
| Bulk, parallelizable, or quota-sensitive leaves | Interactive, context-dependent, needs parent context |
| Harness-to-harness handoffs | Tight loops with shared state |
| Anything deepseek-v4-flash or free-tier can do | High-value work on native Opus slots |

## Invocation shapes (verified 2026-08-12)

- **OMP**: `omp -p "<prompt>" --model <model>` (headless print mode; model via
  `--model` flag, reasoning preset at launch).
- **Codex**: `codex exec -m <model> "<prompt>"` (headless; `-c key=value`
  overrides for reasoning).
- **Claude Code**: `claude -p "<prompt>" --model <model> --effort <level>`.
- **DeepSeek direct (zen/go)**: curl to `https://opencode.ai/zen/v1/chat/
  completions` (or zen/go) with `deepseek-v4-flash` / `deepseek-v4-flash-free`
  and `OPENCODE_API_KEY` / `OPENCODE_GO_API_KEY` — the canonical quota-cheap
  workhorse path.
- **jcode / ante**: use their `-p`/exec equivalents when installed.

## Orchestrator rules

1. Prefer headless delegation for anything on the MIDDLE/SMALL tiers.
2. Set reasoning at deployment; never accept an override that changes it.
3. One headless invocation = one leaf; collect evidence from stdout/artifacts.
4. Keep native Opus slots in-session (interactive high-value work only).
5. If the CLI harness is unavailable, fall back to the proxy-substituted slot
   of the same tier — never to a higher tier than the task requires.

## Toggle
The dual pattern is on only when the orchestrator is invoked. Plain harness
behavior otherwise.
