# Hướng Dẫn Sử Dụng Bộ AI Skills (`AI-skill-help.md`) — `ntc-ai-center-hub`

> **Tài liệu dành cho Chủ dự án & các AI Coding Agents (Antigravity, Claude Code, Codex CLI, DeepSeek Harness).**
> Bạn có thể đính kèm trực tiếp file `@[AI-skill-help.md]` này vào prompt để chỉ định cho Agent biết cần kích hoạt đúng skill nào cho từng giai đoạn (`M0 → M5`), module hoặc vai trò (`role`) trong dự án.

---

## 1. Vị trí lưu trữ & Cơ chế nạp tự động

Toàn bộ bộ skill (cả skill chính thức từ **Felo AI**, **SkillsMP** lẫn **Custom Skills** thiết kế riêng cho kiến trúc `ntc-ai-center-hub`) đã được cài đặt tại thư mục chuẩn của workspace:

```text
.agents/skills/
├── aiogram-3-telegram-bot/       # [Custom/Stack] Telegram Gateway + Topics + Bảo mật chat riêng (M2)
├── agent-browser/                # [SkillsMP - Vercel] Tự động hoá trình duyệt & E2E test (M4, M5)
├── cloudflare-tunnel-caddy/      # [Custom/Stack] Cloudflare Tunnel + Access + Caddy dynamic proxy (M4, M5)
├── deploy-to-vercel/             # [SkillsMP - Vercel] Triển khai preview/prod lên Vercel (M5)
├── docker-container-security/    # [Custom/Stack] Sandbox container ntchub-agent & Preview Docker (M1, M5)
├── felo-search/                  # [Felo AI] Tìm kiếm web thời gian thực có trích dẫn nguồn (M0–M5)
├── felo-superAgent/              # [Felo AI] Nghiên cứu sâu đa bước & tổng hợp tài liệu (M0, M3)
├── felo-web-fetch/               # [Felo AI] Trích xuất nội dung trang web/tài liệu kỹ thuật (M0–M5)
├── frontend-design/              # [SkillsMP - Anthropic] Thiết kế giao diện Webapp hiện đại, tránh mẫu rập khuôn (M4)
├── grill-me/                     # [SkillsMP - Matt Pocock] Phỏng vấn đào sâu, phản biện kế hoạch & spec (M1–M3)
├── ntchub-git-safety/            # [Custom Domain] Quy tắc bất biến bảo vệ Git/GitHub — Anti-goal #1 (M1, M3)
├── ntchub-harness-adapters/      # [Custom Domain] Adapter CLI headless cho claude, codex, agy, dsh (M0, M1, M3)
├── ntchub-pipeline-report/       # [Custom Domain] Pipeline, Interview 1-câu/lượt, Review D-07, Chia tin §2.5 (M1, M2)
├── python-cli-typer-rich/        # [Custom/Stack] Xây dựng `hub` CLI bằng Typer + Rich + Textual (M1–M5)
├── python-fastapi-development/   # [Custom/Stack] Xây dựng `hubd` daemon bằng FastAPI + WebSocket + Unix Socket (M1, M4)
├── python-testing-patterns/      # [Custom/Stack] Pytest, FakeHarness, test bảo mật agent độc hại (M1–M5)
├── skill-creator/                # [SkillsMP - Anthropic] Tạo mới & tối ưu hoá SKILL.md / prompt cho 8 vai (M1–M4)
├── sqlalchemy-2-async-sqlite/    # [Custom/Stack] SQLite WAL + SQLAlchemy 2.0 Async + Alembic + Crash Recovery (M1)
├── vercel-react-best-practices/  # [SkillsMP - Vercel] Tối ưu hoá hiệu năng React + Vite + TypeScript (M4)
├── web-design-guidelines/        # [SkillsMP - Vercel] Kiểm tra UI/UX, accessibility & layout cho Webapp (M4)
└── webapp-testing/               # [SkillsMP - Anthropic] Kiểm thử tự động Webapp bằng Playwright (M4, M5)
```

