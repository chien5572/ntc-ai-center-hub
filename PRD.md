# PRD — ntc-ai-center-hub

> Phiên bản: 0.1 (draft) · Ngày: 2026-10-24 · Chủ sở hữu: chien
> Ký hiệu: **[Owner]** = do chủ dự án quyết định · **[Đề xuất]** = do người viết tài liệu đề xuất, chủ dự án có thể sửa · **[Builder]** = để người xây dựng tự quyết.

---

## 1. Tổng quan

**ntc-ai-center-hub** là một trung tâm điều phối chạy trên máy cá nhân hoặc VPS, nhận yêu cầu phần mềm từ chủ dự án (qua Telegram, CLI hoặc Webapp), **phỏng vấn để làm rõ yêu cầu**, rồi **giao việc cho các AI coding agent** (Claude Code, Codex CLI, Google Antigravity CLI, DeepSeek Harness) theo một quy trình: lên plan → code → test → review → fix → báo cáo → (preview/deploy).

Mục đích: **tự động hoá vibe coding** — chủ dự án chỉ cần mô tả ý tưởng và duyệt ở các điểm quan trọng, có thể làm hoàn toàn từ xa qua điện thoại.

### 1.1 Vì sao
- Các agent CLI đã đủ mạnh để tự code, nhưng việc ngồi trước máy để mớm yêu cầu, chuyển kết quả từ agent này sang agent khác, chạy test, review… vẫn thủ công.
- Chủ dự án đã trả tiền gói đăng ký (Claude, ChatGPT, Google) — muốn tận dụng tối đa quota, kể cả khi không ngồi máy.

### 1.2 Người dùng
- **Một người dùng duy nhất: chủ dự án** [Owner — suy ra từ bối cảnh]. Không có đa người dùng / phân quyền trong phạm vi v1.

---

## 2. Mục tiêu, Non-goals, Anti-goals

### 2.1 Mục tiêu
1. Từ một câu ý tưởng trên Telegram → bot phỏng vấn → spec → hệ thống tự làm đến khi có **Pull Request chạy được + báo cáo ngắn gọn**.
2. Làm việc được trên **cả app mới lẫn repo có sẵn** (thêm tính năng, sửa bug) [Owner].
3. **Chạy nhiều task song song** [Owner].
4. **Cấu hình linh hoạt**: mỗi loại việc gán cho một cặp *(harness, model)* với prompt nạp sẵn; hoạt động được kể cả khi chỉ có **một** harness (ví dụ chỉ Antigravity với nhiều model) [Owner].
5. Điều khiển qua **3 cửa**: Telegram (nhóm có Topics), `hub` CLI trên máy chủ, Webapp [Owner].
6. **Preview và deploy** theo từng project (Docker trên VPS / Vercel / Cloudflare) [Owner].
7. Tận dụng quota hợp lý: chấp nhận đốt nhiều quota cho project lớn, nhưng có cơ chế tối ưu [Owner].

### 2.2 Non-goals (v1)
- Đa người dùng, đội nhóm, phân quyền.
- Tự viết harness/agent riêng — hub chỉ **điều phối** các harness có sẵn.
- Hỗ trợ git host khác GitHub.
- Marketplace prompt, billing.

### 2.3 Anti-goals — coi là THẤT BẠI dù hệ thống vẫn chạy [Owner]
1. **Tự ý push làm hỏng repo.** Không bao giờ được ghi trực tiếp lên nhánh mặc định, không force-push, không merge khi chưa đủ điều kiện.
2. **Báo cáo dài lê thê, không đọc nổi.** Báo cáo trên Telegram phải đọc xong trong < 30 giây trên điện thoại.

(Đốt hết quota trong một đêm **không** phải thất bại [Owner].)

---

## 3. Hành trình người dùng chính

