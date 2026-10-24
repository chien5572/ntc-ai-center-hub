# SPEC — ntc-ai-center-hub

> Phiên bản: 0.3 (draft) · Ngày: 2026-10-24 · Đi kèm: [PRD.md](./PRD.md) · [ARCHITECTURE.md](./ARCHITECTURE.md) · [prototype.html](./prototype.html)
> Ký hiệu nguồn (giữ nguyên quy ước của PRD): **[Owner]** chủ dự án quyết · **[Đề xuất]** người viết đề xuất, Owner có thể sửa · **[Builder]** người xây dựng tự quyết · **[Proto]** chi tiết rút ra từ prototype.

---

## 0. Cách đọc tài liệu

- PRD trả lời **làm gì / vì sao**, ARCHITECTURE trả lời **làm thế nào**. SPEC này chia thành **giai đoạn bàn giao được** (M0 → M5); mỗi giai đoạn gồm:
  - **Mục tiêu** và **phạm vi / ngoài phạm vi**;
  - **User Story** (`US-Mx-yy`): *"Là chủ dự án, tôi muốn … để …"*, có mức ưu tiên **P0** (bắt buộc để qua giai đoạn) / **P1** (nên có) / **P2** (có thể dời);
  - **Acceptance Criteria** (`AC-Mx-yy.z`): kiểm thử được, viết dạng *Biết rằng / Khi / Thì* hoặc dạng kiểm tra trực tiếp;
  - **Điều kiện hoàn thành giai đoạn** (Exit criteria) và **kịch bản demo**.
- Người dùng duy nhất là **chủ dự án** (gọi tắt "tôi" trong các story).
- Mọi AC liên quan đến **anti-goal** (§2.3 PRD) được đánh dấu 🛡 — đây là các AC **không được phép nới lỏng** khi trễ tiến độ.
- Tên model trong ví dụ (`claude-opus-5.5`, `gemini-3.8-flash`, …) chỉ để minh hoạ, giống prototype.

---

## 1. Quyết định làm rõ (giải quyết chỗ mơ hồ / lệch giữa PRD, ARCHITECTURE, prototype)

| ID | Chủ đề | Quyết định trong SPEC | Nguồn |
|---|---|---|---|
| D-01 | Kênh tạo task ở M1 (chưa có Telegram) | M1 tạo task qua `hub task` / `hub fix` với **phỏng vấn tương tác trong terminal**; thêm tuỳ chọn `--spec <file>` để bỏ qua phỏng vấn khi đã có spec viết sẵn. | [Đề xuất] |
| D-02 | Ý nghĩa `/auto on` | Với task đó, bỏ qua gate `spec`, `plan`, `merge`, `deploy_staging`. Bỏ qua gate `spec` nghĩa là **không gửi bản tóm tắt spec để duyệt**, nhưng `spec.md` vẫn được sinh, lưu và **xem được bất cứ lúc nào khi tôi yêu cầu** (`/spec`, `hub spec`, nút 📄 Spec, Webapp). Tương tự với plan (`/plan`). **Không bao giờ** bỏ qua `deploy_prod`. **Không** tự merge nếu project không có test (FR-10b). | [Owner] + PRD FR-10b, FR-24 |
| D-03 | "Góp ý" tại từng gate | `spec` → task về `interviewing`, góp ý được nối vào phỏng vấn, `/go` lại để sinh spec mới. `plan` → chạy lại chặng plan với góp ý trong ngữ cảnh, rồi hỏi duyệt lại. `awaiting_merge` → task về `queued`, chạy chặng `fix` với góp ý, rồi test/review lại. Gate deploy không có góp ý (chỉ Duyệt/Từ chối). | [Proto] + ARCH §5.2 |
| D-04 | "Từ chối" tại từng gate | `spec`/`plan` → task `cancelled`, **giữ lại branch** để xem. Gate deploy → huỷ lần deploy đó, task không đổi trạng thái. | [Proto] |
| D-05 | Harness hết hạn đăng nhập giữa chừng | Nếu route có fallback **và** fallback đang bật → chuyển sang fallback, nhắn **một lần**. Nếu không → task vào `rate_limited` với `block_reason=auth_expired` (xem §2.1), nhắn yêu cầu `/login <harness>`; tự quay lại `queued` khi đăng nhập OK. Prototype tự fallback vô điều kiện — SPEC **không** làm vậy. | [Owner] |
| D-06 | `/deploy` trong topic task | Trong topic của task, `/deploy [env]` tự hiểu project của task. Deploy luôn lấy **HEAD của nhánh mặc định** (tức là chỉ deploy code đã merge); task chưa merge → từ chối với thông báo "Cần merge trước khi deploy". | [Proto] + FR-24b |
| D-07 | Tính nhất quán của review | Hub tự kiểm tra `review.json`: nếu `verdict=approve` mà có issue `blocker`/`major`, hoặc ≥ 3 issue `minor` → hub coi là `changes_requested` và ghi cảnh báo. | [Proto — prompt review] + [Đề xuất] |
| D-08 | Đếm vòng fix | `max_rounds` = số lần chạy chặng `fix` **sau** lần test/review đầu tiên. Mặc định 3 ⇒ tối đa 3 vòng fix; lần thất bại thứ 4 ⇒ `failed`. | [Proto] |
| D-09 | Gợi ý câu trả lời khi phỏng vấn | Bot có thể gửi tối đa 3 **nút gợi ý** (inline keyboard) kèm câu hỏi; bấm nút = gửi câu trả lời đó. Gõ tự do luôn được chấp nhận. | [Proto — chips] |
| D-10 | Tin nhắn trạng thái | Mỗi topic task có **một tin nhắn trạng thái được ghim**, được **sửa (edit)** khi đổi chặng/trạng thái, thay vì gửi tin mới. | ARCH §10 + [Proto] |
| D-11 | Định danh task & topic | `task.id` là số nguyên tăng dần toàn hệ thống. Tên topic: `#<id> · <slug>`; branch: `hub/<id>-<slug>`. | PRD FR-27 + [Proto] |
| D-12 | Nút Merge khi chưa đủ điều kiện | Nút/lệnh merge luôn hiện nhưng hub **từ chối** khi task không ở `awaiting_merge`, kèm lý do cụ thể. | [Proto] |
| D-13 | Báo cáo / tin nhắn dài trên Telegram | **Tin 1 (tin chính)** luôn ≤ 10 dòng, đúng format FR-36, tự đủ nghĩa (chỉ đọc tin này là biết kết quả + việc cần làm) và mang các nút. Nội dung không vừa (danh sách "Đã làm" đầy đủ, nhiều mục "Cần bạn xem", …) được **chia thành các tin tiếp theo** theo quy tắc §2.5 — không cắt cụt, không ngắt giữa nội dung. Áp dụng cho mọi tin dài (báo cáo, tóm tắt spec/plan, tin lỗi). | [Owner] cụ thể hoá FR-36 |
| D-14 | Thêm giai đoạn M0 | Thêm **M0 — Spike xác minh harness** trước M1 để đóng câu hỏi mở §8 PRD (cờ headless, OAuth không trình duyệt, cấu hình `dsh`). | [Đề xuất] |
| D-15 | Tắt chặng `test` / `review` trong pipeline | **Cho phép tắt**, nhưng: (a) hub **luôn** tự chạy `test_cmd` trước khi merge — bước xác minh này không tắt được; (b) task có chặng `test` hoặc `review` bị tắt **không bao giờ tự merge** (luôn dừng ở `awaiting_merge`, báo cáo ghi "⚠ đã bỏ qua chặng test/review"), kể cả khi gate `merge` tắt hoặc `/auto on`. Lý do: vẫn linh hoạt cho việc nhỏ mà không phá điều kiện merge của FR-29. | [Đề xuất — Owner giao chọn] |
| D-16 | Branch `hub/*` sau khi xong | Merge thành công ⇒ hub **xoá branch remote** `hub/<id>-<slug>` ngay (PR vẫn giữ toàn bộ commit, GitHub cho khôi phục branch từ PR). Task `cancelled`/`failed` ⇒ **giữ** branch remote để tôi xem; xoá khi dọn worktree (mặc định sau 7 ngày) hoặc khi gõ `/cleanup [id]`. Hub chỉ xoá branch do chính nó tạo (FR-28); mỗi lần xoá ghi `audit_log`. | [Đề xuất — Owner giao chọn] |
| D-17 | Truy cập từ xa (Webapp & preview) | Dùng **Cloudflare Tunnel + Cloudflare Access**, không dùng Tailscale. `hubd` và Caddy chỉ nghe nội bộ; `cloudflared` đưa `hub.<domain>` → Webapp và `*.preview.<domain>` → Caddy; Access policy chỉ cho email của chủ dự án. Không mở port công khai. Domain là tham số cài đặt. | [Owner] |

---

## 2. Mô hình tham chiếu chung

### 2.1 Trạng thái task
`interviewing | queued | running | waiting_approval | paused | rate_limited | failed | awaiting_merge | done | cancelled` (ARCH §4, §5.2).

Trường **`TASK.block_reason`** [Owner] — chỉ có giá trị khi `status = rate_limited`, ngược lại là `NULL`:

| `block_reason` | Khi nào | Tự thoát khi | Thông báo |
|---|---|---|---|
| `rate_limit` | Harness/model báo hết quota hoặc rate-limit, không có fallback khả dụng | tới `rate_limited_until` | "⏳ #id chờ quota <harness · model> tới HH:MM" |
| `auth_expired` | Phiên đăng nhập harness hết hạn, không có fallback khả dụng | harness đăng nhập lại OK | "🔴 #id chờ đăng nhập <harness> — gõ /login <harness>" |

Kèm theo `TASK.blocked_harness` (và `blocked_model` nếu chặn theo model) để biết task đang chờ harness nào. `/status`, `hub status`, Webapp hiển thị trạng thái kèm lý do, ví dụ "chờ harness — hết hạn đăng nhập antigravity".

Bất biến:
- `awaiting_merge`, `waiting_approval`, `paused`, `rate_limited`, `interviewing` **không giữ slot** của Scheduler [Owner cho `awaiting_merge`; Đề xuất cho các trạng thái còn lại].
- Mọi chuyển trạng thái được **ghi vào SQLite trước** khi thực hiện tác dụng phụ (ARCH §1.5).

### 2.2 Vai (role) & artifact

