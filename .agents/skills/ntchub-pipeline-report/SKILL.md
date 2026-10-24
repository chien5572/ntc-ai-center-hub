---
name: ntchub-pipeline-report
description: >-
  Use this skill when implementing the Orchestrator pipeline engine (`ARCH §5`),
  Interview Service (`ARCH §8`, `US-M1-03`), Structured Review Validator (`D-07`, `US-M1-09`),
  Fix Loop early-stopping (`D-08`, `US-M1-10`), or Report & Message Splitter (`SPEC §2.5`, `US-M1-16`).
---

# NTC Hub Pipeline Engine, Interview, Review & Report Rules

Enforces the domain rules of `ntc-ai-center-hub` across the 8 roles (`interview`, `plan`, `code`, `reproduce`, `test`, `review`, `fix`, `report` — SPEC §2.2).

## When to Activate
- Implementing `InterviewService` (`US-M1-03`, `US-M2-03`, `US-M3-01`).
- Implementing `Orchestrator` stage execution (`ARCH §5.3`), artifact handoff (`.hub/`), or fix loops (`US-M1-10`).
- Implementing `ReviewValidator` (`D-07`, `US-M1-09`) or `ReportService` & the §2.5 message splitter (`D-13`, `US-M1-16`, `US-M2-06`).

## Core Domain Rules

### 1. Interview Service & `spec.md` Generation (FR-3, FR-4, FR-5, US-M1-03)
- **Read-Only Repo Inspection**: For `task` and `fix`, inspect the repo in `read_only` mode (`README`, structure, `AGENTS.md`/`CLAUDE.md`) before asking the first question (`AC-M1-03.2`).
- **One Question Per Turn (`AC-M1-03.3`)**: Every bot turn asks **at most 1 primary question** (optionally accompanied by up to 3 inline suggestion buttons on Telegram — `D-09`).
- **Periodic Summary & Defaults (`AC-M1-03.4`)**: Summarize understood requirements after at most 5 turns; when the owner replies `"tuỳ bạn"`, explicitly state the chosen default.
- **`/go` Output (`SPEC §2.3`, `AC-M1-03.5`)**: Generate `.hub/spec.md` with all 8 mandatory sections (`Mục tiêu`, `Phạm vi (in scope)`, `Ngoài phạm vi (non-goals)`, `Tiêu chí chấp nhận` with ≥1 numbered testable item, `Ràng buộc kỹ thuật`, `Lệnh test`, `Deploy target`, `Repo`).

### 2. Structured Review Consistency Rule (D-07, US-M1-09)
- Review role runs in `read_only` mode with respect to source code (any edits outside `.hub/review.json` are reverted by `hubd` with a warning — `AC-M1-09.4`).
- Validate `.hub/review.json` against schema (`verdict`, `issues[{severity, file, line, note}]`). If invalid JSON/schema, retry review **once** with a schema reminder; if still invalid, force `verdict = "changes_requested"` (`AC-M1-09.2`).
- **Consistency Override (`D-07`, `AC-M1-09.3`)**: Even if `review.json` says `"verdict": "approve"`, `hubd` MUST override it to `"changes_requested"` (and log a warning) if:
  - There is `>= 1` issue with `severity in ("blocker", "major")`, OR
  - There are `>= 3` issues with `severity == "minor"`.

### 3. Bounded Fix Loop & Early Stop (D-08, US-M1-10)
- `max_rounds` (default `3`) counts the number of `fix` executions **after** the initial `test`/`review` pass (`D-08`). Failure on the 4th test/review transitions task to `failed`.
- **Repeated Error Signature Early Stop (`AC-M1-10.3`)**: Normalize failing test names + error messages into a signature. If the signature is identical across **2 consecutive rounds**, stop immediately and mark `failed` with reason `"lỗi lặp lại"`.

### 4. Report Service & Long Message Splitter (🛡 Anti-Goal #2, SPEC §2.5, D-13, US-M1-16)
- **Structured Ground Truth (`AC-M1-16.1`)**: Build reports from `hubd`'s DB records (`STAGE_RUN`, hub test results, `review.json`, `Usage`); only use the `report` LLM role to draft the concise `"Đã làm"` and `"Cần bạn xem"` text.
- **Message Splitting Rules (`SPEC §2.5`, `AC-M1-16.4 🛡`)**:
  1. **Part 1 (Main Message)**: Strictly **≤ 10 lines**, matches FR-36 format, self-contained, carries the action buttons (`[Preview] [Merge] [Chi tiết]`).
  2. **Subsequent Parts `(2/n)...(n/n)`**: Each part **≤ 15 lines** and **≤ 3,500 chars**.
  3. **Split Hierarchy**: Split by (1) section boundary → (2) bullet point boundary → (3) sentence boundary. **NEVER** split inside a sentence, fenced code block, URL, or table.
  4. **Max 5 Parts**: If content exceeds 5 parts, Part 5 ends at a clean boundary with `"📄 Phần còn lại: xem REPORT.md"` and a file attachment button.