### J1 — App mới từ Telegram
1. Trong topic *General*, gõ `/new Tôi muốn làm app quản lý chi tiêu gia đình`.
2. Hub tạo **topic mới** cho task (ví dụ `#12 · app-chi-tieu`) và bot bắt đầu **phỏng vấn** trong topic đó: mỗi lượt một câu hỏi, có tóm tắt lại định kỳ.
3. Khi thấy đủ, chủ dự án gõ `/go`. Bot gửi **bản tóm tắt spec** (ngắn) + nút *Duyệt / Sửa tiếp*.
4. Sau khi duyệt: hub tạo repo GitHub mới, chạy pipeline. Nếu bật gate *plan*, bot gửi tóm tắt plan để duyệt.
5. Agents code → test → review → fix (lặp có giới hạn).
6. Xong: bot gửi **báo cáo ngắn** + link PR + nút *Preview / Merge / Xem chi tiết*.
7. `/preview` → nhận link chạy thử trên điện thoại. `/merge`, `/deploy prod` (có gate).

### J2 — Tính năng trên repo có sẵn
`/task shop-api thêm đăng nhập bằng Google` → phỏng vấn ngắn hơn (agent đã đọc repo trước khi hỏi) → như J1 từ bước 3, nhưng làm trên branch mới của repo đó.

### J3 — Sửa bug
`/fix shop-api lỗi 500 khi checkout giỏ rỗng` → phỏng vấn tối thiểu (bước tái hiện, log nếu có) → pipeline rút gọn: reproduce → fix → test → review → báo cáo.

### J4 — Duyệt từ xa
Khi tới gate đang bật, bot nhắn trong topic của task: tóm tắt + nút **Duyệt / Từ chối / Góp ý**. Góp ý → agent làm lại chặng đó theo góp ý.

### J5 — Làm việc tại máy chủ
SSH vào VPS: `hub tasks`, `hub logs -f 12`, `hub attach 12` (xem agent làm việc trực tiếp), `hub approve 12`, `hub new "..."` (phỏng vấn ngay trong terminal).

### J6 — Setup lần đầu
Qua Webapp **hoặc** `/setup` trên Telegram **hoặc** `hub setup`: kết nối GitHub → đăng nhập harness (OAuth) → (tuỳ chọn) API key cho DeepSeek Harness → bảng phân việc → prompt → gate mặc định → giới hạn song song.

---

## 4. Yêu cầu chức năng

### 4.1 Tiếp nhận & phỏng vấn (Intake)
- **FR-1** Nhận yêu cầu từ Telegram, CLI, Webapp; ba kênh dùng chung một task (tạo ở kênh này, theo dõi/duyệt ở kênh khác).
- **FR-2** Ba loại yêu cầu: `new` (app mới), `task` (repo có sẵn), `fix` (bug). Mỗi loại có **kịch bản phỏng vấn** (prompt) riêng, cấu hình được.
- **FR-3** Bot phỏng vấn **một câu mỗi lượt**, tóm tắt lại những gì đã hiểu định kỳ, đề xuất mặc định khi chủ dự án nói "tuỳ bạn".
- **FR-4** Với `task`/`fix`: trước khi hỏi, agent phỏng vấn được đọc repo (README, cấu trúc, `AGENTS.md`/`CLAUDE.md`…) ở chế độ chỉ-đọc.
- **FR-5** Chủ dự án chốt bằng `/go` (hoặc nút). Kết quả là file **spec.md** theo template cố định (mục tiêu, phạm vi, non-goals, tiêu chí chấp nhận, ràng buộc kỹ thuật, deploy target).
- **FR-6** Có thể huỷ / tạm dừng phỏng vấn và quay lại sau (phiên được lưu).

### 4.2 Pipeline & điều phối
- **FR-7** Pipeline mặc định:
  - `new`/`task`: **Spec → Plan → Code → Test → Review → (Fix ↺ Test/Review) → Report → [Preview] → [Merge] → [Deploy]**
  - `fix`: **Spec → Reproduce → Fix → Test → Review → (Fix ↺) → Report → …**