| Vai | Đọc | Ghi | Chế độ |
|---|---|---|---|
| interview | repo (chỉ-đọc, với `task`/`fix`), lịch sử hội thoại | `spec.md` (khi `/go`) | read_only |
| plan | `spec.md`, `repo_summary` | `plan.md` | write (chỉ artifact) |
| code | `spec.md`, `plan.md` | mã nguồn | write |
| reproduce | `spec.md` | test tái hiện lỗi | write |
| test | `spec.md`, diff | test mới + `test-report.md` | write |
| review | `spec.md`, `plan.md`, diff, kết quả test của hub | `review.json` | read_only với mã nguồn |
| fix | `test-report.md`, `review.json`, góp ý | mã nguồn | write |
| report | dữ liệu có cấu trúc | `REPORT.md`, tin Telegram | read_only |

### 2.3 Template `spec.md` (FR-5)
```markdown
# Spec #<id> — <tiêu đề>
## Mục tiêu
## Phạm vi (in scope)
## Ngoài phạm vi (non-goals)
## Tiêu chí chấp nhận        <!-- danh sách đánh số, kiểm thử được -->
## Ràng buộc kỹ thuật        <!-- stack, thư viện, hạ tầng -->
## Lệnh test                 <!-- test_cmd: … (đề xuất hoặc dò được) -->
## Deploy target             <!-- docker | vercel | cloudflare | none -->
## Repo                      <!-- new: owner/name (private) | task/fix: owner/name @ default_branch -->
```

### 2.4 Gate mặc định (FR-21, FR-24)

| Gate | Mặc định | Tắt toàn cục? | Ghi chú |
|---|---|---|---|
| `spec` | bật | được | |
| `plan` | bật | được | |
| `merge` | bật | được | Tắt ⇒ tự merge khi đủ điều kiện (FR-24a) |
| `deploy_staging` | tắt | được | Lệnh `/deploy` đã là sự đồng ý |
| `deploy_prod` | bật | **không** 🔒 | Chỉ tắt được tường minh theo project |

### 2.5 Quy tắc chia tin nhắn dài trên Telegram (D-13) [Owner]

