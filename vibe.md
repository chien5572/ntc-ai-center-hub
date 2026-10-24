## 1. Viết SPEC.md
> **1.1**
``` text
Dựa vào @PRD.md và @ARCHITECTURE.md, Hãy viết 1 bản SPEC hoàn chỉnh hơn với User Stories rõ rang và Acceptance Criteria cho từng giai đoạn.
```
> **1.2**
``` text
Dựa vào @SPEC.md tổng quan hãy viết cho tôi 1 SPEC.<slug>.md hoàn chỉnh hơn với User Stories rõ rang và Acceptance Criteria từng giai đoạn cho service .
```
## 2. Plan:
``` text
Bắt đầu với @AGENTS.md hãy đọc bản @SPEC.md trước khi chúng ta bắt đầu làm bất cứ một việc gì, mình muốn bạn hãy hiểu codebase hiện tại. Đọc @AI-skill-help.md và khám phá các skills tương ứng cho từng chức năng. Từ đó sẽ phát huy hết khả năng của các bộ skills, công cụ.
```

## 3. Áp dung skills
> Tìm skills phù hợp
``` text
Dựa vào web skill AI này "https://skillsmp.com/search" và web plugin AI này "https://felo.ai/vi/skills", hãy đọc [SPEC.md](file;file:///d:/vibe-code/onl247/SPEC.md) và đề xuất các skill/plugin tương ứng để tôi cài vào dự án này để tăng sức mạnh cho agents. Đề xuất trước, tôi duyệt thì mới cài.
```
> Tạo file **AI-skill-help.md**
``` text
Tạo cho tôi 1 file AI-skill-help.md ở thư mục gốc dựa án, ghi lại cách sử dụng từng skill này thế nào, dùng khi nào, cho chức năng/module/servies nào nhưng 1 bản hướng dẫn sử dụng để tôi đính kèm cho agents biết cách dùng chúng.
Sau đó cài toàn bộ chúng vào thư mục chứa các skill local của dự án này (bao gồm cả những skill custom)
```
## 4. Viết STEP.md để biết được các bước thực thi.

## 5. Viết cho từng service.
``` text
Tôi muốn bắt đầu triển khai Module CORE (core-service - Phần Nền tảng) độc lập ở tab này.
Căn cứ vào:
- @AI-skill-help.md (BẮT BUỘC đọc và áp dụng các skill: `atomic-harness-gatekeeper`, `onl247-domain-invariants`, `nestjs-skills`, `postgres-patterns`, `redis-patterns`)
- @docs/plans/contract-registry.md (hợp đồng CT-SYS-* đã AVAILABLE)
- @services/core-service/SPEC.domain-model.md
- @services/core-service/SPEC.identity.md
- @services/core-service/SPEC.layout.md
- Quy tắc AGENTS.md: chỉ tạo/sửa file trong services/core-service/ và docs/plans/

Hãy thực hiện:
1. Gate 3: Lập Module Blueprint tại docs/plans/modules/core-service.md
2. Gate 4: Lập Phase Dossier cho Phase CORE-P01 tại docs/plans/active/core-service/phase-01-domain-and-identity.md và phân rã các Atomic Tasks.
Dừng lại ở Gate 4 và báo cáo bảng task để tôi duyệt trước khi viết code.

````
> Hoặc làm việc với git worktree. VD Module CORE (core-service - Phần Nền tảng) trên một Git Worktree độc lập.
``` text
A. Thiết lập Không gian làm việc Cô lập (Worktree):
- Hãy kiểm tra và tạo git worktree tại thư mục `../onl247-core` trên nhánh mới `feat/core-service` tách từ `main`:
  `git worktree add ../onl247-core -b feat/core-service main`
- Mọi thao tác tạo/sửa file chỉ được thực hiện trong thư mục worktree này và phạm vi `services/core-service/` + `docs/plans/`.

B. Căn cứ Kỹ thuật & Kỹ năng Bắt buộc:
- @AI-skill-help.md (BẮT BUỘC đọc & áp dụng: `using-git-worktrees`, `atomic-harness-gatekeeper`, `onl247-domain-invariants`, `nestjs-skills`, `postgres-patterns`, `redis-patterns`)
- @docs/plans/contract-registry.md (hợp đồng CT-SYS-* đã AVAILABLE)
- @services/core-service/SPEC.domain-model.md (DDL PostgreSQL, phân vùng project_id)
- @services/core-service/SPEC.identity.md (Auth JWT, RBAC, Tổ/Ca B3, seed admin/123456)
- @services/core-service/SPEC.layout.md (Cấu hình N cấp PHONG/HOP/HO_SO, Schema Layout Bìa/Văn bản)

C. Quy trình Thực hiện:
1. Gate 3: Lập Module Blueprint tại docs/plans/modules/core-service.md
2. Gate 4: Lập Phase Dossier cho Phase CORE-P01 tại docs/plans/active/core-service/phase-01-domain-and-identity.md và phân rã các Atomic Tasks.
Dừng lại ở Gate 4 và báo cáo bảng task để tôi duyệt trước khi viết code.

D. Nghiệm thu & Merge vào Main:
Khi hoàn thành toàn bộ Atomic Tasks của module: Chạy test NestJS (`npm run test`), chạy `powershell scripts/validate-plans.ps1`, commit gọn gàng, checkout về `main`, merge nhánh `feat/core-service` vào `main` và xóa worktree.

````
## Khác
> Vấn đề "Tiếp tục"
``` text
khi tôi nói "tiếp tục" nghĩa là triển khai task tiếp theo (nếu hết task thì báo, để tôi ra lệnh viết phase mới).
Tiếp tục
```