- **FR-8** Pipeline được định nghĩa bằng cấu hình (YAML) và chỉnh được trên Webapp; v1 cho phép bật/tắt chặng và đổi thứ tự chặng trong khuôn mẫu, chưa cần trình soạn thảo đồ thị tự do [Đề xuất].
- **FR-9** Vòng Fix có **giới hạn số vòng** (mặc định 3). Vượt giới hạn → dừng, báo chủ dự án kèm tóm tắt lý do.
- **FR-10** **Test được hub xác minh độc lập**: ngoài việc agent tự chạy test, hub tự chạy lệnh test của project (khai báo trong cấu hình project) và lấy kết quả đó làm chuẩn — không tin tuyệt đối vào lời agent.
- **FR-10a** [Đề xuất] **Lệnh test do ai đặt**: với app mới, `test_cmd` được ghi trong **spec** (agent phỏng vấn đề xuất, chủ dự án thấy khi duyệt spec); với repo có sẵn, hub tự dò (`package.json`, `pyproject.toml`, `Makefile`…) và hiển thị trong spec để xác nhận. Sau khi duyệt, `test_cmd` được lưu ở cấu hình project và **agent không sửa được** (chỉ hub/chủ dự án).
- **FR-10b** [Đề xuất] Vai **test** vừa **viết test** cho phần mới (theo tiêu chí chấp nhận trong spec) vừa chạy; review kiểm tra test có thực chất không. Project **không có test** thì **không bao giờ tự merge** — bắt buộc chủ dự án `/merge` thủ công, báo cáo ghi rõ "⚠ không có test".
- **FR-11** Review trả về **kết luận có cấu trúc** (`approve` / `changes_requested` + danh sách vấn đề có mức độ). `changes_requested` → vào Fix.
- **FR-12** Các chặng trao đổi qua **artifact file** trong thư mục làm việc (`spec.md`, `plan.md`, `test-report.md`, `review.json`, …) để agent khác harness vẫn đọc được.
- **FR-13** Khởi động lại hub không mất task: task tiếp tục từ chặng hoàn tất gần nhất.

### 4.3 Agents, model & prompt
- **FR-14** Hỗ trợ 4 harness qua lớp *adapter* chung; thêm harness mới = thêm adapter:
  - **Claude Code, Codex CLI, Antigravity CLI** — dùng gói đăng ký (OAuth);
  - **DeepSeek Harness (`dsh`)** — harness dành cho **mọi model dùng API key** (DeepSeek và các nhà cung cấp API khác) [Owner].
- **FR-15** Xác thực chính bằng **OAuth của gói đăng ký** của từng harness [Owner]; harness tự giữ phiên đăng nhập trên máy chạy hub. **API key chỉ là bổ trợ** và **chỉ được cắm vào DeepSeek Harness** — không cắm API key vào Claude Code/Codex/Antigravity [Owner].
- **FR-15a** Hub **chỉ gọi harness qua CLI chính chủ** ở chế độ headless (`claude -p`, `codex exec`, `agy -p`, `dsh --profile headless`), **không dùng SDK** của các harness [Owner] — để dùng gói đăng ký đúng cách mà nhà cung cấp cho phép.
- **FR-16** **Bảng phân việc (routing)**: mỗi *vai* (interview, plan, code, test, review, fix, reproduce, report) → *(harness, model, prompt, giới hạn thời gian)*. Danh sách model lấy từ harness, không cứng trong code. Một harness có thể phục vụ mọi vai với các model khác nhau [Owner].
- **FR-17** Có thể **ghi đè routing theo project** và theo task.
- **FR-18** **Prompt nạp sẵn** cho từng vai, có biến (`{{spec}}`, `{{plan}}`, `{{repo_summary}}`…), chỉnh được trên Webapp / `/prompt` / `hub prompt`. Có phiên bản (version) để quay lại.
- **FR-19** Hiển thị **trạng thái từng harness**: đăng nhập OK / hết hạn / đang bị rate-limit, số task đang chạy. Phiên hết hạn → bot nhắn yêu cầu đăng nhập lại.
- **FR-20** **Dự phòng (fallback)** cho mỗi vai: danh sách *(harness, model)* thay thế khi gặp rate-limit/hết quota — có thể là harness đăng ký khác, hoặc **DeepSeek Harness với API key**. Bật/tắt được.

