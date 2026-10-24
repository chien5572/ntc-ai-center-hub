---
name: docker-container-security
description: >-
  Use this skill when building or auditing the `ntchub-agent` sandbox Docker image,
  container execution runner, volume mounts, resource limits, or M5 Docker preview containers.
---

# Docker Container Isolation & Sandbox Security (`ntchub-agent` & Preview)

Enforces the mandatory container isolation rules defined in ARCH §6.2, §13, §15 and SPEC US-M1-05 🛡, US-M5-01 🛡, X-07.

## When to Activate
- Authoring or updating the `ntchub-agent` Dockerfile (which bundles `claude`, `codex`, `agy`, `dsh` CLIs).
- Implementing the container runner in `HarnessAdapter.run()`.
- Implementing Docker preview builds and container lifecycle in `DeployService` (`US-M5-01`).

## Mandatory Security Rules (🛡 Anti-Goal #1 & §15)

### 1. Strict Volume Mount Whitelist for `ntchub-agent` (🛡 AC-M1-05.2, AC-M1-05.3)
Each `HarnessAdapter.run()` invocation MUST spawn an ephemeral container (`--rm`) mounting **ONLY**:
1. **Task Worktree**: `-v ~/.ntc-hub/work/<task-id>:/workspace:rw` (or `:ro` when `mode == "read_only"` such as `interview` or `review` stage).
2. **Single Harness Credential Directory**: Mount ONLY the credential folder of the **specific harness** being invoked (e.g., `~/.ntc-hub/creds/claude:/home/agent/.claude` or `~/.ntc-hub/creds/antigravity:/home/agent/.gemini/antigravity`).

**NEVER mount or expose**:
- `~/.ntc-hub/hub.db`, `~/.ntc-hub/secrets.enc`, `~/.ntc-hub/hubd.sock`
- `~/.ssh`, host git credential helpers, or `/var/run/docker.sock`
- `GITHUB_TOKEN`, `TELEGRAM_BOT_TOKEN`, `VERCEL_TOKEN`, `CLOUDFLARE_API_TOKEN` in container `env`.

### 2. API Key Isolation for DeepSeek Harness (`dsh`) (🛡 AC-M0-03.2, AC-M3-03.2)
- Third-party LLM API keys (`DEEPSEEK_API_KEY`, `OPENAI_API_KEY`, etc.) may **ONLY** be injected via environment variables into containers running `dsh`.
- Never write API keys to files inside `/workspace` (the task worktree).
- Reject any routing configuration that assigns an API key to `claude-code`, `codex`, or `antigravity`.

### 3. Resource Limits, Timeouts & Orphan Cleanup (AC-M1-05.5, AC-M1-13.3, X-07)
- Always pass configurable CPU and RAM limits (`--cpus`, `--memory`, `--pids-limit 256`).
- Label every agent container with `--label ntchub.managed=true --label ntchub.task_id=<id> --label ntchub.stage=<stage>`.
- On `timeout_s` expiry or `hub cancel`: kill container immediately (`docker rm -f <container_id>`).
- On `hubd` startup: find and remove all orphaned containers matching `label=ntchub.managed=true` (`AC-M1-13.3`).

### 4. M5 Preview Container Security (🛡 AC-M5-01.1, AC-M5-01.3)
- Build preview images from the task branch worktree with CPU/RAM limits.
- Never mount `~/.ntc-hub` or `/var/run/docker.sock` into preview containers.
- Bind preview ports only to internal Docker network / localhost routed through internal Caddy + Cloudflare Tunnel (`*.preview.<domain>`).