---

## 2. Bảng tra cứu nhanh: Làm chức năng/module nào → Dùng Skill nào?

| Giai đoạn | Module / Service (theo `ARCHITECTURE.md`) | User Story (`SPEC.md`) | Skill cần kích hoạt (`.agents/skills/<tên>/SKILL.md`) |
| :---: | :--- | :--- | :--- |
| **M0** | **Spike xác minh 4 Harness CLI** (`claude`, `codex`, `agy`, `dsh`) | `US-M0-01` → `US-M0-03` | `ntchub-harness-adapters`, `felo-search`, `felo-web-fetch`, `docker-container-security` |
| **M1** | **`hubd` Daemon & API Layer** (FastAPI, REST, WebSocket, Unix Socket) | `US-M1-01`, `US-M1-13` | `python-fastapi-development`, `sqlalchemy-2-async-sqlite` |
| **M1** | **Database & Persistence (`hub.db`)** (SQLite WAL, Alembic, Crash Recovery) | `US-M1-13`, `US-M1-14`, `X-02` | `sqlalchemy-2-async-sqlite` |
| **M1** | **`hub` CLI** (Typer, Rich, Textual `attach`, `doctor`, `logs -f`) | `US-M1-01`, `US-M1-03`, `US-M1-12`, `US-M1-15` | `python-cli-typer-rich` |
| **M1 & M3** | **Harness Adapters & Sandbox Container** (`ntchub-agent`) | `US-M1-05 🛡`, `US-M3-02`, `US-M3-03 🛡`, `US-M3-04`, `US-M3-05` | `ntchub-harness-adapters`, `docker-container-security` |
| **M1 & M3** | **Git / GitHub Service** (Worktree, Ref Guard, 4 điều kiện Merge, Dọn branch) | `US-M1-02`, `US-M1-11 🛡`, `US-M3-01 🛡` | `ntchub-git-safety` |
| **M1** | **Orchestrator, Interview, Review & Report Service** | `US-M1-03`, `US-M1-06` → `US-M1-10`, `US-M1-16 🛡` | `ntchub-pipeline-report`, `grill-me`, `skill-creator` |
| **M1 → M5** | **Hệ thống Test (`FakeHarness`, Unit, Integration, Bảo mật 🛡)** | `SPEC §10`, mọi AC có biểu tượng `🛡` | `python-testing-patterns` |
| **M2** | **Telegram Gateway (`aiogram 3`, Forum Topics, Nút duyệt, Xoá secret)** | `US-M2-01 🛡` → `US-M2-10` | `aiogram-3-telegram-bot`, `ntchub-pipeline-report` |
| **M4** | **Webapp SPA** (React, Vite, TypeScript, Dashboard, Diff, Live Log) | `US-M4-02` → `US-M4-11` | `frontend-design`, `vercel-react-best-practices`, `web-design-guidelines` |
| **M4 & M5** | **Kiểm thử E2E Webapp & Preview URL** | `Exit M4`, `Exit M5` | `webapp-testing`, `agent-browser` |
| **M4 & M5** | **Cloudflare Tunnel + Access & Deploy Service** (Docker/Caddy, Vercel, CF) | `US-M4-01 🛡`, `US-M5-01 🛡` → `US-M5-07` | `cloudflare-tunnel-caddy`, `deploy-to-vercel`, `docker-container-security` |

---

## 3. Chi tiết cách dùng từng nhóm Skill

### Nhóm 1: Custom Domain Skills (Đặc thù riêng của `ntc-ai-center-hub`)

