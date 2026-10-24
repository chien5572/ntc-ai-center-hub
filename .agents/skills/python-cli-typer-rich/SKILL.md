---
name: python-cli-typer-rich
description: >-
  Use this skill when building or updating the `hub` CLI commands, interactive terminal
  interview sessions, live log streaming (`hub logs -f`), `hub attach` TUI, or `hub doctor`
  using Typer, Rich, and Textual.
---

# Python CLI with Typer, Rich & Textual (`hub` CLI)

Guides implementation of the `hub` command-line client (ARCH §3, §11; SPEC US-M1-01, US-M1-03, US-M1-12, US-M1-15).

## When to Activate
- Implementing `hub` commands (`task`, `fix`, `new`, `tasks`, `status`, `logs`, `attach`, `approve`, `reject`, `comment`, `merge`, `auto`, `route`, `prompt`, `doctor`, `setup`, `config`, `audit`, `usage`, `cleanup`).
- Designing terminal tables, timelines, and interactive interview loops.

## Core Design Rules

### 1. Thin Client over Unix Socket (ARCH §1.1, AC-M1-01.3)
- `hub` CLI must NOT access `hub.db` directly or duplicate business logic.
- All operations call `hubd` over `~/.ntc-hub/hubd.sock` using `httpx.AsyncClient(transport=httpx.AsyncHTTPTransport(uds=...))`.
- Read commands (`hub tasks`, `hub status`) must complete in **< 1 second** (X-05).

### 2. Smart Active Task Inference (AC-M1-15.6)
- When `<id>` is omitted on task-scoped commands (`status`, `logs`, `attach`, `approve`, `pause`, `resume`, `report`, `spec`, `plan`):
  - If exactly 1 active task exists, automatically target it and print a header: `[dim]Đang thao tác trên #<id> · <slug>[/dim]`.
  - If 0 or >1 active tasks exist, show a clear message and list active tasks.

### 3. `hub doctor` Output Standard (AC-M1-01.2, AC-M2-01.1, AC-M4-01.2)
- Render `✓` (green) or `✗` (red) for each check:
  1. `hubd` daemon running & socket reachable
  2. Each installed harness CLI version & auth status (`ok | expired | missing`)
  3. GitHub App / PAT permissions
  4. Docker daemon + `ntchub-agent` image availability
  5. Free disk space
  6. Telegram bot admin & `Manage Topics` permission (M2+)
  7. Cloudflare Tunnel & Access policy (M4+)
- Every `✗` **must** print a concrete remediation command (e.g., `→ Chạy: hub login antigravity`).

### 4. Interactive Terminal Interview & `hub attach` (US-M1-03, AC-M1-15.4)
- Handle `SIGINT` (`Ctrl+C`) gracefully: save session state on `hubd` and print: `Phiên phỏng vấn #<id> đã lưu. Chạy 'hub attach <id>' để tiếp tục.` (AC-M1-03.6).
- Support `--spec <file>` on `hub task` / `hub fix` to validate the 8 required sections of `spec.md` (§2.3) and jump straight to the `spec` gate (AC-M1-03.8).