### 4.4 Điểm duyệt (Gates)
- **FR-21** Các gate: `spec`, `plan`, `merge`, `deploy_staging`, `deploy_prod`. Mỗi gate có công tắc **bật/tắt** mặc định (cấu hình toàn cục, theo project) [Owner].
- **FR-22** Ghi đè cho từng task: `/auto on|off` hoặc nút trong Telegram [Owner].
- **FR-23** Khi tới gate bật: task chuyển trạng thái *chờ duyệt*, bot gửi tóm tắt + nút **Duyệt / Từ chối / Góp ý**. Duyệt được từ bất kỳ kênh nào.
- **FR-24** Mặc định [Owner đồng ý đề xuất]: `spec` bật, `plan` bật, `merge` bật, `deploy_staging` tắt (vì deploy luôn do lệnh `/deploy` khởi động — lệnh đó đã là sự đồng ý), `deploy_prod` **bật và không thể tắt toàn cục** (chỉ tắt được theo project một cách tường minh).
- **FR-24a** [Owner] Gate `merge` **tắt** ⇒ hub **tự merge** khi đủ điều kiện (test của hub pass + review `approve` + có test). Gate `merge` **bật** mà chủ dự án chưa merge ⇒ task ở trạng thái **`awaiting_merge`** ("xong, chờ merge"), **không giữ slot** chạy.
- **FR-24b** [Owner] **Deploy không bao giờ tự chạy sau merge** — chỉ bắt đầu từ lệnh `/deploy` (hoặc nút tương ứng), rồi đi qua gate deploy của môi trường đó.

### 4.5 Song song & quota
- **FR-25** Nhiều task chạy song song [Owner]. Giới hạn đồng thời cấu hình theo **harness** và theo **(harness, model)**. Vượt → xếp hàng.
- **FR-26** Tối ưu quota [Owner, chi tiết là Đề xuất]:
  - phân tầng model: model rẻ/nhanh cho phỏng vấn, test, báo cáo; model mạnh cho plan/code/review;
  - tái sử dụng phiên (session resume) trong cùng chặng/vòng fix thay vì nạp lại toàn bộ ngữ cảnh;
  - cache *repo summary* theo commit để không phải đọc lại repo mỗi task;
  - giới hạn vòng fix; dừng sớm khi cùng một lỗi lặp lại 2 lần;
  - khi bị rate-limit: tạm hoãn task và tự chạy lại khi quota hồi (hoặc dùng fallback nếu bật);
  - thống kê thời gian/lượt gọi/token (nếu harness trả về) theo task, chặng, harness.

### 4.6 An toàn Git (anti-goal #1)
- **FR-27** Mỗi task làm trong **thư mục/branch riêng** (`hub/<task-id>-<slug>`), git worktree tách biệt.
- **FR-28** **Agent không giữ credential push.** Chỉ hub được push, và chỉ push lên branch của task. Hub chặn: push lên nhánh mặc định, force-push, xoá branch không phải của hub, sửa cấu hình repo.
- **FR-29** Mọi thay đổi vào nhánh mặc định **chỉ qua Pull Request**. Merge chỉ khi: test (do hub chạy) pass + review `approve` + gate `merge` đã duyệt (hoặc tắt).
- **FR-30** Với repo mới, hub bật branch protection cho nhánh mặc định (nếu gói GitHub cho phép).
- **FR-31** Mọi thao tác git/GitHub của hub được ghi nhật ký (audit log).