#### 1.1. `ntchub-git-safety` — Bảo vệ tuyệt đối Repo & GitHub (`🛡 Anti-Goal #1`)
* **Đường dẫn:** [`.agents/skills/ntchub-git-safety/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/ntchub-git-safety/SKILL.md)
* **Dùng khi nào:** Khi viết hoặc sửa `GitService`, `GitHubService`, quản lý git worktree (`~/.ntc-hub/work/<id>/`), logic push, tạo PR, kiểm tra điều kiện merge, rebase từ nhánh mặc định, hoặc xoá branch (`D-16`).
* **Quy tắc cốt lõi mà skill này cưỡng chế:**
  * Chặn cứng trong code mọi lệnh push ra ngoài `refs/heads/hub/<id>-<slug>` và mọi `--force` (trừ `--force-with-lease` khi chính `hubd` rebase).
  * Kiểm tra đủ 4 điều kiện trước khi gọi GitHub Merge API: (1) `hubd` tự chạy `test_cmd` pass trên HEAD hiện tại, (2) `review.verdict == "approve"`, (3) gate `merge` đã duyệt/tắt, (4) không xung đột.
  * Kiểm tra `.git/config` và `.git/hooks` không bị agent chỉnh sửa trước khi push; ghi `audit_log` cho mọi thao tác.

#### 1.2. `ntchub-harness-adapters` — Kết nối 4 Harness CLI Headless
* **Đường dẫn:** [`.agents/skills/ntchub-harness-adapters/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/ntchub-harness-adapters/SKILL.md)
* **Dùng khi nào:** Khi làm giai đoạn **M0** (viết `docs/harness-matrix.md`, `scripts/smoke-<harness>.sh`), **M1** (`ClaudeCodeAdapter`, `AntigravityAdapter`) và **M3** (`CodexAdapter`, `DeepSeekHarnessAdapter`, fallback, rate-limit).
* **Quy tắc cốt lõi:**
  * Chỉ gọi CLI chính chủ ở chế độ headless (`claude -p`, `agy -p`, `codex exec`, `dsh --profile ntchub`), tuyệt đối không import SDK của hãng (`FR-15a 🛡`).
  * Tách biệt OAuth (cho `claude`, `codex`, `agy`) và API key (chỉ truyền vào env của container `dsh` — `AC-M3-03.2 🛡`).
  * Xử lý chuẩn `TASK.block_reason` (`rate_limit` tự chạy lại khi tới `rate_limited_until` vs. `auth_expired` chờ `/login <harness>` — `SPEC §2.1`, `D-05`).

#### 1.3. `ntchub-pipeline-report` — Điều phối Pipeline, Phỏng vấn, Review & Chia tin nhắn
* **Đường dẫn:** [`.agents/skills/ntchub-pipeline-report/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/ntchub-pipeline-report/SKILL.md)
* **Dùng khi nào:** Khi code `Orchestrator`, `InterviewService`, `ReviewValidator`, vòng lặp `fix`, và `ReportService` (cả CLI lẫn Telegram).
* **Quy tắc cốt lõi:**
  * **Interview:** Đọc repo ở chế độ `read_only` trước khi hỏi; hỏi **đúng 1 câu mỗi lượt**; sinh `.hub/spec.md` đủ 8 mục chuẩn `SPEC §2.3`.
  * **Review (`D-07`):** Tự động chuyển `approve` thành `changes_requested` nếu có ≥ 1 lỗi `blocker`/`major` hoặc ≥ 3 lỗi `minor`; hoàn tác nếu agent review tự ý sửa code ngoài `.hub/review.json`.
  * **Fix Loop (`D-08`):** Tối đa `max_rounds = 3` vòng fix sau lần test/review đầu; dừng sớm nếu chữ ký lỗi test lặp lại 2 vòng liên tiếp (`AC-M1-10.3`).
  * **Bộ chia tin nhắn (`SPEC §2.5`, `D-13 🛡`):** Tin chính luôn ≤ 10 dòng và mang nút bấm; các tin phụ `(2/n)...(n/n)` ≤ 15 dòng và ≤ 3.500 ký tự, không bao giờ cắt ngang câu, khối code hay đường link.

---

### Nhóm 2: Custom Tech-Stack Skills (Chuẩn hoá kiến trúc Python, Docker, Telegram, Cloudflare)

#### 2.1. `python-fastapi-development` — Daemon `hubd` & API Layer
* **Đường dẫn:** [`.agents/skills/python-fastapi-development/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/python-fastapi-development/SKILL.md)
* **Dùng cho module:** `API layer` (`REST + WebSocket /ws`), `EventBus`, quản lý vòng đời `hubd`.
* **Khi nào dùng:** Khi viết API cho CLI (qua Unix socket `~/.ntc-hub/hubd.sock` quyền `0600`) và Webapp (chỉ bind `127.0.0.1:<port>` — `AC-M4-01.1 🛡`), đồng bộ sự kiện đa kênh trong ≤ 2–3 giây (`X-08`).

