---
name: sqlalchemy-2-async-sqlite
description: >-
  Use this skill when designing models, writing database queries, managing transactions,
  or creating Alembic migrations for hub.db using SQLite (WAL mode) and SQLAlchemy 2.0 / SQLModel.
---

# SQLAlchemy 2.0 Async & SQLite WAL Patterns (`hub.db`)

Guides the implementation of the persistence layer (`~/.ntc-hub/hub.db`) to satisfy durability, concurrency, and recovery invariants (ARCH §1.5, §3, §4; SPEC US-M1-13, X-02).

## When to Activate
- Defining or updating schema entities: `PROJECT`, `TASK`, `STAGE_RUN`, `AGENT_RUN`, `APPROVAL`, `ARTIFACT`, `INTERVIEW`, `MESSAGE`, `HARNESS`, `ROUTE`, `PROMPT`, `PROMPT_VERSION`, `DEPLOYMENT`, `audit_log`, `events`.
- Writing Alembic migrations or configuring the async SQLite engine (`aiosqlite`).
- Implementing task state machine transitions or queue/scheduler locking.

## Core Invariants for `ntc-ai-center-hub`

### 1. Mandatory SQLite Pragmas on Every Connection (X-02)
Always configure WAL mode, busy timeout, and foreign keys via SQLAlchemy connection events so parallel tasks (`US-M1-14`) never fail with `SQLITE_BUSY`:
```python
from sqlalchemy import event
from sqlalchemy.ext.asyncio import create_async_engine

engine = create_async_engine("sqlite+aiosqlite:///" + str(db_path))

@event.listens_for(engine.sync_engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA journal_mode=WAL")
    cursor.execute("PRAGMA synchronous=NORMAL")
    cursor.execute("PRAGMA busy_timeout=5000")
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.close()
```

### 2. Persist-Before-Side-Effect Rule (ARCH §1.5, §5.3)
- **Rule**: Every state transition (`TASK.status`, `STAGE_RUN.status`, `APPROVAL.status`) MUST be committed to SQLite **before** spawning containers, calling GitHub APIs, or sending Telegram messages.
- **Block Reason Invariant (SPEC §2.1)**:
  - `TASK.block_reason` ∈ `{'rate_limit', 'auth_expired'}` when and only when `TASK.status == 'rate_limited'`.
  - Whenever `TASK.status` transitions away from `'rate_limited'`, clear `block_reason = NULL`, `blocked_harness = NULL`, `blocked_model = NULL` in the same transaction.

### 3. Crash Recovery Invariant (AC-M1-13.1, AC-M1-13.2)
- On `hubd` startup:
  - Query tasks with `status == 'running'`.
  - Reset their worktree (`git reset --hard` to the last `hub(<id>): ...` commit) and transition `status` to `'queued'`.
  - Leave tasks in `'waiting_approval'`, `'awaiting_merge'`, `'interviewing'`, `'paused'`, `'rate_limited'` untouched.

### 4. Audit Log Immutability (FR-31, AC-M1-11.5, X-03)
- Every external side-effect (git push, PR create, PR merge, branch delete, repo create, branch protection, `test_cmd` modification, deploy/rollback) writes an append-only row to `audit_log` (`timestamp`, `task_id`, `project_id`, `action`, `ref`, `result`, `detail`).
