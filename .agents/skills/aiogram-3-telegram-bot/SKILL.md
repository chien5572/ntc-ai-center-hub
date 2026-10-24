---
name: aiogram-3-telegram-bot
description: >-
  Use this skill when building or modifying the Telegram gateway (M2) using aiogram 3,
  including Forum Topics (`message_thread_id`), owner user ID security filtering,
  private-chat secret auto-deletion, inline approval keyboards, pinned status editing,
  and report message splitting.
---

# Telegram Gateway with `aiogram 3` (Forum Topics & Remote Control)

Guides the implementation of the Telegram bot gateway in `hubd` (ARCH §10; PRD §4.8, §4.9; SPEC §2.5, §5 M2).

## When to Activate
- Setting up `aiogram 3` long-polling router, middlewares, and error handlers.
- Implementing topic creation (`#<id> · <slug>`), General topic vs. Task topic command routing (`message_thread_id`).
- Implementing interactive interviews with suggestion buttons (`D-09`), gate approval buttons (`US-M2-04`), pinned status messages (`D-10`, `US-M2-05`), and ≤10-line reports (`US-M2-06`).
- Implementing private chat secret handling (`/github`, `/apikey`, `/setup` — `US-M2-09 🛡`).

## Core Invariants for `ntc-ai-center-hub`

### 1. Owner Whitelist Middleware (🛡 AC-M2-01.2, FR-40)
Install an outer middleware on `Dispatcher` covering both `message` and `callback_query`:
- If `event.from_user.id != config.owner_user_id`:
  - Log a warning (` unauthorized telegram access from user_id=...`).
  - Return immediately (**do NOT reply, do NOT answer callback, execute zero side effects**).

### 2. Sensitive Commands Only in Private Chat (🛡 US-M2-09, FR-41)
For `/github`, `/apikey`, and secret steps in `/setup`:
- **If sent in a group/supergroup (`AC-M2-09.1`)**:
  1. Immediately delete the user's message (`bot.delete_message`).
  2. Send warning: `"🔒 Lệnh này chỉ dùng trong chat riêng với bot. Tin nhắn của bạn đã được xoá."`
  3. Do NOT save the secret.
- **If sent in private chat (`AC-M2-09.2`)**:
  1. Encrypt and save secret into `secrets.enc`.
  2. Delete the user's secret message within **≤ 3 seconds**.
  3. Reply with masked confirmation: `"✅ Đã lưu ••••<4 ký tự cuối>"`.
  4. Never log the raw secret string (`AC-M2-09.3`).

### 3. Topic Mapping & Anti-Spam Policy (🛡 US-M2-02, US-M2-05)
- **General Topic**: `/task <project> <req>`, `/fix <project> <bug>`, `/new <idea>` creates task + forum topic `#<id> · <slug>` (`create_forum_topic`), posts a link button `"Mở topic →"` in General, and starts the interview inside the new topic.
- **Task Topic (`TASK.tg_topic_id == message_thread_id`)**: Commands omit `<id>` and infer it from `message_thread_id`. Plain text when `status == 'interviewing'` is routed to `InterviewService`; when in "commenting on gate" mode (`AC-M2-04.2`), routed as gate feedback.
- **Pinned Status Message (`D-10`, `AC-M2-05.2`)**: Create and pin 1 status message per task topic; `edit_message_text` on stage/status change.
- **New Message Budget (`AC-M2-05.1`, `AC-M2-05.3`)**: Only send a new message on stage transition (`▶ <Stage> — harness · model`), approval gate, error, or completion (≤ 15 bot messages per clean pipeline run).

### 4. Gate Inline Keyboards & Cross-Channel Sync (US-M2-04)
- Callback data format: `approve:<approval_id>`, `reject:<approval_id>`, `comment:<approval_id>`, `preview:<task_id>`, `merge:<task_id>`, `detail:<task_id>`.
- Once decided (via Telegram, CLI, or Webapp), edit the gate message inline keyboard to a single static label (`✅ Đã duyệt`, `💬 Đã góp ý`, `✖ Đã từ chối`, or `✅ Đã duyệt (qua CLI)`).
- Report action buttons (`[Preview] [Merge] [Chi tiết]`) **persist** after clicks (`AC-M2-06.2`).

### 5. Long Message Splitting (🛡 SPEC §2.5, D-13, AC-M2-06.1, AC-M2-06.5)
- Always pass outgoing reports, spec/plan summaries, and error details through the shared §2.5 splitter.
- Part 1 is always ≤ 10 lines, self-contained, and holds the inline keyboard.
- Send parts `(2/n)...(n/n)` sequentially right after Part 1 with retry-in-order on network failure.