### 4.7 Preview & Deploy
- **FR-32** Mỗi project khai báo **deploy target** riêng: `docker` (trên chính VPS), `vercel`, `cloudflare` [Owner]. Với app mới, agent plan đề xuất target trong spec.
- **FR-33** `/preview [id]`: dựng bản chạy thử từ branch của task, trả link. Docker → container tạm + reverse proxy subdomain `<task>.preview.<domain>`; Vercel/Cloudflare → preview deployment của nền tảng.
- **FR-33a** [Owner] Preview Docker **không công khai** mặc định: đặt sau **Cloudflare Tunnel + Access**, chỉ chủ dự án mở được. Domain là tham số cài đặt (`hub setup`, DNS quản lý trên Cloudflare), không phải quyết định thiết kế.
- **FR-34** Preview tự tắt sau thời gian cấu hình (mặc định 24h) hoặc khi `/down`, khi task đóng.
- **FR-35** `/deploy <project> [staging|prod]`, `/rollback <project> [env]`. Deploy đi qua gate tương ứng.

### 4.8 Báo cáo (anti-goal #2)
- **FR-36** **Báo cáo Telegram** tối đa ~10 dòng, cố định format:
  ```
  ✅ #12 app-chi-tieu — HOÀN THÀNH (1h42m)
  PR: <link>  · Preview: <link>
  Đã làm: 3 màn hình, API CRUD chi tiêu, đăng nhập email
  Test: 48/48 pass · Review: approve (2 góp ý nhỏ đã sửa)
  ⚠ Cần bạn xem: chưa có gửi email thật (đang mock)
  Quota: Opus 12 lượt · Sonnet 41 lượt
  [Preview] [Merge] [Chi tiết]
  ```
- **FR-37** Báo cáo chi tiết (markdown) nằm trong mô tả PR và trên Webapp, không gửi nguyên văn vào Telegram.
- **FR-38** Tin nhắn tiến độ trong topic: chỉ khi **chuyển chặng**, **chờ duyệt**, **lỗi**, **xong**. Không spam log; log xem bằng `/logs`.

### 4.9 Telegram
- **FR-39** Hoạt động trong **một nhóm Telegram bật Topics** [Owner]. Topic *General* cho lệnh hệ thống; **mỗi task một topic** tự tạo; trong topic không cần gõ `id`.
- **FR-40** Chỉ chấp nhận lệnh từ Telegram user ID của chủ dự án; bỏ qua mọi người khác trong nhóm.
- **FR-41** Lệnh setup có dữ liệu nhạy cảm (`/github`, `/apikey`) **chỉ chạy trong chat riêng** với bot; bot **xoá tin nhắn chứa secret** ngay sau khi lưu.
- **FR-42** Bộ lệnh:

| Nhóm | Lệnh |
|---|---|
| Tạo việc | `/new <ý tưởng>` · `/task <project> <yêu cầu>` · `/fix <project> <bug>` · `/go` |
| Theo dõi | `/tasks` · `/status [id]` · `/logs [id]` · `/report [id]` |
| Điều khiển | `/pause` · `/resume` · `/cancel [id]` · `/approve [id]` · `/reject [id] <góp ý>` · `/auto [id] on\|off` |
| Preview/Deploy | `/preview [id]` · `/down [id]` · `/merge [id]` · `/deploy <project> [env]` · `/rollback <project> [env]` |
| Setup | `/setup` · `/github` · `/login <harness>` · `/apikey <provider>` · `/route` · `/prompt <vai>` · `/gates` · `/limits` · `/config` |
| Hệ thống | `/projects` · `/agents` · `/help` |

- **FR-43** `/login <harness>` hoạt động khi harness hỗ trợ đăng nhập bằng device code / URL (bot gửi link + mã). Harness nào không hỗ trợ → hướng dẫn đăng nhập qua `hub login` (SSH) hoặc Webapp.

