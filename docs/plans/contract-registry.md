# Cross-Module Contract Registry: ntc-ai-center-hub

> **Status:** APPROVED  
> **Authority:** Derived from `docs/plans/system-roadmap.md` and `SPEC.md` (v0.3); changes follow the Contract Protocol in `docs/WORKFLOW.md` §3.  
> **Last Synchronized:** 2026-10-24  

<!--
STRUCTURE INVARIANTS (checked by scripts/validate-plans.ps1):
- Keep the headings "## 1. Contracts" and "## 2. Contract Change Log" and the table header names.
- Contract ID format: CT-<PROVIDER>-<NN>; <PROVIDER> must equal the Provider column and be a registered module code.
- Consumers: comma-separated registered module codes (or —).
- Status: DRAFT | AVAILABLE | DEPRECATED. An AVAILABLE contract's Spec Path must exist in the repository.
-->

---

## 1. Contracts

| Contract ID | Type | Provider | Consumers | Spec Path | Version | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `CT-SYS-01` | Schema | `SYS` | `CORE`, `ADP`, `GIT`, `ORCH`, `DEP`, `API`, `CLI`, `BOT`, `WEB` | `contracts/schemas/domain-enums.schema.json` | `0.1.0` | `DRAFT` |
| `CT-CORE-01` | Library | `CORE` | `ADP`, `GIT`, `ORCH`, `DEP`, `API` | `contracts/core/store-protocol.py` | `0.1.0` | `DRAFT` |
| `CT-CORE-02` | Event | `CORE` | `ORCH`, `API`, `BOT` | `contracts/events/hub-events.schema.json` | `0.1.0` | `DRAFT` |
| `CT-ADP-01` | Library | `ADP` | `ORCH`, `API` | `contracts/adapters/harness-protocol.py` | `0.1.0` | `DRAFT` |
| `CT-GIT-01` | Library | `GIT` | `ORCH`, `DEP`, `API` | `contracts/git/git-service-protocol.py` | `0.1.0` | `DRAFT` |
| `CT-ORCH-01` | Library | `ORCH` | `API` | `contracts/orchestrator/orchestrator-protocol.py` | `0.1.0` | `DRAFT` |
| `CT-DEP-01` | Library | `DEP` | `API` | `contracts/deploy/deploy-protocol.py` | `0.1.0` | `DRAFT` |
| `CT-API-01` | REST | `API` | `CLI`, `BOT`, `WEB` | `contracts/openapi/hubd-rest.yaml` | `0.1.0` | `DRAFT` |
| `CT-API-02` | Event | `API` | `CLI`, `BOT`, `WEB` | `contracts/events/ws-stream.schema.json` | `0.1.0` | `DRAFT` |

**Type values:** REST | gRPC | Event | Schema | Library

---

## 2. Contract Change Log
*(Append-only. Breaking changes require explicit Human Architect approval and a decision record before implementation.)*

| Date | Contract ID | Version | Breaking | Impacted Consumers | Decision / Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2026-10-24 | `CT-SYS-01` | `—` → `0.1.0` | No | `CORE`, `ADP`, `GIT`, `ORCH`, `DEP`, `API`, `CLI`, `BOT`, `WEB` | Initial registration at Gate 2: shared domain enums (`TaskKind`, `TaskStatus`, `BlockReason`, `RoleName`, `GateName`, `DeployTargetType`). |
| 2026-10-24 | `CT-CORE-01` | `—` → `0.1.0` | No | `ADP`, `GIT`, `ORCH`, `DEP`, `API` | Initial registration at Gate 2: async unit-of-work, repositories (`hub.db` WAL), encrypted `secrets.enc` store, and YAML config loader. |
| 2026-10-24 | `CT-CORE-02` | `—` → `0.1.0` | No | `ORCH`, `API`, `BOT` | Initial registration at Gate 2: persisted `events` table & in-process `EventBus` envelopes for cross-channel sync (`X-08`). |
| 2026-10-24 | `CT-ADP-01` | `—` → `0.1.0` | No | `ORCH`, `API` | Initial registration at Gate 2: `HarnessAdapter` protocol (`check_auth`, `list_models`, `login`, `run`, `cancel`), `RunRequest`, and `AgentEvent` union (`ARCH §6.1`). |
| 2026-10-24 | `CT-GIT-01` | `—` → `0.1.0` | No | `ORCH`, `DEP`, `API` | Initial registration at Gate 2: `GitService` & `GitHubService` protocol (worktrees, ref guards `🛡`, PR creation, 4-condition merge gate `🛡`, `D-16` cleanup). |
| 2026-10-24 | `CT-ORCH-01` | `—` → `0.1.0` | No | `API` | Initial registration at Gate 2: `Orchestrator`, `Scheduler`, `InterviewService`, `ReportService`, and §2.5 `MessageSplitter` interfaces. |
| 2026-10-24 | `CT-DEP-01` | `—` → `0.1.0` | No | `API` | Initial registration at Gate 2: `DeployTarget` protocol (`preview`, `teardown`, `deploy`, `rollback` — `ARCH §13`) & Cloudflare Tunnel/Access manager (`D-17`). |
| 2026-10-24 | `CT-API-01` | `—` → `0.1.0` | No | `CLI`, `BOT`, `WEB` | Initial registration at Gate 2: `hubd` REST OpenAPI specification over Unix socket (`~/.ntc-hub/hubd.sock`) and loopback (`127.0.0.1:<port>`). |
| 2026-10-24 | `CT-API-02` | `—` → `0.1.0` | No | `CLI`, `BOT`, `WEB` | Initial registration at Gate 2: WebSocket `/ws?since_event_id=` event replay and live log streaming schema (`ARCH §11`). |
