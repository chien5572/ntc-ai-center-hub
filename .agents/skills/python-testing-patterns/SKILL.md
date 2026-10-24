---
name: python-testing-patterns
description: >-
  Use this skill when writing unit, integration, or security tests for ntc-ai-center-hub
  using pytest, pytest-asyncio, FakeHarness, M0 CLI output fixtures, or security regression suites.
---

# Python Testing Patterns (`pytest` + `FakeHarness` + Security Suites)

Provides testing architecture and patterns required by SPEC §10 and all `🛡` Acceptance Criteria.

## When to Activate
- Building `FakeHarness` (`SPEC §10`, `X-06`) to test the orchestrator, scheduler, and gates without burning real LLM quota.
- Writing unit tests for harness stream-JSON parsers against `tests/fixtures/<harness>/` (`AC-M0-01.3`, `AC-M1-05.4`).
- Writing unit tests for the Telegram message splitter (`SPEC §2.5`, `AC-M1-16.4 🛡`), review verdict consistency (`D-07`, `AC-M1-09.3`), and auto-merge guards (`D-15`, `AC-M1-08.5 🛡`).
- Writing security regression tests (`AC-M1-05.3 🛡`, `AC-M1-11.6 🛡`, `AC-M2-01.2 🛡`, `X-01 🛡`).

## Core Testing Architecture

### 1. `FakeHarness` (SPEC §10, X-06)
Implement `FakeHarness` conforming to `HarnessAdapter` (`ARCH §6.1`) with scriptable behaviors per stage/round:
- **`pass`**: Writes expected artifact (`spec.md`, `plan.md`, `test-report.md`, `review.json` with `verdict="approve"`, `REPORT.md`) and yields `Usage` + `Done(exit_code=0)`.
- **`fail_test` / `changes_requested`**: Simulates failing tests or review issues to test fix loops (`max_rounds=3`) and repeated error signature early-stop (`AC-M1-10.3`).
- **`rate_limited`**: Yields `RateLimited(retry_after=...)` to test `TASK.block_reason="rate_limit"` and fallback routing (`US-M3-04`, `US-M3-05`).
- **`auth_expired`**: Simulates expired OAuth session to test `TASK.block_reason="auth_expired"` (`D-05`, `AC-M2-08.4`).
- **`hang`**: Sleeps past `timeout_s` to verify container kill and `STAGE_RUN` timeout handling (`AC-M1-05.5`).

### 2. Independent Hub Test Verification (🛡 AC-M1-08.1, AC-M1-08.2, AC-M1-08.3)
Always include tests proving:
1. Even if the agent writes `"all tests passed"` in `test-report.md`, `hubd` executes `project.test_cmd` independently and marks the stage failed if exit code != 0.
2. If an agent modifies `.hub/project.yml` or attempts to alter `test_cmd`, `hubd` ignores the change and emits a warning in the report.

### 3. Malicious Agent Regression Suite (🛡 AC-M1-05.3, AC-M1-11.6, X-01)
Include automated CI tests simulating a rogue agent inside the worktree/container that attempts to:
- Read `~/.ntc-hub/secrets.enc`, `~/.ssh`, or `os.environ["GITHUB_TOKEN"]`.
- Modify `.git/config`, add a remote, create/checkout `main` locally, modify `.git/hooks`, or run `git push`.
- **Assertion**: Zero credentials read, push fails inside container, and zero changes reach the remote outside `refs/heads/hub/<id>-<slug>`.
- **Post-test secret scan (X-01 🛡)**: Grep all `logs/` and SQLite tables (except `secrets.enc`) for test secret strings and assert 0 matches.

### 4. Telegram Message Splitter Torture Test (🛡 AC-M1-16.4)
Test the shared splitter module (`SPEC §2.5`) with:
- 50 "Đã làm" items, a single 2,000-char bullet point, fenced code blocks, and long URLs.
- **Assertions**:
  - Part 1 ≤ 10 lines and contains action buttons.
  - Every subsequent part ≤ 15 lines and ≤ 3,500 characters.
  - Concatenating parts preserves 100% of original body characters (modulo part headers).
  - Never splits mid-sentence, mid-code-block, or mid-URL.
  - Maximum 5 parts (overflow falls back to `REPORT.md` attachment prompt).