### 4.10 `hub` CLI
- **FR-44** Có lệnh tương ứng với mọi lệnh Telegram (`hub new`, `hub task`, `hub fix`, `hub tasks`, `hub approve`, `hub deploy`, `hub config`, …).
- **FR-45** Thêm: `hub logs -f <id>` (stream log), `hub attach <id>` (xem trực tiếp output agent), phỏng vấn tương tác ngay trong terminal, `hub doctor` (kiểm tra cài đặt, đăng nhập harness, kết nối GitHub/Telegram).
- **FR-46** `hub config export|import` cấu hình dạng YAML (không chứa secret).

### 4.11 Webapp
- **FR-47** Dashboard: task đang chạy / chờ duyệt / hàng đợi; chi tiết task (timeline chặng, artifact, log trực tiếp, diff, báo cáo); duyệt gate.
- **FR-48** Cấu hình: GitHub, harness (trạng thái đăng nhập, model khả dụng), API key, bảng phân việc, prompt (có version), pipeline, gate, giới hạn, project (repo, lệnh test, deploy target).
- **FR-49** Truy cập được từ xa một cách an toàn (xem Kiến trúc §Bảo mật); có đăng nhập.

---

## 5. Yêu cầu phi chức năng
- **NFR-1 Bảo mật:** secret mã hoá khi lưu; không log secret; agent chạy với quyền hạn chế (không có token push, tuỳ chọn sandbox Docker).
- **NFR-2 Độ bền:** trạng thái lưu bền (SQLite); khởi động lại không mất task; agent treo quá thời gian cho phép → kill và đánh dấu lỗi.
- **NFR-3 Quan sát:** log theo task/chặng/agent, lưu lại để xem lại; audit log thao tác git/deploy.
- **NFR-4 Cài đặt:** một lệnh cài trên Linux VPS; chạy được trên máy cá nhân (Linux/macOS; Windows qua WSL2) [Đề xuất].
- **NFR-5 Phản hồi:** lệnh Telegram được phản hồi < 3 giây (kể cả chỉ là "đã nhận").
- **NFR-6 Khả năng mở rộng:** thêm harness, deploy target, kênh chat mới mà không sửa lõi điều phối.

---

## 6. Chỉ số thành công
- Từ `/new` đến PR chạy được cho một app nhỏ **không cần chủ dự án ngồi máy**.
- **0** lần thay đổi vào nhánh mặc định ngoài PR đã đủ điều kiện.
- Báo cáo Telegram ≤ 10 dòng trong 100% trường hợp.
- ≥ 80% task nhỏ/vừa kết thúc ở trạng thái "xong" mà không cần can thiệp ngoài các gate.

---

## 7. Phân kỳ [Đề xuất]

| Giai đoạn | Nội dung |
|---|---|
| **M1 — Lõi** | Daemon, SQLite, task/pipeline engine, adapter **Antigravity** + **Claude Code** (CLI), container cách ly agent, git worktree + quy tắc an toàn, `hub` CLI cơ bản, pipeline `task`/`fix` trên repo có sẵn |
| **M2 — Telegram** | Bot + Topics, phỏng vấn, gate có nút, báo cáo ngắn, lệnh theo dõi/điều khiển |
| **M3 — App mới + Codex + DeepSeek Harness** | Tạo repo mới, pipeline `new`, adapter Codex, adapter `dsh` cho API key, fallback, giới hạn song song theo model |
| **M4 — Webapp** | Dashboard, cấu hình routing/prompt/gate/project, log trực tiếp |
| **M5 — Preview/Deploy** | Docker + Caddy preview, Vercel, Cloudflare; deploy/rollback; setup đầy đủ qua Telegram |

---

## 8. Câu hỏi mở
- Domain cụ thể cho Webapp/preview — nhập lúc cài đặt.
- Xác minh khi bắt đầu M1: cờ headless/model/output JSON thực tế của `claude`, `codex`, `agy`, `dsh`; cách đăng nhập OAuth trên máy không có trình duyệt; cách `dsh` cấu hình nhà cung cấp API.