#### 2.2. `sqlalchemy-2-async-sqlite` — Lưu trữ bền vững `hub.db`
* **Đường dẫn:** [`.agents/skills/sqlalchemy-2-async-sqlite/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/sqlalchemy-2-async-sqlite/SKILL.md)
* **Dùng cho module:** `DB (SQLite)`, Alembic migrations, `Scheduler` queue, `Orchestrator` state transitions.
* **Khi nào dùng:** Khi thiết kế bảng dữ liệu (`ARCH §4`), bật `PRAGMA journal_mode=WAL` + `busy_timeout=5000`, đảm bảo nguyên tắc *"ghi trạng thái xuống SQLite trước khi thực hiện tác dụng phụ"* (`ARCH §1.5`) và khôi phục sau khi `hubd` bị kill (`US-M1-13`).

#### 2.3. `python-cli-typer-rich` — Giao diện dòng lệnh `hub`
* **Đường dẫn:** [`.agents/skills/python-cli-typer-rich/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/python-cli-typer-rich/SKILL.md)
* **Dùng cho module:** `hub CLI` (`ARCH §11`, `US-M1-01`, `US-M1-03`, `US-M1-15`).
* **Khi nào dùng:** Khi xây dựng các lệnh `hub task`, `hub fix`, `hub tasks`, `hub status`, `hub logs -f`, `hub attach` (TUI), `hub doctor` (in `✓`/`✗` kèm lệnh sửa lỗi).

#### 2.4. `docker-container-security` — Cách ly Container `ntchub-agent` & Preview
* **Đường dẫn:** [`.agents/skills/docker-container-security/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/docker-container-security/SKILL.md)
* **Dùng cho module:** `Harness adapters` (container runner), `Deploy service` (preview Docker ở M5).
* **Khi nào dùng:** Khi viết `Dockerfile` cho `ntchub-agent` và lệnh `docker run` cách ly: chỉ mount worktree của task + thư mục credential của đúng 1 harness; chặn tuyệt đối truy cập vào `~/.ntc-hub/secrets.enc`, `~/.ssh`, `GITHUB_TOKEN`, Docker socket (`AC-M1-05.2 🛡`, `AC-M1-05.3 🛡`).

#### 2.5. `python-testing-patterns` — Chiến lược Kiểm thử & `FakeHarness`
* **Đường dẫn:** [`.agents/skills/python-testing-patterns/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/python-testing-patterns/SKILL.md)
* **Dùng cho module:** Toàn bộ thư mục `tests/` (`SPEC §10`).
* **Khi nào dùng:** Khi xây dựng `FakeHarness` đầu giai đoạn M1 để test toàn bộ pipeline mà không tốn quota thật; khi viết unit test bộ chia tin nhắn (`AC-M1-16.4 🛡`), test xác minh `test_cmd` độc lập (`AC-M1-08 🛡`), và bộ test hồi quy "agent độc hại" (`AC-M1-11.6 🛡`).