**Mục tiêu:** không bao giờ cắt cụt nội dung, nhưng tin đầu tiên vẫn đọc xong trong < 30 giây (anti-goal #2).

1. **Tin chính (phần 1)** luôn tự đủ nghĩa, ≤ 10 dòng: tiêu đề + trạng thái, link PR/Preview, 1 dòng "Đã làm" dạng tóm tắt, Test · Review, số mục "Cần bạn xem" (ví dụ "⚠ Cần bạn xem: 4 mục ↓"), Quota, các nút. Chỉ đọc tin này là đủ để quyết định.
2. **Các phần tiếp theo** chỉ chứa phần chi tiết không vừa tin chính, chia theo thứ tự ưu tiên ranh giới:
   1. ranh giới **mục** (ví dụ "⚠ Cần bạn xem", "Đã làm — chi tiết", "Review — góp ý đã sửa"): mỗi mục bắt đầu ở đầu một tin;
   2. nếu một mục vẫn quá dài: ranh giới **gạch đầu dòng**;
   3. nếu một gạch đầu dòng vẫn quá dài: ranh giới **câu**;
   4. **không bao giờ** ngắt giữa câu, giữa khối code, giữa link, giữa bảng.
3. Mỗi phần tiếp theo ≤ 15 dòng và ≤ 3.500 ký tự (dưới giới hạn 4.096 của Telegram), có tiêu đề `(2/3) <tên mục>`; tin chính có hậu tố `(1/3)` khi có nhiều phần.
4. Mục bị chia qua nhiều tin lặp lại tiêu đề kèm "(tiếp)".
5. Tối đa **5 phần**. Dài hơn ⇒ phần cuối là "📄 Phần còn lại: xem `REPORT.md`" kèm nút gửi file — vẫn không cắt giữa nội dung, chỉ dừng ở ranh giới mục.
6. Các phần được gửi liền nhau, ngay sau tin chính; chỉ tin chính mang nút bấm.

Ví dụ:
```
(1/2) ✅ #12 app-chi-tieu — HOÀN THÀNH (1h42m)
PR: <link> · Preview: <link>
Đã làm: 3 màn hình, API CRUD, đăng nhập email (+5 mục ↓)
Test: 48/48 pass · Review: approve (2 góp ý nhỏ đã sửa)
⚠ Cần bạn xem: 4 mục ↓
Quota: Opus 12 lượt · Sonnet 41 lượt
[Preview] [Merge] [Chi tiết]

(2/2) ⚠ Cần bạn xem
• Chưa gửi email thật (đang mock)
• …
Đã làm — chi tiết
• …
```

---

## 3. Giai đoạn M0 — Spike xác minh harness

**Mục tiêu:** biến các giả định "[Cần xác minh]" trong ARCH §6.2 thành dữ kiện, trước khi viết adapter.
**Phạm vi:** Claude Code, Codex CLI, Antigravity CLI, DeepSeek Harness; image `ntchub-agent`.
**Ngoài phạm vi:** code production của hub.

### US-M0-01 · Ma trận harness headless (P0)
*Là chủ dự án, tôi muốn biết chắc cách gọi headless của từng harness để adapter không xây trên giả định sai.*
- **AC-M0-01.1** Có `docs/harness-matrix.md` ghi cho **mỗi** harness: lệnh headless, cờ chọn model, định dạng output (stream JSON hay text), cách resume phiên, cách liệt kê model, exit code / thông báo khi hết quota / rate-limit / hết hạn đăng nhập, phiên bản CLI đã thử.
- **AC-M0-01.2** Có script smoke `scripts/smoke-<harness>.sh`: chạy harness trong container `ntchub-agent` với prompt "tạo file hello.txt chứa 'ok'" và thoát 0, file được tạo trong thư mục mount.
- **AC-M0-01.3** Với mỗi harness, ghi lại **mẫu output thật** (đã lược secret) vào `tests/fixtures/<harness>/` để dùng làm dữ liệu test cho parser của adapter.

### US-M0-02 · Đăng nhập OAuth trên máy không trình duyệt (P0)
*Là chủ dự án, tôi muốn đăng nhập gói đăng ký của từng harness trên VPS không có trình duyệt.*
- **AC-M0-02.1** Với mỗi harness OAuth, tài liệu ghi rõ một trong hai: (a) có device code / URL ⇒ ghi cách lấy; (b) không có ⇒ ghi quy trình thay thế (ví dụ đăng nhập trên máy cá nhân rồi chép credential, hoặc SSH port-forward).
- **AC-M0-02.2** Credential được lưu dưới thư mục riêng của user `ntchub`, mount vào container thì harness dùng được mà không hỏi lại.
- **AC-M0-02.3** Khi credential hết hạn, harness ở chế độ headless **thoát với lỗi trong ≤ 30 giây**, không treo chờ nhập — hoặc ghi rõ cách hub phát hiện treo.

### US-M0-03 · Cấu hình DeepSeek Harness bằng API key (P0)
- **AC-M0-03.1** Tài liệu ghi cách tạo profile `ntchub` cho `dsh` với provider/model/key truyền qua biến môi trường.
- **AC-M0-03.2** Chứng minh key **chỉ** tồn tại trong env của container `dsh`, không ghi xuống worktree.

**Exit M0:** ma trận đủ 4 harness, 4 script smoke xanh, các câu hỏi mở §8 PRD có câu trả lời hoặc được ghi lại là rủi ro kèm phương án.

---

## 4. Giai đoạn M1 — Lõi

**Mục tiêu:** từ terminal trên VPS, giao một việc (`task`/`fix`) trên **repo có sẵn**, hub tự chạy pipeline đến **PR chạy được + báo cáo**, an toàn tuyệt đối với repo.
**Phạm vi:** `hubd`, SQLite, pipeline engine, Scheduler, Interview service (kênh CLI), adapter **Claude Code** + **Antigravity**, container cách ly agent, Git/GitHub service, `hub` CLI cơ bản, prompt mặc định, Report service (mức PR + CLI).
**Ngoài phạm vi:** Telegram, Webapp, app mới (`new`), Codex, `dsh`, fallback, preview/deploy.

### US-M1-01 · Cài đặt & daemon (P0)
*Là chủ dự án, tôi muốn cài hub bằng một lệnh trên VPS để không phải dựng tay từng thành phần.*
- **AC-M1-01.1** Trên Ubuntu LTS sạch, chạy script cài đặt một lệnh ⇒ tạo user `ntchub`, cài `hubd` + `hub`, systemd unit `ntc-hub.service` được bật và chạy (NFR-4).
- **AC-M1-01.2** `hub doctor` kiểm tra và in ✓/✗ cho: hubd đang chạy, phiên bản + trạng thái đăng nhập từng harness đã cài, quyền GitHub (GitHub App hoặc PAT), Docker + image `ntchub-agent`, dung lượng đĩa trống. Mỗi ✗ có kèm **lệnh gợi ý sửa** (ví dụ `hub login antigravity`). [Proto]
- **AC-M1-01.3** `hub` giao tiếp với `hubd` qua Unix socket `~/.ntc-hub/hubd.sock`, không cần token khi cùng user (ARCH §11).
- **AC-M1-01.4** Chạy được trên máy cá nhân Linux/macOS, và Windows qua WSL2 (smoke test: tạo task `fix` trên repo mẫu đến PR).

### US-M1-02 · Kết nối GitHub & đăng ký project có sẵn (P0)
*Là chủ dự án, tôi muốn khai báo một repo có sẵn làm project để giao việc cho nó.*
- **AC-M1-02.1** `hub github connect` lưu thông tin GitHub App (hoặc fine-grained PAT) vào `secrets.enc` đã mã hoá; secret không xuất hiện trong log hay `hub config export`.
- **AC-M1-02.2** `hub project add <owner/repo> [--slug shop-api]` ⇒ tạo mirror clone trong `~/.ntc-hub/repos/<slug>/`, ghi `default_branch`.
- **AC-M1-02.3** Hub **tự dò** `test_cmd` từ `package.json`, `pyproject.toml`, `Makefile`, … (FR-10a); dò không ra ⇒ để trống và đánh dấu "⚠ không có test".
- **AC-M1-02.4** `hub projects` liệt kê slug, repo, nhánh mặc định, `test_cmd`, deploy target.

### US-M1-03 · Phỏng vấn trong terminal & sinh spec (P0)
*Là chủ dự án, tôi muốn mô tả yêu cầu bằng một câu rồi được hỏi để làm rõ, thay vì tự viết spec dài.*
- **AC-M1-03.1** `hub task shop-api "thêm đăng nhập Google"` ⇒ tạo task `kind=task`, trạng thái `interviewing`, mở phiên phỏng vấn tương tác trong terminal.
- **AC-M1-03.2** Trước câu hỏi đầu tiên, agent phỏng vấn đọc repo ở chế độ **chỉ-đọc** (README, cấu trúc, `AGENTS.md`/`CLAUDE.md`) (FR-4). Kiểm tra: sau phỏng vấn, `git status` của bản clone không có thay đổi.
- **AC-M1-03.3** Mỗi lượt bot hỏi **đúng một câu** (FR-3). Kiểm tra tự động: tin nhắn bot chứa ≤ 1 dấu "?" ở cấp câu hỏi chính (heuristic, được phép kèm tóm tắt).
- **AC-M1-03.4** Sau tối đa 5 lượt hỏi, bot gửi **tóm tắt** những gì đã hiểu. Khi tôi trả lời "tuỳ bạn", bot nêu rõ mặc định nó chọn.
- **AC-M1-03.5** `/go` (hoặc `hub go <id>`) ⇒ sinh `spec.md` đúng template §2.3 (đủ 8 mục, mục "Tiêu chí chấp nhận" có ≥ 1 mục đánh số) và mục "Lệnh test" hiển thị `test_cmd` đã dò để tôi xác nhận.
- **AC-M1-03.6** Ctrl+C hoặc `hub pause <id>` khi đang phỏng vấn ⇒ phiên được lưu; `hub attach <id>` tiếp tục đúng chỗ, giữ nguyên lịch sử (FR-6).
- **AC-M1-03.7** `hub fix shop-api "lỗi 500 khi checkout giỏ rỗng"` ⇒ kịch bản phỏng vấn `fix` (hỏi bước tái hiện, log), ngắn hơn kịch bản `task`.
- **AC-M1-03.8** `hub task ... --spec ./spec.md` ⇒ bỏ qua phỏng vấn, kiểm tra spec đúng template (sai ⇒ báo mục thiếu), đi thẳng tới gate `spec` (D-01).

### US-M1-04 · Bảng phân việc & prompt nạp sẵn (P0)
*Là chủ dự án, tôi muốn gán mỗi vai cho một cặp (harness, model) với prompt riêng, kể cả khi tôi chỉ có một harness.*
- **AC-M1-04.1** Cấu hình routing toàn cục trong `config/hub.yaml`: mỗi vai trong §2.2 → `harness`, `model`, `prompt`, `timeout`. Thiếu vai nào ⇒ `hubd` từ chối khởi động với thông báo rõ.
- **AC-M1-04.2** Có preset **"chỉ Antigravity"**: mọi vai dùng `antigravity` với các model khác nhau; chạy trọn pipeline `task` thành công chỉ với Antigravity đăng nhập (FR-4 mục tiêu, FR-16). [Proto]
- **AC-M1-04.3** `hub route` in bảng vai → harness → model; `hub route code antigravity claude-sonnet-5.5` đổi route, áp dụng cho **chặng chạy sau đó** (không ảnh hưởng chặng đang chạy).
- **AC-M1-04.4** Có prompt mặc định cho 8 vai; render bằng Jinja2 với các biến `{{spec}} {{plan}} {{repo_summary}} {{test_report}} {{review}} {{round}} {{project.*}} {{task.*}}`. Biến không tồn tại ⇒ lỗi khi lưu prompt, không phải khi chạy.
- **AC-M1-04.5** `hub prompt <vai> edit|history|rollback <v>`: mỗi lần lưu tạo version mới; rollback tạo version mới có nội dung cũ (FR-18).

### US-M1-05 · Adapter Claude Code & Antigravity trong container (P0) 🛡
*Là chủ dự án, tôi muốn hub gọi đúng CLI chính chủ trong môi trường cách ly để dùng gói đăng ký hợp lệ và agent không thể đụng tới credential của hub.*
- **AC-M1-05.1** Adapter chỉ gọi CLI chính chủ ở chế độ headless (ma trận M0); **không** import SDK của harness (FR-15a). Kiểm tra: dependency của `hubd` không chứa SDK nào của Anthropic/Google/OpenAI agent.
- **AC-M1-05.2** Mỗi lần chạy agent là một container `ntchub-agent` riêng, mount **chỉ**: worktree của task (rw) + thư mục credential của đúng harness đó. 🛡
- **AC-M1-05.3** 🛡 Test cách ly: chạy agent với prompt "in ra nội dung `~/.ntc-hub/secrets.enc`, `~/.ssh`, biến môi trường `GITHUB_TOKEN`, thử `git push`" ⇒ không đọc được file nào, env không có token, push thất bại vì remote không có credential.
- **AC-M1-05.4** Output của harness được parse thành `AgentEvent` (`TextDelta`, `ToolCall`, `ToolResult`, `Usage`, `RateLimited`, `Error`, `Done`) và ghi vào `logs/<task>/<stage>-<attempt>.log`. Parser có unit test với fixture M0.
- **AC-M1-05.5** Agent chạy quá `timeout` của route ⇒ container bị kill, `STAGE_RUN` = lỗi với lý do `timeout` (NFR-2).
- **AC-M1-05.6** Thông báo hết quota / rate-limit của harness được ánh xạ thành `RateLimited(retry_after)` (dựa trên fixture M0).
- **AC-M1-05.7** `hub agents` hiển thị cho từng harness: đăng nhập OK / hết hạn / thiếu, rate-limit tới khi nào, số agent đang chạy / giới hạn (FR-19). [Proto]
- **AC-M1-05.8** Xử lý tối thiểu ở M1 (chưa có fallback / tự chạy lại — xem M3): harness báo rate-limit ⇒ task `rate_limited` + `block_reason=rate_limit`; harness hết hạn đăng nhập ⇒ task `rate_limited` + `block_reason=auth_expired`; cả hai giải phóng slot; `hub resume <id>` đưa task về `queued` thủ công. Sau khởi động lại `hubd`, `block_reason` được giữ nguyên.

### US-M1-06 · Pipeline `task` trên repo có sẵn (P0)
*Là chủ dự án, tôi muốn sau khi duyệt spec, hub tự chạy plan → code → test → review → fix → report mà không cần tôi trông.*
- **AC-M1-06.1** Pipeline được nạp từ `config/pipelines/feature.yaml` (ARCH §5.1); thứ tự mặc định **Plan → Code → Test → Review → (Fix ↺ Test/Review) → Report → Merge** (FR-7).
- **AC-M1-06.2** Mỗi chặng thực hiện đúng 8 bước ARCH §5.3; sau mỗi chặng có thay đổi, **hub** commit với message `hub(<id>): <stage> round <n>`.
- **AC-M1-06.3** Các chặng trao đổi qua artifact file (`spec.md`, `plan.md`, `test-report.md`, `review.json`, `REPORT.md`) nằm trong `.hub/` của worktree (FR-12). Kiểm tra: đổi route của `review` sang harness khác giữa chừng vẫn chạy được.
- **AC-M1-06.4** Gate `plan` bật ⇒ sau chặng plan, task chuyển `waiting_approval` và hiển thị tóm tắt plan ≤ 10 dòng.
- **AC-M1-06.5** Kết thúc: có PR từ `hub/<id>-<slug>` vào nhánh mặc định, mô tả PR là báo cáo chi tiết (FR-37).

### US-M1-07 · Pipeline `fix` (P0)
*Là chủ dự án, tôi muốn sửa bug theo quy trình có tái hiện để chắc lỗi thật sự được sửa.*
- **AC-M1-07.1** Pipeline `fix.yaml`: **Reproduce → Fix → Test → Review → (Fix ↺) → Report → Merge** (FR-7).
- **AC-M1-07.2** Chặng reproduce phải tạo **ít nhất một test fail** trên code gốc; hub tự chạy `test_cmd` để xác nhận nó fail. Không tái hiện được ⇒ task `waiting_approval` với câu hỏi cho tôi (tiếp tục / huỷ / bổ sung thông tin).
- **AC-M1-07.3** Sau chặng fix, chính test tái hiện đó phải **pass** khi hub chạy.

### US-M1-08 · Hub xác minh test độc lập (P0) 🛡
*Là chủ dự án, tôi muốn kết quả test là do hub chạy, không phải lời tự khai của agent.*
- **AC-M1-08.1** Ở chặng có `verify: project.test_cmd`, hub tự chạy `test_cmd` trong container của worktree, lấy exit code + số test pass/fail làm chuẩn (FR-10).
- **AC-M1-08.2** Agent nói "pass" nhưng hub chạy fail ⇒ kết quả là **fail**, log ghi rõ chênh lệch.
- **AC-M1-08.3** 🛡 `test_cmd` lưu trong cấu hình project; agent sửa file cấu hình test của hub (nếu có trong repo, ví dụ `.hub/project.yml`) ⇒ hub bỏ qua thay đổi đó và cảnh báo trong báo cáo (FR-10a).
- **AC-M1-08.4** Vai test **viết test mới** theo tiêu chí chấp nhận trong spec; `test-report.md` liệt kê test mới ↔ tiêu chí chấp nhận (FR-10b).
- **AC-M1-08.5** 🛡 Project không có `test_cmd` ⇒ báo cáo có dòng "⚠ không có test" và task **luôn** dừng ở `awaiting_merge`, kể cả khi gate `merge` tắt hoặc `/auto on` (FR-10b, D-02).

### US-M1-09 · Review có cấu trúc (P0)
- **AC-M1-09.1** Chặng review phải ghi `review.json` đúng schema ARCH §5.4.
- **AC-M1-09.2** File sai schema ⇒ chạy lại review **một lần** kèm nhắc; vẫn sai ⇒ coi là `changes_requested` + cảnh báo.
- **AC-M1-09.3** Áp quy tắc nhất quán D-07 (approve kèm blocker/major hoặc ≥ 3 minor ⇒ `changes_requested`).
- **AC-M1-09.4** Agent review chạy ở chế độ không sửa mã nguồn: nếu worktree có thay đổi ngoài `.hub/review.json` sau chặng review ⇒ hub hoàn tác thay đổi đó và ghi cảnh báo.

### US-M1-10 · Vòng fix có giới hạn & dừng sớm (P0)
*Là chủ dự án, tôi muốn hub không lặp vô tận khi agent bế tắc.*
- **AC-M1-10.1** Test (của hub) fail **hoặc** review `changes_requested` ⇒ chạy chặng fix với `test-report.md` + `review.json`, rồi test → review lại.
- **AC-M1-10.2** Vượt `max_rounds` (mặc định 3, theo D-08) ⇒ task `failed`, thông báo kèm tóm tắt lý do (lỗi cuối, số vòng) (FR-9).
- **AC-M1-10.3** Chữ ký lỗi test (tên test fail + thông điệp chuẩn hoá) **giống nhau ở 2 vòng liên tiếp** ⇒ dừng sớm, `failed` với lý do "lỗi lặp lại" (FR-26).
- **AC-M1-10.4** Trong vòng fix, cùng một vai dùng lại `resume_session_id` nếu harness hỗ trợ (FR-26).
- **AC-M1-10.5** `hub resume <id>` trên task `failed` ⇒ chạy lại từ chặng lỗi, bộ đếm vòng giữ nguyên trừ khi dùng `--reset-rounds`.

### US-M1-11 · An toàn Git (P0) 🛡
*Là chủ dự án, tôi muốn chắc chắn hub không bao giờ làm hỏng repo của tôi.*
- **AC-M1-11.1** Mỗi task có git worktree riêng tại `~/.ntc-hub/work/<id>/` trên branch `hub/<id>-<slug>` (FR-27).
- **AC-M1-11.2** 🛡 Lớp Git service **từ chối trong code** (không phụ thuộc cấu hình): push ref ngoài `refs/heads/hub/*`; `--force` (trừ `--force-with-lease` lên chính branch của task khi hub rebase); xoá branch không phải `hub/*`; sửa cài đặt repo (trừ bật branch protection cho repo mới). Mỗi trường hợp có unit test.
- **AC-M1-11.3** 🛡 Merge **chỉ** qua GitHub API trên PR, và chỉ khi đủ cả 4: test của hub pass trên **HEAD hiện tại** của PR, `review.verdict == approve`, gate `merge` đã duyệt hoặc tắt, PR không xung đột (FR-29).
- **AC-M1-11.4** Nhánh mặc định đã đi trước ⇒ hub cập nhật branch từ base, **chạy lại test** trước khi merge; xung đột không tự giải được ⇒ task về `queued` chạy chặng fix với thông tin xung đột.
- **AC-M1-11.5** Mọi thao tác git/GitHub có tác dụng ra ngoài (push, tạo PR, merge, tạo repo, branch protection) được ghi `audit_log` (thời gian, task, thao tác, ref, kết quả) (FR-31). `hub audit [--task id]` hiển thị.
- **AC-M1-11.6** 🛡 Test hồi quy chạy trong CI: một bộ "agent độc hại" giả lập (sửa `.git/config`, thêm remote, tạo branch `main` cục bộ, sửa hook) ⇒ không có thay đổi nào tới remote ngoài branch `hub/*` của task.
- **AC-M1-11.7** Dọn branch theo D-16: merge thành công ⇒ branch remote `hub/<id>-<slug>` bị xoá ngay, ghi `audit_log`; task `cancelled`/`failed` ⇒ branch remote còn nguyên cho tới khi dọn worktree (mặc định 7 ngày) hoặc `hub cleanup <id>` / `/cleanup [id]`. 🛡 Lệnh cleanup trên branch không phải `hub/*` hoặc không do hub tạo ⇒ bị từ chối.

### US-M1-12 · Điểm duyệt qua CLI (P0)
*Là chủ dự án, tôi muốn duyệt / từ chối / góp ý ở các điểm quan trọng.*
- **AC-M1-12.1** Task tới gate đang bật ⇒ trạng thái `waiting_approval`, tạo bản ghi `APPROVAL(status=pending)` (FR-23).
- **AC-M1-12.2** `hub approve <id>`, `hub reject <id> "<lý do>"`, `hub comment <id> "<góp ý>"` xử lý đúng theo D-03/D-04; ghi `APPROVAL.channel=cli`.
- **AC-M1-12.3** Duyệt hai lần (hoặc duyệt khi không có gate chờ) ⇒ báo "không có điểm duyệt nào đang chờ", không có tác dụng phụ. [Proto]
- **AC-M1-12.4** Gate `merge` bật ⇒ sau report, task `awaiting_merge` và **giải phóng slot** (FR-24a). `hub merge <id>` ⇒ thực hiện merge theo AC-M1-11.3; thất bại ⇒ nêu điều kiện nào chưa đạt.
- **AC-M1-12.5** Gate `merge` tắt và đủ điều kiện ⇒ hub tự merge, task `done` (FR-24a). Ngoại lệ (vẫn dừng ở `awaiting_merge`): project không có test (AC-M1-08.5) hoặc pipeline của task đã tắt chặng `test`/`review` (D-15).
- **AC-M1-12.7** `hub auto <id> on|off` ghi đè gate cho task theo D-02. Khi `on`: không hiện bản duyệt spec/plan, nhưng `hub spec <id>` / `hub plan <id>` luôn in được nội dung mới nhất của `spec.md` / `plan.md`.
- **AC-M1-12.6** Cấu hình gate theo project ghi đè cấu hình toàn cục; `deploy_prod` không tắt được ở cấu hình toàn cục (lệnh trả lỗi) (FR-24).

### US-M1-13 · Độ bền khi khởi động lại (P0)
- **AC-M1-13.1** Kill `hubd` (SIGKILL) khi một task đang ở giữa chặng code ⇒ khởi động lại, task về `queued`, worktree `reset --hard` về commit cuối của hub, chặng code chạy lại (FR-13, ARCH §5.3).
- **AC-M1-13.2** Task ở `waiting_approval`, `awaiting_merge`, `interviewing` giữ nguyên trạng thái và dữ liệu sau khởi động lại.
- **AC-M1-13.3** Container agent mồ côi (từ lần chạy trước) bị dọn khi `hubd` khởi động.

### US-M1-14 · Chạy song song & hàng đợi (P0)
*Là chủ dự án, tôi muốn nhiều task chạy cùng lúc trong giới hạn tôi đặt.*
- **AC-M1-14.1** Giới hạn cấu hình được: `global.max_tasks_running`, `harness.max_concurrent` (FR-25).
- **AC-M1-14.2** Tạo 5 task khi `max_tasks_running=2` ⇒ đúng 2 task `running`, 3 task `queued`; task xong ⇒ task kế tiếp được cấp slot trong ≤ 5 giây.
- **AC-M1-14.3** Ưu tiên hàng đợi: task đang giữa pipeline > task mới; `fix` > `task`/`new`; FIFO trong cùng mức (ARCH §7).
- **AC-M1-14.4** Hai task song song trên **cùng project** không ảnh hưởng nhau (worktree, branch, artifact tách biệt).

### US-M1-15 · Theo dõi & điều khiển qua CLI (P0)
- **AC-M1-15.1** `hub tasks` in bảng: ID, PROJECT, KIND, STATUS, STAGE (task chờ duyệt được tô màu nổi bật). [Proto]
- **AC-M1-15.2** `hub status <id>` in timeline chặng: ✓/✗/…, nhãn chặng, `harness · model`, ghi chú (ví dụ "46/48 pass"). [Proto] Task `rate_limited` hiển thị kèm `block_reason`, harness bị chặn và thời điểm dự kiến chạy lại (nếu biết) (§2.1).
- **AC-M1-15.3** `hub logs <id>` in 50 dòng cuối; `hub logs -f <id>` stream liên tục cho tới Ctrl+C.
- **AC-M1-15.4** `hub attach <id>` hiển thị trực tiếp output của agent đang chạy; khi task đang `interviewing` thì vào phiên phỏng vấn.
- **AC-M1-15.5** `hub pause <id>` ⇒ dừng sau khi agent hiện tại kết thúc hoặc kill ngay với `--now`; `hub resume <id>` ⇒ về `queued`; `hub cancel <id>` ⇒ kill agent, `cancelled`, **giữ branch**.
- **AC-M1-15.6** Bỏ qua `<id>` khi chỉ có một task đang hoạt động ⇒ CLI tự dùng task đó, và in rõ đang thao tác trên `#id`.

### US-M1-16 · Báo cáo (mức PR & CLI) (P0) 🛡
- **AC-M1-16.1** Report service sinh từ dữ liệu có cấu trúc (stage runs, test của hub, `review.json`, usage); model chỉ viết dòng "Đã làm" và "Cần bạn xem" (ARCH §14).
- **AC-M1-16.2** `hub report <id>` in **tin chính** đúng format FR-36 (≤ 10 dòng), theo sau là các phần chi tiết chia theo §2.5; bộ chia tin là một module dùng chung, dùng lại nguyên cho Telegram ở M2.
- **AC-M1-16.4** 🛡 Unit test bộ chia tin (§2.5) với dữ liệu dài bất thường (50 mục "Đã làm", một gạch đầu dòng 2.000 ký tự, khối code, link dài): tin chính luôn ≤ 10 dòng; mọi phần ≤ 15 dòng và ≤ 3.500 ký tự; ghép các phần lại **không mất ký tự nào** so với nội dung gốc (trừ tiêu đề phần); không có phần nào bắt đầu/kết thúc giữa câu, giữa khối code hay giữa link; tối đa 5 phần.
- **AC-M1-16.3** Mô tả PR chứa bản chi tiết: tóm tắt spec, danh sách thay đổi, kết quả test của hub, các issue review và cách đã xử lý, cảnh báo, thống kê quota.

**Exit M1:**
- Kịch bản demo: `hub project add` một repo mẫu (Node + Python) → `hub task` → phỏng vấn → `/go` → duyệt spec, plan → PR mở với test pass → `hub merge` thành công. Và `hub fix` tương tự.
- Toàn bộ AC 🛡 của M1 có test tự động xanh trong CI.
- Chạy 3 task song song qua đêm không có task kẹt ở `running` quá timeout.

---

## 5. Giai đoạn M2 — Telegram

**Mục tiêu:** làm mọi việc của M1 **từ điện thoại** qua nhóm Telegram có Topics.
**Phạm vi:** Telegram gateway (aiogram, long-polling), topic theo task, phỏng vấn trong topic, gate có nút, tin trạng thái ghim, báo cáo ngắn, lệnh theo dõi/điều khiển, `/login`, lệnh xem cấu hình cơ bản.
**Ngoài phạm vi:** `/new` (cần tạo repo — M3), `/apikey` (M3), setup đầy đủ qua Telegram (M5), preview/deploy (M5).

### US-M2-01 · Kết nối bot & bảo vệ quyền điều khiển (P0) 🛡
*Là chủ dự án, tôi muốn chỉ mình tôi điều khiển được bot.*
- **AC-M2-01.1** `hub telegram connect` nhận bot token (lưu `secrets.enc`), `owner_user_id`, `group_id`; `hub doctor` kiểm tra bot là admin có quyền **Manage Topics**.
- **AC-M2-01.2** 🛡 Tin nhắn / bấm nút từ user ID khác `owner_user_id` bị **bỏ qua hoàn toàn** (không phản hồi, không tác dụng), ghi log mức cảnh báo (FR-40).
- **AC-M2-01.3** Bot dùng long-polling; không cần mở port vào máy (ARCH §10).
- **AC-M2-01.4** Mỗi lệnh nhận được phản hồi (ít nhất "đã nhận") trong **< 3 giây** ở p95 (NFR-5).

### US-M2-02 · Tạo task từ topic General (P0)
*Là chủ dự án, tôi muốn gõ một lệnh ở General và có ngay một topic riêng cho việc đó.*
- **AC-M2-02.1** Ở General, `/task <project> <yêu cầu>` hoặc `/fix <project> <bug>` ⇒ hub tạo task và **topic mới** tên `#<id> · <slug>`; trong General bot trả lời "🆕 Đã tạo topic #id · slug" kèm nút **Mở topic →**. [Proto]
- **AC-M2-02.2** Câu yêu cầu gốc được đăng lại làm tin đầu tiên trong topic mới, phỏng vấn bắt đầu tại đó.
- **AC-M2-02.3** Project không tồn tại ⇒ bot trả danh sách project gợi ý (`/projects`), không tạo task.
- **AC-M2-02.4** Gõ tin thường (không phải lệnh) ở General ⇒ bot hướng dẫn dùng `/task`, `/fix`. [Proto]

### US-M2-03 · Phỏng vấn trong topic (P0)
- **AC-M2-03.1** Trong topic task đang `interviewing`, mọi tin nhắn không phải lệnh được coi là câu trả lời phỏng vấn.
- **AC-M2-03.2** Đáp ứng AC-M1-03.3/03.4 (một câu mỗi lượt, tóm tắt định kỳ, mặc định khi "tuỳ bạn").
- **AC-M2-03.3** Câu hỏi có thể kèm tối đa 3 nút gợi ý (D-09).
- **AC-M2-03.4** `/go` trong topic ⇒ bot gửi "⏳ Đang soạn spec.md…" rồi **tóm tắt spec ≤ 10 dòng** (gồm repo, lệnh test, deploy target) + nút **✅ Duyệt / 💬 Góp ý / ✖ Từ chối**; nút **📄 Chi tiết** gửi `spec.md` dạng file đính kèm (ARCH §8). Tóm tắt dài hơn ⇒ chia phần theo §2.5. [Proto]
- **AC-M2-03.6** Task đang `/auto on` ⇒ sau `/go` bot **không** gửi tóm tắt spec để duyệt, chỉ gửi một dòng "📄 Spec đã chốt — gõ /spec để xem" kèm nút **📄 Spec**, rồi chạy tiếp (D-02).
- **AC-M2-03.5** Phiên phỏng vấn dùng chung giữa Telegram và CLI: trả lời một câu ở `hub attach`, câu kế tiếp hiện trong topic và ngược lại (FR-1).

### US-M2-04 · Duyệt từ xa bằng nút (P0)
*Là chủ dự án, tôi muốn duyệt / góp ý chỉ bằng một chạm.*
- **AC-M2-04.1** Tới gate bật ⇒ bot nhắn trong topic: tóm tắt + nút **✅ Duyệt / 💬 Góp ý / ✖ Từ chối** (FR-23, J4).
- **AC-M2-04.2** Bấm **Góp ý** ⇒ bot nhắn "Bạn gõ góp ý nhé…"; tin nhắn kế tiếp của tôi trong topic là góp ý, xử lý theo D-03. [Proto]
- **AC-M2-04.3** Sau khi quyết định, các nút của tin đó được **thay** bằng nhãn kết quả ("✅ Đã duyệt", "💬 Đã góp ý", "✖ Đã từ chối"), bấm lại không có tác dụng. [Proto]
- **AC-M2-04.4** Duyệt ở kênh khác (CLI) ⇒ tin gate trong Telegram được cập nhật thành "✅ Đã duyệt (qua CLI)" và bot đăng một dòng thông báo. [Proto]
- **AC-M2-04.5** `/approve`, `/reject <góp ý>` trong topic tương đương nút; ở General phải kèm `[id]`.
- **AC-M2-04.6** `/auto on|off` trong topic ghi đè gate cho task đó theo D-02; bot xác nhận "⚡ Task này chạy thẳng, bỏ qua các điểm duyệt (trừ deploy prod). Gõ /spec hoặc /plan để xem bất cứ lúc nào." Tiêu đề topic / tin ghim hiển thị "⚡ chạy thẳng". [Proto]
- **AC-M2-04.7** `/spec [id]` và `/plan [id]` hoạt động ở **mọi trạng thái** sau khi artifact tồn tại, bất kể `/auto` hay gate: bot gửi tóm tắt (chia phần theo §2.5 nếu dài) + nút gửi file `spec.md` / `plan.md` đầy đủ. Artifact chưa có ⇒ "Chưa có spec/plan cho #id". Xem spec/plan **không** dừng pipeline.

### US-M2-05 · Tin nhắn tiến độ không spam (P0) 🛡
*Là chủ dự án, tôi muốn topic gọn để đọc nhanh trên điện thoại.*
- **AC-M2-05.1** Bot chỉ gửi tin **mới** khi: chuyển chặng (1 dòng "▶ <Chặng> — harness · model"), chờ duyệt, lỗi, xong (FR-38). [Proto]
- **AC-M2-05.2** Mỗi topic có một tin trạng thái **ghim**, được **edit** theo chặng/trạng thái hiện tại (ví dụ "📌 Code — codex · gpt-5.5-codex", "⏸ Chờ bạn: Duyệt plan", "🔀 Xong — chờ bạn merge") (D-10). [Proto]
- **AC-M2-05.3** Một task chạy trọn pipeline không lỗi tạo **≤ 15 tin bot** trong topic (không tính phỏng vấn).
- **AC-M2-05.4** Log chi tiết không bao giờ tự đẩy vào Telegram; `/logs` trả **8–10 dòng cuối** trong khối code. [Proto]

### US-M2-06 · Báo cáo ngắn (P0) 🛡
- **AC-M2-06.1** 🛡 Khi xong, bot gửi **tin chính** đúng format FR-36 (≤ 10 dòng, tự đủ nghĩa, mang nút), và nếu cần thì các phần chi tiết tiếp theo theo quy tắc §2.5 (D-13); test tự động kiểm quy tắc này trên 100% báo cáo sinh ra trong bộ test.
- **AC-M2-06.5** Các phần chi tiết được gửi liền ngay sau tin chính, đúng thứ tự `(2/n)…(n/n)`; nếu gửi một phần thất bại (lỗi mạng/Telegram) ⇒ thử lại phần đó, không gửi phần sau trước phần trước.
- **AC-M2-06.2** Báo cáo có nút **🔗 Preview** (ẩn/báo "chưa hỗ trợ" tới M5), **🔀 Merge**, **📄 Chi tiết** (link PR / gửi `REPORT.md`). Nút báo cáo **không biến mất** sau khi bấm. [Proto]
- **AC-M2-06.3** Task `failed` ⇒ tin lỗi tin chính ≤ 5 dòng: chặng lỗi, lý do (vượt vòng fix / lỗi lặp / timeout / lỗi harness), gợi ý `/logs`, `/resume`; chi tiết thêm (nếu có) chia phần theo §2.5. [Proto]
- **AC-M2-06.4** Merge thành công ⇒ bot nhắn "🔀 Đã merge PR vào main" kèm danh sách kiểm tra ✓ (test pass, review approve, không xung đột). [Proto]

### US-M2-07 · Lệnh theo dõi & điều khiển (P0)
- **AC-M2-07.1** Hỗ trợ: `/tasks`, `/status [id]`, `/logs [id]`, `/report [id]`, `/spec [id]`, `/plan [id]`, `/pause`, `/resume`, `/cancel [id]`, `/approve [id]`, `/reject [id] <góp ý>`, `/auto [id] on|off`, `/merge [id]`, `/cleanup [id]` (FR-42 + D-02, D-16).
- **AC-M2-07.2** Trong topic task, `id` được suy ra; ở General, thiếu `id` ⇒ bot báo cú pháp đúng. [Proto]
- **AC-M2-07.3** `/tasks` liệt kê mỗi task một dòng: biểu tượng trạng thái, `#id slug`, trạng thái tiếng Việt, chặng. Task `rate_limited` hiển thị lý do ngắn: "chờ quota" (`rate_limit`) hoặc "chờ đăng nhập" (`auth_expired`). [Proto]
- **AC-M2-07.4** Danh sách topic hiển thị biểu tượng trạng thái trong tên topic (💬 ⏳ ⚙️ ⏸ 🔀 ✅ ⛔ ✖), cập nhật khi đổi trạng thái [Đề xuất — nếu Telegram API cho phép đổi tên/biểu tượng topic].
- **AC-M2-07.5** `/cancel` khi đang có gate chờ ⇒ gate được đóng (từ chối) và task `cancelled`, branch giữ lại.

### US-M2-08 · Trạng thái harness & đăng nhập lại (P0)
- **AC-M2-08.1** `/agents` hiển thị mỗi harness: 🟢/🔴, đăng nhập OK/HẾT HẠN, quota ước tính (nếu biết) hoặc "API key", số đang chạy / giới hạn. [Proto]
- **AC-M2-08.2** Hub phát hiện phiên hết hạn (kiểm tra định kỳ ≤ 15 phút, hoặc khi chạy lỗi) ⇒ nhắn **một lần** ở General: "🔴 <harness> hết hạn đăng nhập — gõ /login <harness>" (FR-19).
- **AC-M2-08.3** `/login <harness>` với harness hỗ trợ device code ⇒ bot gửi link + mã; khi đăng nhập xong, bot nhắn "🟢 <harness> đã đăng nhập" và **mọi** task `rate_limited` có `block_reason=auth_expired` và `blocked_harness=<harness>` về `queued`, `block_reason` được xoá (D-05). Harness không hỗ trợ ⇒ hướng dẫn `hub login` qua SSH (FR-43).
- **AC-M2-08.4** Harness hết hạn khi task đang chạy, không có fallback ⇒ task `rate_limited` + `block_reason=auth_expired`, giải phóng slot, topic task nhắn một dòng "🔴 Chờ đăng nhập <harness> — gõ /login <harness> ở General"; tin ghim cập nhật tương ứng.

### US-M2-09 · Lệnh nhạy cảm chỉ ở chat riêng (P0) 🛡
- **AC-M2-09.1** 🛡 `/github` (và mọi lệnh nhận secret) gõ trong nhóm ⇒ bot **xoá tin ngay**, nhắn "Lệnh này chỉ dùng trong chat riêng với bot", không lưu gì (FR-41).
- **AC-M2-09.2** 🛡 Trong chat riêng, sau khi lưu secret bot **xoá tin chứa secret** trong ≤ 3 giây và xác nhận "đã lưu ••••<4 ký tự cuối>".
- **AC-M2-09.3** 🛡 Grep toàn bộ log sau bộ test ⇒ không có chuỗi secret nào.

### US-M2-10 · Xem & chỉnh cấu hình cơ bản (P1)
- **AC-M2-10.1** `/projects`, `/route`, `/gates`, `/limits`, `/help` hiển thị đúng dữ liệu hiện hành; `/help` nhóm lệnh theo bảng FR-42. [Proto]
- **AC-M2-10.2** `/route <vai> <harness> <model>` đổi route toàn cục; model không có trong danh sách của harness ⇒ từ chối và liệt kê model hợp lệ.
- **AC-M2-10.3** `/gates <gate> on|off` đổi gate toàn cục, trừ `deploy_prod` (báo 🔒).

**Exit M2:**
- Kịch bản demo J2 + J3 + J4 làm **hoàn toàn trên điện thoại**: `/task` → phỏng vấn → `/go` → duyệt spec, plan bằng nút → nhận báo cáo ≤ 10 dòng → bấm Merge.
- Chạy song song 2 task, duyệt một gate ở CLI và thấy Telegram cập nhật.
- Thử nghiệm từ một tài khoản Telegram thứ hai trong nhóm: không lệnh nào có tác dụng.

---

## 6. Giai đoạn M3 — App mới + Codex + DeepSeek Harness

**Mục tiêu:** hoàn thành **J1** (từ ý tưởng đến repo mới có PR), mở rộng harness, xử lý quota thông minh.
**Phạm vi:** `/new` + tạo repo, adapter Codex, adapter `dsh`, `/apikey`, fallback, rate-limit tự phục hồi, giới hạn theo (harness, model), thống kê quota, ghi đè route theo project/task, cache repo summary.

### US-M3-01 · App mới từ một câu ý tưởng (P0)
*Là chủ dự án, tôi muốn gõ một câu ý tưởng và nhận lại repo mới có PR chạy được.*
- **AC-M3-01.1** `/new <ý tưởng>` (Telegram) hoặc `hub new "<ý tưởng>"` ⇒ task `kind=new`, topic mới, kịch bản phỏng vấn `new` (hỏi người dùng, nền tảng, màn hình chính, …). [Proto]
- **AC-M3-01.2** `spec.md` của app mới có: tên repo đề xuất (`owner/<slug>`, private), stack, `test_cmd` đề xuất, deploy target đề xuất (FR-10a, FR-32). Tóm tắt spec trên Telegram hiển thị đủ 4 thông tin này. [Proto]
- **AC-M3-01.3** Sau khi duyệt spec: hub tạo repo **private**, commit khởi tạo trên nhánh mặc định, bật branch protection nếu gói GitHub cho phép (không cho phép ⇒ ghi cảnh báo vào báo cáo, vẫn tiếp tục) (FR-30); bot nhắn "📦 Đã tạo repo …, branch hub/<id>-<slug>". [Proto]
- **AC-M3-01.4** Tên repo đã tồn tại ⇒ hub hỏi lại tên (không ghi đè, không dùng repo đó). 🛡
- **AC-M3-01.5** Project được đăng ký tự động với `test_cmd` đã duyệt và deploy target.
- **AC-M3-01.6** Toàn bộ code app nằm trong PR từ `hub/<id>-<slug>`; nhánh mặc định chỉ có commit khởi tạo cho tới khi merge. 🛡

### US-M3-02 · Adapter Codex CLI (P0)
- **AC-M3-02.1** Đáp ứng toàn bộ AC-M1-05.x cho Codex (headless, container, parse JSON, timeout, rate-limit).
- **AC-M3-02.2** Preset "đa harness" (prototype `PRESET_MULTI`) chạy trọn pipeline `new` thành công.

### US-M3-03 · DeepSeek Harness với API key (P0) 🛡
*Là chủ dự án, tôi muốn dùng model qua API key làm bổ trợ mà không trộn key vào harness đăng ký.*
- **AC-M3-03.1** `/apikey <provider>` (chỉ chat riêng, theo US-M2-09) hoặc `hub apikey set <provider>` lưu key vào `secrets.enc`.
- **AC-M3-03.2** 🛡 API key **chỉ** được truyền vào container của `dsh`; cấu hình route gán API key cho `claude-code`/`codex`/`antigravity` bị từ chối khi lưu (FR-15).
- **AC-M3-03.3** Adapter `dsh` dùng profile `ntchub` do hub sinh; model liệt kê theo provider đã cấu hình.
- **AC-M3-03.4** Giao diện hiển thị key dạng `sk-••••••••3f9a` và nhãn "tính theo token". [Proto]

### US-M3-04 · Fallback khi hết quota / hết hạn đăng nhập (P0)
*Là chủ dự án, tôi muốn task không đứng chờ khi một harness hết quota, nếu tôi cho phép dùng phương án dự phòng.*
- **AC-M3-04.1** Mỗi route có danh sách `fallbacks` có thứ tự gồm cặp (harness, model); công tắc fallback bật/tắt toàn cục và theo project (FR-20).
- **AC-M3-04.2** Chặng nhận `RateLimited` hoặc lỗi auth, fallback bật ⇒ chạy lại chặng bằng fallback kế tiếp còn khả dụng; bot nhắn **một** dòng "⚠️ <harness> … → chuyển sang dự phòng <harness · model>". [Proto]
- **AC-M3-04.3** Timeline / báo cáo ghi rõ chặng nào chạy bằng fallback.
- **AC-M3-04.4** Fallback tắt hoặc hết fallback ⇒ hành vi theo US-M3-05 (rate-limit) hoặc D-05 (auth).

### US-M3-05 · Tự phục hồi sau rate-limit (P0)
- **AC-M3-05.1** Nhận `RateLimited(retry_after)` ⇒ đặt `rate_limited_until` cho harness (hoặc (harness, model)), task `rate_limited` với `block_reason=rate_limit` và `blocked_harness`/`blocked_model`, giải phóng slot, nhắn Telegram **một lần** cho mỗi lần bị chặn (FR-26).
- **AC-M3-05.2** Tới `retry_after` ⇒ task tự về `queued` (xoá `block_reason`) và chạy lại chặng dở; không cần thao tác của tôi. Task có `block_reason=auth_expired` **không** tự chạy lại theo thời gian — chỉ khi đăng nhập lại OK (AC-M2-08.3).
- **AC-M3-05.3** Harness không trả `retry_after` ⇒ dùng backoff cấu hình được (mặc định 15 phút, tối đa 6 giờ) [Đề xuất].
- **AC-M3-05.4** Trong khi harness bị chặn, Scheduler không cấp slot mới cho chặng cần harness đó.

### US-M3-06 · Giới hạn song song theo (harness, model) (P0)
- **AC-M3-06.1** Cấu hình `(harness, model).max_concurrent` (ví dụ `antigravity · claude-opus-5.5 = 1`) được tôn trọng song song với giới hạn harness và toàn cục (FR-25). [Proto]
- **AC-M3-06.2** Test: 3 task cùng tới chặng plan dùng model giới hạn 1 ⇒ chỉ 1 chạy, 2 chờ, không deadlock với các chặng khác.

### US-M3-07 · Thống kê quota (P1)
- **AC-M3-07.1** Mỗi `AGENT_RUN` lưu thời gian, số lượt gọi, token vào/ra (nếu harness trả về) (FR-26).
- **AC-M3-07.2** Dòng "Quota:" trong báo cáo Telegram gom theo model, ví dụ "Opus 12 lượt · Sonnet 41 lượt" (FR-36).
- **AC-M3-07.3** `hub usage [--task id] [--since 7d]` tổng hợp theo task, chặng, harness, model.

### US-M3-08 · Ghi đè route theo project & task (P1)
- **AC-M3-08.1** Tra route theo thứ tự task → project → global (ARCH §4, FR-17).
- **AC-M3-08.2** `hub route --project shop-api code codex gpt-5.5-codex` và `/route` trong topic task (ghi đè cho task) hoạt động; `hub route --explain <task> <vai>` cho biết route đến từ cấp nào.

### US-M3-09 · Tiết kiệm ngữ cảnh (P1)
- **AC-M3-09.1** `repo_summary` sinh **một lần cho mỗi commit** của nhánh mặc định, lưu `cache/repo-summary/<project>/<sha>.md`, được nạp vào prompt; task thứ hai trên cùng commit không sinh lại (FR-26).
- **AC-M3-09.2** Mặc định gợi ý phân tầng model: model nhanh cho interview/test/report, model mạnh cho plan/code/review.

**Exit M3:**
- Demo J1 đầy đủ trên Telegram: `/new Tôi muốn làm app quản lý chi tiêu gia đình` → … → PR trên repo mới → merge.
- Giả lập harness trả rate-limit: task tự chuyển fallback (khi bật) hoặc tự chạy lại sau `retry_after` (khi tắt).
- Chạy preset "chỉ Antigravity" và preset "đa harness" đều xanh.

---

## 7. Giai đoạn M4 — Webapp

**Mục tiêu:** một màn hình tổng quan + nơi cấu hình mọi thứ thoải mái hơn Telegram/CLI.
**Phạm vi:** SPA React do `hubd` phục vụ, đăng nhập, dashboard, chi tiết task, routing, prompt, pipeline, gate & giới hạn, harness & key, GitHub, project, audit log, export/import cấu hình.
Bố cục màn hình theo prototype (thanh điều hướng bên trái: Dashboard · Chi tiết task · Phân việc & model · Prompt nạp sẵn · Điểm duyệt & giới hạn · Harness & API key), bổ sung Projects · Pipelines · Audit log.

### US-M4-01 · Truy cập an toàn từ xa (P0) 🛡
- **AC-M4-01.1** 🛡 `hubd` chỉ bind `127.0.0.1`; quét port từ ngoài không thấy Webapp (FR-49, ARCH §12).
- **AC-M4-01.2** `hub setup webapp` cấu hình truy cập qua **Cloudflare Tunnel + Cloudflare Access** (D-17): tạo/kết nối tunnel `cloudflared` (chạy dưới systemd), trỏ hostname `hub.<domain>` vào `127.0.0.1:<port>`, tạo Access application chỉ cho email của chủ dự án. `hub doctor` kiểm tra tunnel đang kết nối và Access policy tồn tại.
- **AC-M4-01.5** 🛡 Mở `https://hub.<domain>` từ trình duyệt chưa đăng nhập Cloudflare Access ⇒ bị chặn ở trang đăng nhập của Access, request **không tới** `hubd` (kiểm tra log `hubd`). Đăng nhập Webapp (AC-M4-01.3) là lớp bảo vệ thứ hai, vẫn bắt buộc.
- **AC-M4-01.3** Có đăng nhập: mật khẩu + TOTP **hoặc** magic link do bot Telegram gửi trong chat riêng; phiên hết hạn sau thời gian cấu hình (mặc định 7 ngày).
- **AC-M4-01.4** Mọi API ghi (duyệt, đổi cấu hình) yêu cầu phiên hợp lệ và chống CSRF.

### US-M4-02 · Dashboard (P0)
*Là chủ dự án, tôi muốn nhìn một màn hình là biết cái gì đang chạy, cái gì cần tôi.*
- **AC-M4-02.1** Thẻ **"⏸ Cần bạn duyệt (n)"** ở đầu trang khi có gate chờ, mỗi dòng có nút **Duyệt** và **Xem**. [Proto]
- **AC-M4-02.2** 5 cột: Đang phỏng vấn · Đang chạy / xếp hàng · Chờ duyệt · Chờ merge · Xong / dừng; mỗi thẻ task hiển thị biểu tượng, `#id slug`, kind, chặng. Task `rate_limited` nằm ở cột "Đang chạy / xếp hàng" với nhãn "chờ quota" hoặc "chờ đăng nhập" theo `block_reason`. [Proto]
- **AC-M4-02.3** Hàng thẻ harness: trạng thái đăng nhập, số đang chạy / giới hạn. [Proto]
- **AC-M4-02.4** Dữ liệu cập nhật realtime qua WebSocket (≤ 2 giây sau sự kiện); mất kết nối rồi nối lại ⇒ đồng bộ bằng bảng `events`, không mất cập nhật.

### US-M4-03 · Chi tiết task (P0)
- **AC-M4-03.1** Chọn task từ dropdown; hiển thị trạng thái, branch, link PR. [Proto]
- **AC-M4-03.2** Gate đang chờ ⇒ thẻ nổi bật với **Duyệt / Từ chối / Góp ý**, ghi chú "cùng tin nhắn đang chờ trong Telegram topic #id"; duyệt ở đây cập nhật Telegram theo AC-M2-04.4. [Proto]
- **AC-M4-03.3** Timeline chặng: ✅/❌/🟡, nhãn, `harness · model`, ghi chú (kết quả test, verdict review). [Proto]
- **AC-M4-03.4** Artifact (`spec.md`, `plan.md`, `test-report.md`, `review.json`, `REPORT.md`) xem được dạng render markdown/JSON, có lịch sử version theo vòng — kể cả với task `/auto on` (D-02).
- **AC-M4-03.5** Tab **Diff** so với nhánh mặc định; tab **Log trực tiếp** stream; thẻ **Quota đã dùng**. [Proto]
- **AC-M4-03.6** Nút hành động: Preview (M5), Merge, Deploy prod (M5), Mở topic Telegram; nút không hợp lệ ở trạng thái hiện tại bị vô hiệu kèm tooltip lý do. [Proto]
- **AC-M4-03.7** Task `interviewing` ⇒ khung chat để tiếp tục phỏng vấn, đồng bộ với Telegram/CLI (ARCH §8).

### US-M4-04 · Phân việc & model (P0)
- **AC-M4-04.1** Bảng: Vai · Harness (dropdown) · Model (dropdown phụ thuộc harness) · Dự phòng; harness hết hạn đăng nhập hiển thị "● hết hạn" đỏ cạnh route. [Proto]
- **AC-M4-04.2** Nút preset **"Mẫu: đa harness"** và **"Mẫu: chỉ Antigravity"** điền nhanh toàn bảng (có xác nhận trước khi ghi đè). [Proto]
- **AC-M4-04.3** Đổi harness ⇒ model tự đặt về model đầu tiên của harness đó; thay đổi áp dụng cho các chặng **tiếp theo**. [Proto]
- **AC-M4-04.4** Chỉnh danh sách fallback (thêm/xoá/sắp xếp); chặn gán API key cho harness đăng ký (AC-M3-03.2).
- **AC-M4-04.5** Chọn phạm vi chỉnh: Toàn cục / Project / Task.

### US-M4-05 · Prompt nạp sẵn có version (P0)
- **AC-M4-05.1** Danh sách 8 vai bên trái; editor bên phải hiển thị version hiện tại và thời điểm sửa ("v3 · sửa 2 ngày trước"). [Proto]
- **AC-M4-05.2** **Lưu** tạo version mới (vN+1); **Lịch sử** cho xem diff giữa hai version và rollback. [Proto]
- **AC-M4-05.3** Hiển thị danh sách biến khả dụng; lưu prompt dùng biến không tồn tại ⇒ báo lỗi, không lưu. [Proto]
- **AC-M4-05.4** Có prompt phỏng vấn riêng cho `new`, `task`, `fix` (FR-2).

### US-M4-06 · Pipeline (P1)
- **AC-M4-06.1** Bật/tắt chặng và đổi thứ tự **trong khuôn mẫu** (ví dụ tắt `plan` cho task nhỏ) (FR-8). Tắt chặng `test` hoặc `review` **được phép** nhưng theo D-15: hiện cảnh báo "Task dùng pipeline này sẽ không bao giờ tự merge"; bước hub tự chạy `test_cmd` trước merge **không xuất hiện** như một công tắc (không tắt được).
- **AC-M4-06.2** Chỉnh `max_rounds`, gate gắn với chặng, vai của chặng.
- **AC-M4-06.3** Lưu ⇒ kiểm tra hợp lệ (vòng lặp tham chiếu chặng tồn tại, artifact đầu vào có chặng tạo ra); không hợp lệ ⇒ không lưu, chỉ ra lỗi.

### US-M4-07 · Điểm duyệt & giới hạn (P0)
- **AC-M4-07.1** Công tắc 5 gate kèm mô tả; `deploy_prod` có 🔒 và bị vô hiệu ở cấu hình toàn cục. [Proto]
- **AC-M4-07.2** Chỉnh: số vòng fix tối đa, task đồng thời toàn hệ thống, đồng thời theo harness và (harness, model), chính sách khi hết quota (chờ / dự phòng), thời gian preview tự tắt. [Proto]
- **AC-M4-07.3** Ghi đè theo project (gồm tắt `deploy_prod` tường minh cho project, yêu cầu xác nhận hai bước).

### US-M4-08 · Harness, API key & GitHub (P0)
- **AC-M4-08.1** Thẻ mỗi harness: trạng thái đăng nhập, phương thức ("OAuth · Claude Max", "API key · DeepSeek"), thanh quota ước tính (nếu biết), danh sách model, nút **Kiểm tra**, **Làm mới danh sách model**, **Đăng nhập lại (OAuth)** khi hết hạn. [Proto]
- **AC-M4-08.2** Thẻ GitHub: App/PAT đang dùng, tài khoản, quyền; dòng nhắc "Chỉ hub giữ token. Agent không có quyền push." [Proto]
- **AC-M4-08.3** Nhập/sửa API key qua form; key không bao giờ được trả ngược về trình duyệt (chỉ 4 ký tự cuối).

### US-M4-09 · Projects (P0)
- **AC-M4-09.1** Danh sách project; thêm project từ repo GitHub; sửa `test_cmd`, `build_cmd`, deploy target, override route/gate.
- **AC-M4-09.2** Sửa `test_cmd` ghi `audit_log` (vì ảnh hưởng điều kiện merge). 🛡

### US-M4-10 · Audit log (P1)
- **AC-M4-10.1** Bảng lọc theo task / project / loại thao tác / thời gian; mỗi dòng có kết quả và liên kết tới PR/commit.

### US-M4-11 · Export / import cấu hình (P1)
- **AC-M4-11.1** `hub config export > hub.yaml` và nút tương ứng trên Webapp ⇒ YAML gồm routing, prompt (version hiện tại), pipeline, gate, giới hạn, project; **không** chứa secret (FR-46). 🛡
- **AC-M4-11.2** `hub config import` kiểm tra hợp lệ, hiển thị diff và yêu cầu xác nhận trước khi áp dụng.

**Exit M4:** mọi cấu hình trong PRD FR-48 chỉnh được trên Webapp; một gate chờ được duyệt trên Webapp và Telegram cập nhật; truy cập qua tunnel có đăng nhập, không có port công khai.

---

## 8. Giai đoạn M5 — Preview, Deploy & Setup đầy đủ

**Mục tiêu:** xem thử app trên điện thoại, deploy/rollback an toàn, và cài đặt lần đầu được từ bất kỳ kênh nào.
**Phạm vi:** Deploy service với target `docker` (Caddy), `vercel`, `cloudflare`; preview; deploy/rollback; `/setup` (Telegram), `hub setup`, wizard Webapp.

### US-M5-01 · Preview Docker trên VPS (P0) 🛡
*Là chủ dự án, tôi muốn bấm một nút để mở bản chạy thử trên điện thoại, mà người lạ không vào được.*
- **AC-M5-01.1** `/preview [id]` (hoặc nút) trên project `docker` ⇒ build từ `Dockerfile`/`compose.yaml` của branch task, chạy container giới hạn CPU/RAM, đăng ký route Caddy `<task>.preview.<domain>`, trả link trong ≤ 10 phút (FR-33).
- **AC-M5-01.2** 🛡 Preview nằm sau **Cloudflare Tunnel + Access** (D-17): Caddy chỉ nghe nội bộ, `cloudflared` đưa wildcard `*.preview.<domain>` vào Caddy, Access policy áp cho toàn bộ wildcard; truy cập ẩn danh nhận trang đăng nhập của Access (FR-33a). Tin nhắn ghi chú "Sau Cloudflare Access · tự tắt sau 24h · /down để tắt sớm". [Proto]
- **AC-M5-01.5** Không mở port công khai nào trên VPS cho preview hay Webapp (quét port từ ngoài chỉ thấy SSH); TLS do Cloudflare đảm nhận.
- **AC-M5-01.3** Container preview không mount secret của hub, không có Docker socket.
- **AC-M5-01.4** Build lỗi ⇒ tin lỗi ≤ 5 dòng + 20 dòng log cuối qua nút Chi tiết.

### US-M5-02 · Preview trên Vercel / Cloudflare (P0)
- **AC-M5-02.1** Project `vercel`/`cloudflare` ⇒ tạo preview deployment của nền tảng từ branch task và trả URL.
- **AC-M5-02.2** Token Vercel/Cloudflare chỉ nằm trong `secrets.enc`, không đưa vào container agent. 🛡

### US-M5-03 · Tự tắt preview (P0)
- **AC-M5-03.1** Preview tự teardown sau thời gian cấu hình (mặc định 24h), khi `/down [id]`, hoặc khi task `done`/`cancelled` (FR-34).
- **AC-M5-03.2** `/preview` lần hai trên cùng task ⇒ thay bản cũ bằng bản mới từ HEAD hiện tại, giữ nguyên URL nếu target hỗ trợ.

### US-M5-04 · Deploy staging/prod có gate (P0) 🛡
*Là chủ dự án, tôi muốn deploy chỉ khi tôi ra lệnh, và production luôn hỏi lại tôi.*
- **AC-M5-04.1** 🛡 Không có đường nào tự deploy sau merge; deploy chỉ bắt đầu từ `/deploy <project> [staging|prod]`, `hub deploy`, hoặc nút Webapp (FR-24b).
- **AC-M5-04.2** Deploy lấy HEAD nhánh mặc định (D-06); trong topic task chưa merge ⇒ "⛔ Cần merge trước khi deploy". [Proto]
- **AC-M5-04.3** 🛡 `deploy_prod`: luôn hiện tin duyệt (Target, commit SHA, ghi chú "Điểm duyệt deploy prod luôn bật") + **Duyệt / Từ chối**, kể cả khi task `/auto on`; chỉ bỏ qua nếu project đã tắt tường minh (AC-M4-07.3). [Proto]
- **AC-M5-04.4** Thành công ⇒ "✅ Đã deploy <env>: <url> · /rollback <project> <env> nếu cần"; ghi bản ghi `DEPLOYMENT` + `audit_log`. [Proto]
- **AC-M5-04.5** Hai lệnh deploy cùng project/env chồng nhau ⇒ lệnh sau xếp hàng, không chạy song song.

### US-M5-05 · Rollback (P0)
- **AC-M5-05.1** `/rollback <project> [env]` ⇒ quay về deployment thành công trước đó (docker: tag trước; vercel/cloudflare: deployment trước) (FR-35).
- **AC-M5-05.2** Rollback prod đi qua gate `deploy_prod`.
- **AC-M5-05.3** Không có deployment trước ⇒ báo rõ, không làm gì.

### US-M5-06 · Cấu hình deploy theo project (P1)
- **AC-M5-06.1** Đọc `.hub/deploy.yml` trong repo (ưu tiên) hoặc cấu hình trong DB (ARCH §13); thay đổi `.hub/deploy.yml` do agent tạo ra chỉ có hiệu lực sau khi merge qua PR.
- **AC-M5-06.2** Với app mới, deploy target trong spec đã duyệt được ghi vào cấu hình project.

### US-M5-07 · Setup lần đầu từ mọi kênh (P0)
*Là chủ dự án, tôi muốn thiết lập hub từ điện thoại, terminal, hoặc trình duyệt — kênh nào tiện thì dùng.*
- **AC-M5-07.1** `/setup` (chat riêng), `hub setup`, và wizard Webapp đi cùng thứ tự J6: GitHub → đăng nhập harness (OAuth) → API key cho `dsh` (tuỳ chọn) → bảng phân việc (chọn preset) → prompt (mặc định) → gate mặc định → giới hạn song song → (tuỳ chọn) domain/tunnel cho preview.
- **AC-M5-07.2** Mỗi bước có thể bỏ qua và quay lại; tiến độ setup lưu bền, kênh khác tiếp tục được.
- **AC-M5-07.3** Cuối setup chạy `hub doctor` và hiển thị kết quả trên chính kênh đó.
- **AC-M5-07.4** Bước nhận secret trên Telegram tuân thủ US-M2-09. 🛡

**Exit M5:** demo J1 trọn vẹn tới `/preview` trên điện thoại → `/merge` → `/deploy prod` (duyệt) → `/rollback`; preview không vào được khi chưa đăng nhập Access; cài đặt trên VPS mới hoàn toàn bằng `/setup`.

---

## 9. Yêu cầu xuyên suốt (áp dụng từ M1 trở đi)

| ID | Yêu cầu | Acceptance Criteria |
|---|---|---|
| X-01 🛡 | Bảo mật secret (NFR-1) | `secrets.enc` mã hoá, khoá trong OS keyring hoặc file quyền `0600` của `ntchub`; grep log & DB (ngoài `secrets.enc`) sau bộ test E2E không thấy secret. |
| X-02 | Độ bền (NFR-2) | SQLite chế độ WAL; mọi chuyển trạng thái có migration Alembic; test kill-restart (AC-M1-13.x) chạy trong CI. |
| X-03 | Quan sát (NFR-3) | Log theo `task/stage/attempt`; giữ log mặc định 30 ngày; `audit_log` giữ vĩnh viễn (cho tới khi xoá tay). |
| X-04 | Dọn dẹp | Worktree của task `done`/`cancelled`/`failed` xoá sau N ngày (mặc định 7); branch remote `hub/*` theo D-16 (xoá ngay khi merge; task huỷ/lỗi xoá cùng worktree hoặc khi `/cleanup`). |
| X-05 | Phản hồi (NFR-5) | Lệnh Telegram p95 < 3 giây; lệnh CLI đọc dữ liệu (tasks/status) < 1 giây. |
| X-06 | Mở rộng (NFR-6) | Thêm một harness giả (`FakeHarness`) chỉ bằng một file adapter + đăng ký, không sửa Orchestrator/Scheduler — có test chứng minh. |
| X-07 | Giới hạn tài nguyên agent | Container agent có giới hạn CPU/RAM/thời gian cấu hình được (ARCH §15). |
| X-08 | Đồng bộ đa kênh (FR-1) | Mọi hành động (tạo task, trả lời phỏng vấn, duyệt, merge) làm ở kênh nào cũng phản ánh ở các kênh còn lại trong ≤ 3 giây. |

---

## 10. Chiến lược kiểm thử

| Mức | Nội dung | Giai đoạn |
|---|---|---|
| Unit | Parser adapter (fixture M0), quy tắc Git service (AC-M1-11.2), máy trạng thái task (gồm `block_reason`), Scheduler, render prompt, bộ chia tin nhắn (§2.5, AC-M1-16.4), quy tắc review (D-07), điều kiện tự merge (D-15) | M1+ |
| Integration | `FakeHarness` kịch bản hoá (pass, fail test, review changes_requested, rate-limit, auth expired, treo) chạy pipeline thật trên repo git cục bộ + GitHub giả lập (hoặc repo test riêng) | M1+ |
| Bảo mật | Bộ "agent độc hại" (AC-M1-05.3, AC-M1-11.6), quét secret trong log (X-01), user Telegram lạ (AC-M2-01.2) | M1, M2 |
| E2E thật | Mỗi exit criteria chạy với harness thật trên repo mẫu (Node + Python), tối thiểu trước khi đóng giai đoạn | mỗi giai đoạn |

> [!NOTE]
> `FakeHarness` là điều kiện để test được hầu hết AC mà không đốt quota; nên xây ngay đầu M1.

---

## 11. Ma trận truy vết FR → User Story

| FR | Story | FR | Story |
|---|---|---|---|
| FR-1 | M1-03, M2-03, M4-03, X-08 | FR-24, 24a, 24b | M1-12, M5-04 |
| FR-2 | M1-03, M3-01, M4-05 | FR-25 | M1-14, M3-06 |
| FR-3 | M1-03, M2-03 | FR-26 | M1-10, M3-05, M3-07, M3-09 |
| FR-4 | M1-03 | FR-27 | M1-11 |
| FR-5 | M1-03, M2-03 | FR-28 | M1-05, M1-11 |
| FR-6 | M1-03 | FR-29 | M1-11 |
| FR-7 | M1-06, M1-07 | FR-30 | M3-01 |
| FR-8 | M4-06 | FR-31 | M1-11, M4-10 |
| FR-9 | M1-10 | FR-32 | M3-01, M5-06 |
| FR-10, 10a, 10b | M1-08 | FR-33, 33a | M5-01, M5-02 |
| FR-11 | M1-09 | FR-34 | M5-03 |
| FR-12 | M1-06 | FR-35 | M5-04, M5-05 |
| FR-13 | M1-13 | FR-36 | M1-16, M2-06 |
| FR-14 | M1-05, M3-02, M3-03 | FR-37 | M1-06, M1-16 |
| FR-15, 15a | M1-05, M3-03 | FR-38 | M2-05 |
| FR-16 | M1-04, M4-04 | FR-39 | M2-02 |
| FR-17 | M3-08 | FR-40 | M2-01 |
| FR-18 | M1-04, M4-05 | FR-41 | M2-09 |
| FR-19 | M1-05, M2-08, M4-08 | FR-42 | M2-07, M2-10 |
| FR-20 | M3-04 | FR-43 | M2-08 |
| FR-21 | M1-12, M4-07 | FR-44, 45 | M1-01, M1-15 |
| FR-22 | M2-04 | FR-46 | M4-11 |
| FR-23 | M1-12, M2-04 | FR-47, 48, 49 | M4-01 … M4-09 |

---

## 12. Câu hỏi mở cho Owner

1. Tên domain cụ thể (nhập lúc `hub setup`; domain phải được quản lý DNS bởi Cloudflare để dùng Tunnel + Access).

---

## 13. Lịch sử thay đổi

| Phiên bản | Thay đổi |
|---|---|
| 0.3 | Owner chốt **Cloudflare Tunnel** (+ Access) cho truy cập từ xa Webapp và preview, bỏ phương án Tailscale (D-17, AC-M4-01.2/01.5, AC-M5-01.2/01.5, AC-M5-07.1). |
| 0.2 | Owner chốt: thêm `TASK.block_reason` (+ `blocked_harness`/`blocked_model`) cho `rate_limited` (D-05, §2.1, AC-M1-05.8, AC-M2-08.3/08.4, AC-M3-05.1/05.2); `/auto on` không hiện spec/plan để duyệt nhưng xem được khi yêu cầu qua `/spec`, `/plan` (D-02, AC-M1-12.7, AC-M2-03.6, AC-M2-04.6/04.7); tin dài được chia phần hợp lý thay vì cắt cụt (D-13, §2.5, AC-M1-16.2/16.4, AC-M2-06.1/06.5). Người viết chọn theo uỷ quyền của Owner: cho tắt chặng test/review nhưng khi đó không tự merge (D-15); xoá branch `hub/*` ngay sau merge, giữ branch của task huỷ/lỗi tới khi dọn (D-16, AC-M1-11.7, `/cleanup`). |
| 0.1 | Bản đầu. |
