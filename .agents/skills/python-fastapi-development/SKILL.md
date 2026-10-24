---
name: python-fastapi-development
description: >-
  Use this skill when building or modifying the hubd daemon API layer, REST endpoints,
  WebSocket streaming (/ws), or Unix domain socket server (~/.ntc-hub/hubd.sock)
  using Python 3.12+, FastAPI, Uvicorn, and Pydantic v2.
---

# Python FastAPI Development (`hubd` Daemon & API Layer)

Provides architectural patterns and implementation rules for the `hubd` daemon in `ntc-ai-center-hub` (ARCHITECTURE §2, §3, §11; SPEC M1 & M4).

## When to Activate
- Implementing or updating `hubd` REST endpoints (`/tasks`, `/approvals`, `/projects`, `/config`, `/harnesses`).
- Implementing WebSocket `/ws` event & live log streaming (ARCH §11, US-M4-02).
- Configuring dual-listener Uvicorn server: Unix domain socket (`~/.ntc-hub/hubd.sock` for local `hub` CLI without token) and `127.0.0.1:<port>` (for Webapp via Cloudflare Tunnel).
- Designing Pydantic v2 request/response DTOs and lifespan management for background workers (Orchestrator, Scheduler, Telegram gateway).

## Core Architectural Rules for `ntc-ai-center-hub`

### 1. Binding & Network Security Invariants (🛡 AC-M1-01.3, AC-M4-01.1)
- **Unix Domain Socket (`~/.ntc-hub/hubd.sock`)**: Used by `hub` CLI on the same machine/user (`ntchub`). Permissions must be `0600` owned by `ntchub`.
- **Loopback HTTP (`127.0.0.1:<port>`)**: Used by Webapp and `cloudflared` tunnel. **NEVER** bind `0.0.0.0` in production defaults.
- **Mutating API Security (AC-M4-01.4)**: Every state-changing HTTP request (`POST`, `PUT`, `DELETE`) from Webapp must validate session auth + CSRF token.

### 2. Async Lifespan & Background Services
Manage long-running tasks cleanly via FastAPI's `asynccontextmanager` lifespan:
```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 1. Recover interrupted 'running' tasks & clean orphan containers (AC-M1-13.1, AC-M1-13.3)
    await app.state.orchestrator.recover_on_startup()
    # 2. Start Scheduler loop & Event Bus
    scheduler_task = asyncio.create_task(app.state.scheduler.run_forever())
    yield
    # 3. Graceful shutdown
    scheduler_task.cancel()
    await asyncio.gather(scheduler_task, return_exceptions=True)
```

### 3. In-Process Event Bus & WebSocket Sync (ARCH §2, AC-M4-02.4, X-08)
- Every state change (`TASK`, `STAGE_RUN`, `APPROVAL`) must:
  1. Commit to SQLite `events` table with a monotonic `event_id`.
  2. Publish to the in-process `EventBus` so Telegram gateway, CLI `attach`, and WebSocket clients reflect changes in **≤ 2–3 seconds**.
- Support `?since_event_id=<id>` on `/ws` reconnect so Webapp clients replay missed events without state drift.

### 4. Validation Checklist
- [ ] All endpoints use async `def` and non-blocking I/O (no synchronous `subprocess.run` or blocking file/DB calls inside the event loop).
- [ ] Pydantic v2 models (`ConfigDict(extra="forbid")`) validate all inputs and config payloads.
- [ ] Secrets (`secrets.enc`, GitHub tokens, API keys) are masked (`••••last4`) and never returned in full by any `GET /config/*` or `/harnesses` endpoint (AC-M4-08.3, X-01 🛡).
