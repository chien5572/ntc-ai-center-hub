---
name: ntchub-harness-adapters
description: >-
  Use this skill when implementing or debugging the M0 harness verification scripts or
  the HarnessAdapter implementations (`ARCH §6`, `SPEC M0, US-M1-05, US-M3-02, US-M3-03`)
  for Claude Code (`claude`), Codex CLI (`codex`), Antigravity CLI (`agy`), and DeepSeek Harness (`dsh`).
---

# NTC Hub Harness Adapters (`claude`, `codex`, `agy`, `dsh`)

Guides implementation of the unified `HarnessAdapter` protocol and headless CLI wrappers (ARCH §6; SPEC M0, US-M1-05 🛡, US-M3-02, US-M3-03 🛡, US-M3-04, US-M3-05).

## When to Activate
- Executing M0 spike tasks (`docs/harness-matrix.md`, `scripts/smoke-<harness>.sh`, `tests/fixtures/<harness>/`).
- Implementing `ClaudeCodeAdapter`, `AntigravityAdapter`, `CodexAdapter`, `DeepSeekHarnessAdapter`, or `FakeHarness`.
- Parsing streaming JSON / stdout into `AgentEvent` (`TextDelta`, `ToolCall`, `ToolResult`, `Usage`, `RateLimited`, `Error`, `Done`).
- Handling OAuth expiration (`block_reason="auth_expired"`), quota rate-limits (`block_reason="rate_limit"`), and route fallbacks.

## Core Adapter Rules

### 1. Official Headless CLI Only — Zero Agent SDKs (🛡 FR-15a, AC-M1-05.1)
- **Forbidden**: Do NOT import `anthropic`, `openai`, or Google agent SDKs in `hubd`.
- **Required**: Invoke the official CLI binaries inside the `ntchub-agent` container via `asyncio.create_subprocess_exec`:
  - **Claude Code**: `claude -p "<prompt>" --model <m> --output-format stream-json` (+ `--resume <session_id>`)
  - **Antigravity**: `agy -p "<prompt>" --model <m> --output-format stream-json`
  - **Codex CLI**: `codex exec --json --model <m> "<prompt>"`
  - **DeepSeek Harness**: `dsh --profile ntchub "<prompt>"`

### 2. Auth vs. API Key Separation (🛡 FR-15, AC-M3-03.2)
- `claude`, `codex`, and `antigravity` authenticate **exclusively via OAuth subscription credentials** mounted into the container for user `ntchub`.
- API keys (`DEEPSEEK_API_KEY`, etc.) are **exclusively** used by `dsh` (`DeepSeekHarnessAdapter`) via environment variables inside the `dsh` container.
- When OAuth expires in headless mode, the adapter must detect auth failure within **≤ 30 seconds** (never hang waiting for interactive browser input — `AC-M0-02.3`).

### 3. Rate-Limit & Auth Expiry State Mapping (SPEC §2.1, D-05, US-M3-04, US-M3-05)
- When an adapter emits `RateLimited(retry_after)` or `AuthExpired`:
  1. **If route has `fallbacks` enabled and available**: switch to the next `(harness, model)` fallback and notify once (`⚠️ <harness> ... → chuyển sang dự phòng <harness · model>`).
  2. **If no fallback available on Rate-Limit**: set `HARNESS.rate_limited_until`, set `TASK.status = 'rate_limited'`, `TASK.block_reason = 'rate_limit'`, `TASK.blocked_harness = ...`, release Scheduler slot, and auto-requeue when `rate_limited_until` arrives.
  3. **If no fallback available on Auth Expiry (`D-05`)**: set `TASK.status = 'rate_limited'`, `TASK.block_reason = 'auth_expired'`, `TASK.blocked_harness = ...`, release Scheduler slot, prompt `/login <harness>`, and auto-requeue all tasks blocked on that harness once `/login` succeeds (`AC-M2-08.3`).