#### 2.6. `aiogram-3-telegram-bot` — Cổng điều khiển Telegram (M2)
* **Đường dẫn:** [`.agents/skills/aiogram-3-telegram-bot/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/aiogram-3-telegram-bot/SKILL.md)
* **Dùng cho module:** `Telegram gateway` (`ARCH §10`, `SPEC §5 M2`).
* **Khi nào dùng:** Khi lập trình bot với `aiogram 3`: tạo Forum Topic `#<id> · <slug>`, lọc cứng `owner_user_id` (`AC-M2-01.2 🛡`), tự xoá tin nhắn chứa secret trong chat riêng ≤ 3 giây (`US-M2-09 🛡`), ghim & sửa tin nhắn trạng thái (`D-10`), đồng bộ nút duyệt giữa Telegram – CLI – Webapp.

#### 2.7. `cloudflare-tunnel-caddy` — Truy cập từ xa Zero-Trust & Preview (M4 & M5)
* **Đường dẫn:** [`.agents/skills/cloudflare-tunnel-caddy/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/cloudflare-tunnel-caddy/SKILL.md)
* **Dùng cho module:** `Webapp setup` (M4) & `Deploy service` (M5) (`SPEC D-17`).
* **Khi nào dùng:** Khi cấu hình `cloudflared` tunnel + Cloudflare Access (chỉ cho phép email chủ dự án) cho `hub.<domain>` và `*.preview.<domain>`, kết hợp Caddy Admin API để cấp phát và thu hồi subdomain preview tự động sau 24h.

---

### Nhóm 3: Skills từ SkillsMP (`skillsmp.com` — Anthropic, Vercel, Matt Pocock)

| Tên Skill | Đường dẫn | Dùng khi nào & Cho chức năng gì? |
| :--- | :--- | :--- |
| **`grill-me`** | [`.agents/skills/grill-me/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/grill-me/SKILL.md) | - **Khi lập kế hoạch M1–M5:** Dùng để phản biện thiết kế module/phase trước khi code.<br>- **Khi thiết kế prompt cho vai `interview` (`US-M1-03`, `US-M1-04`):** Áp dụng triết lý hỏi từng câu sắc bén, đi dọc từng nhánh quyết định để làm rõ yêu cầu của chủ dự án. |
| **`skill-creator`** | [`.agents/skills/skill-creator/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/skill-creator/SKILL.md) | - **Ở `US-M1-04` & `US-M4-05`:** Dùng để soạn thảo, tinh chỉnh và đánh giá bộ prompt Jinja2 mặc định cho 8 vai (`interview`, `plan`, `code`, `reproduce`, `test`, `review`, `fix`, `report`) hoặc tạo thêm skill mới nạp vào container `ntchub-agent`. |
| **`frontend-design`** | [`.agents/skills/frontend-design/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/frontend-design/SKILL.md) | - **Ở Giai đoạn M4 (Webapp) & vai `code` khi làm `/new` app:** Giúp agent thiết kế giao diện React tinh tế, rõ ràng, bám sát bố cục `prototype.html` mà không bị rập khuôn. |
| **`vercel-react-best-practices`** | [`.agents/skills/vercel-react-best-practices/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/vercel-react-best-practices/SKILL.md) | - **Ở Giai đoạn M4 (Webapp SPA):** Tối ưu hoá hiệu năng render cho bảng Kanban 5 cột, luồng log WebSocket trực tiếp (`Log trực tiếp`) và trình xem `Diff` mà không bị giật lag. |
| **`web-design-guidelines`** | [`.agents/skills/web-design-guidelines/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/web-design-guidelines/SKILL.md) | - **Ở Giai đoạn M4 (`US-M4-02` → `US-M4-10`):** Kiểm tra UX, trạng thái nút bấm bị khoá kèm tooltip lý do (`AC-M4-03.6`), form nhập API key an toàn (`AC-M4-08.3`). |
| **`webapp-testing`** | [`.agents/skills/webapp-testing/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/webapp-testing/SKILL.md) | - **Ở Giai đoạn M4 & M5:** Viết và chạy kiểm thử tự động trên trình duyệt bằng Playwright để xác minh Dashboard, duyệt Gate trên Webapp và luồng đăng nhập. |
| **`agent-browser`** | [`.agents/skills/agent-browser/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/agent-browser/SKILL.md) | - **Ở Giai đoạn M4 & M5 (và trong container `ntchub-agent`):** Cho phép agent điều khiển trình duyệt headless qua CLI để kiểm tra thực tế URL preview (`*.preview.<domain>`) hoặc tự test giao diện app mới tạo (`/new`). |
| **`deploy-to-vercel`** | [`.agents/skills/deploy-to-vercel/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/deploy-to-vercel/SKILL.md) | - **Ở Giai đoạn M5 (`US-M5-02`, `US-M5-04`, `US-M5-05`):** Tham chiếu quy trình triển khai preview branch và promote/rollback production trên nền tảng Vercel. |

---

### Nhóm 4: Skills từ Felo AI (`felo.ai/vi/skills` — `Felo-Inc/felo-skills`)

| Tên Skill | Đường dẫn | Dùng khi nào & Cho chức năng gì? |
| :--- | :--- | :--- |
| **`felo-search`** | [`.agents/skills/felo-search/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/felo-search/SKILL.md) | - **Ở Giai đoạn M0 (`US-M0-01` → `03`):** Tìm kiếm tài liệu mới nhất về cờ CLI headless, OAuth device flow và định dạng JSON output của `claude`, `codex`, `agy`, `dsh`.<br>- **Trong quá trình code M1–M5:** Tra cứu nhanh breaking changes của thư viện (aiogram 3, Cloudflare API, Caddy API). |
| **`felo-web-fetch`** | [`.agents/skills/felo-web-fetch/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/felo-web-fetch/SKILL.md) | - Dùng kèm với `felo-search` để bóc tách nội dung sạch (Markdown) từ trang tài liệu kỹ thuật hoặc trang GitHub docs mà không bị nhiễu HTML. |
| **`felo-superAgent`** | [`.agents/skills/felo-superAgent/SKILL.md`](file:///d:/vibe-code/ntc-ai-center-hub/.agents/skills/felo-superAgent/SKILL.md) | - Dùng khi cần nghiên cứu đa bước chuyên sâu (ví dụ: tìm hiểu kiến trúc tích hợp mới hoặc hỗ trợ vai `plan` nghiên cứu giải pháp kỹ thuật cho yêu cầu `/new` phức tạp). |

> **Cấu hình `FELO_API_KEY` (khi cần chạy lệnh CLI `felo`):**
> * Trên máy phát triển (Local/VPS): chạy `npm install -g felo-ai` và đặt biến môi trường `FELO_API_KEY="your-key"`.
> * **Lưu ý bảo mật (`🛡 AC-M1-05.2`, `AC-M3-03.2`):** Không bao giờ ghi `FELO_API_KEY` xuống file trong worktree của task.

---

## 4. Mẫu câu lệnh (Prompt) để bạn giao việc cho Agent kèm file này

Khi làm việc với Agent (theo quy trình 5 Gate trong [AGENTS.md](file:///d:/vibe-code/ntc-ai-center-hub/AGENTS.md)), bạn chỉ cần đính kèm `@[AI-skill-help.md]` như sau:

* **Khi lập kế hoạch Module / Phase (Gate 3 & Gate 4):**
  > *"Đọc `@[SPEC.md]`, `@[ARCHITECTURE.md]` và `@[AI-skill-help.md]`. Hãy kích hoạt các skill tương ứng với giai đoạn M1 và lập kế hoạch cho module..."*
* **Khi thực thi Atomic Task (Gate 5):**
  > *"Thực thi task `<MOD>-P01.1.1`. Hãy tra cứu `@[AI-skill-help.md]` và đọc `SKILL.md` của skill phụ trách module này trước khi viết code và test."*
* **Khi đóng gói Docker image `ntchub-agent` (ở M1/M3):**
  > *"Dựa vào `@[AI-skill-help.md]`, hãy copy các skill phù hợp cho từng vai (`interview`, `plan`, `code`, `test`, `review`, `fix`, `report`) vào thư mục `.agents/skills/` bên trong image `ntchub-agent`."*
