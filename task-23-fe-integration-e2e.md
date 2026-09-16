# Task 23 — FE tích hợp thật toàn bộ + e2e + dọn mock

**Wave:** 5 · **Người phụ trách:** D · **Effort:** 6 điểm · **Trạng thái:** [ ] Chưa làm  [ ] Đang làm  [x] Xong (2026-09-16, nhánh `feat/FLF-161-fe-integration-e2e`)

## Mục tiêu
Nối mọi panel FE vào BE thật (bỏ `NEXT_PUBLIC_API_MOCK` khỏi runtime), sửa drift so với contract phát sinh ở Wave 3–4, thêm Playwright e2e một kịch bản đầu-cuối, nhãn song ngữ từ step registry.

## Lệch hướng audit cần đóng
Đảm bảo C1/C6/C8/B4 phần FE thực sự đóng trên BE thật; dọn nợ FE còn lại.

## File / module liên quan
### Hiện có (đọc / sửa / xoá)
- Sửa: `flintflow_fe/mocks/*` (chỉ dùng trong vitest; gỡ `browser.ts` khỏi `app/layout.tsx`), `flintflow_fe/lib/api/*` (đối chiếu `pipeline.dto.ts`), `hooks/*`, mọi `_components/*` mới (T12, T16, T20), `app/home/page.tsx`, `components/ProjectCard.tsx` (`STEP_LABELS` cứng thay bằng `progress.current_step` + registry; variant theo readiness thay `progressPercent`), `components/Sidebar.tsx`.
- Sửa: `flintflow_fe/package.json` (thêm `@playwright/test`, script `e2e`), `.github/workflows/ci.yml` (job e2e: khởi BE với `AI_PROVIDER=mock` + mongo memory hoặc service, chạy Playwright).
### Tạo mới
- `flintflow_fe/e2e/{workspace.spec.ts, playwright.config.ts}` — kịch bản: đăng ký (hoặc seed user), login, onboarding, tạo project, B-0.1 trả lời, … (rút gọn qua fixture minimal seed), chạy S-2.1, gate Accept, xem Document pane có §1, Verification hiện 0 cờ đỏ, Export Word draft tải về (kiểm content-type).
- `flintflow_fe/lib/i18n.ts` (nhỏ: `t(step, "vi"|"en")` từ registry; ngôn ngữ UI theo `user.locale` mặc định vi).
- `docs/fe-architecture.md` (routing, hooks, api layer, mock trong test).

## Các bước implement
1. Tắt mock runtime; chạy toàn bộ luồng trên BE dev; ghi drift rồi sửa FE (hoặc PR `contract-change` nếu BE sai).
2. ProjectCard/home dùng dữ liệu Spine.
3. Playwright + CI job.
4. i18n nhãn; docs.

## Dependency
- Phụ thuộc: toàn bộ Wave 4; T21 (legacy đã gỡ để không còn call site cũ).
- Chặn: M5, T24 (e2e chạy trong compose).
- Chạy song song với: T21, T22, T24.

## Output kỳ vọng
- FE hoàn toàn trên BE thật; e2e xanh trên CI.

## Tiêu chí hoàn thành (DoD)
- [x] Rỗng. Gỡ hẳn nhánh bật msw trong `useWorkspace`, xoá `mocks/browser.ts` và `public/mockServiceWorker.js`; msw chỉ còn ở `mocks/server.ts` cho vitest.
- [x] Kịch bản pass — **2/2 xanh trên BE + Mongo thật tại máy** (11,6s). Job CI đã viết (dựng Mongo service + checkout BE + chạy Playwright) nhưng **chưa chạy trên GitHub**. **Lệch có chủ ý:** không dùng `AI_PROVIDER=mock` vì cách đó không tồn tại — xem mục Ghi chú.
- [x] Rỗng cả hai. Việc tiếp theo lấy từ step registry (`tStep`), trạng thái thẻ theo `readiness` thật của Spine.
- [x] typecheck sạch · lint 0 lỗi (9 warning có sẵn trên `develop`) · **207 unit test xanh** (trước: 192) · build xanh.

## Ghi chú / rủi ro
- e2e dùng mock provider BE để ổn định; luồng AI thật kiểm tay tại M4/M5.

## Kết quả (2026-09-16)

- Nhánh `feat/FLF-161-fe-integration-e2e` (FE), 1 commit, 18 file (+746 / −481).
- Đã làm thêm ngoài danh sách: gỡ `rollbackChat` khỏi `lib/api/chat.ts` — không component nào dùng từ
  T16, và `/undo` (T17) đã thay nó; endpoint cũ còn trên BE tới khi T21 xoá.
- Một dòng spec-gaps thêm thẳng vào `develop` của BE (`a5c0498`) vì phát hiện thuộc về BE, không thuộc
  nhánh FE này.

### Lệch quan trọng so với đề bài: `AI_PROVIDER=mock` không tồn tại

Task dự tính e2e chạy step S-2.1 rồi gate Accept, với BE khởi bằng `AI_PROVIDER=mock`. Kiểm thực tế:

1. **Không có biến môi trường nào ghi đè provider** — provider chọn theo frontmatter của từng skill
   (`provider: glm`), `llm.router` chỉ đọc `providerConfig.provider`.
2. Kể cả ép dùng mock, `mock.provider.ts` trả JSON cố định (`{status, message, promptSnippet}`) **không
   khớp** `opTransactionSchema`/`elicitSchema` — step sẽ chết ở bước parse, không sinh được op nào.

Nghĩa là **không thể chạy một step pipeline trong CI mà không gọi model thật**. Kịch bản vì vậy gieo nội
dung bằng op qua `POST /changes` (T17) — không cần model nhưng vẫn đi qua op engine và invariants thật,
nên vẫn kiểm đúng cái e2e cần kiểm là seam FE ↔ BE. Muốn e2e phủ luôn bước AI thì cần (a) `AI_PROVIDER`
ghi đè mọi skill và (b) mock provider trả output hợp schema theo `ActionType` — thuộc **T22/T24**, đã ghi
`docs/spec-gaps.md`. Cùng một nguyên nhân với dòng T18 về nhánh `E2E_AI=1`.

### Một phát hiện nhỏ nhưng đáng giữ

Dự án vừa tạo **đã có Spine rỗng** với `progress.current_step = B-0.1` (`spine.repository.INITIAL_STEP`),
nên thẻ dự án chỉ đúng việc đầu tiên ("Kể hết ý tưởng") thay vì "Chưa bắt đầu". Assertion đầu tiên của tôi
sai, không phải code sai — nhánh "Chưa bắt đầu" chỉ dành cho project thật sự không có Spine.
