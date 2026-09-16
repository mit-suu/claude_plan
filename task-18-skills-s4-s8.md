# Task 18 — Content skill S-4, S-5 (loop theo màn), S-6, S-7, S-8.1

**Wave:** 4 · **Người phụ trách:** B · **Effort:** 10 điểm · **Trạng thái:** [ ] Chưa làm  [x] Đang làm (mock xong, chờ lượt chạy provider thật)  [ ] Xong · nhánh `feat/FLF-158-skills-s4-s8`

## Mục tiêu
Mở rộng ngang các phase còn lại (Phases §9.3 bước 5) trên khung runner đã có: S-4 (chốt N, screen_queue, flow, authorization, non-screen, ERD), S-5 lặp theo màn (chia lô ≤ 6 function, chỉ 3–5 màn cốt lõi, còn lại placeholder), S-6 NFR có số, S-7 suy dẫn, S-8.1 glossary.

## Lệch hướng audit cần đóng
B1 (không có vòng lặp theo màn / N), B5 (context), A3 (thiếu `function:*`, §3.1.x đúng template), B8 phần scope (S-2.2 đã ở T14).

## File / module liên quan
### Hiện có (đọc / sửa / xoá)
- Đọc: `context/Product-Brief-to-SRS-Phases.md` §6.2, §6.4 (S-4…S-8.1), §7.2, §9.1; `context/srs-spine.md` §2, §6 (bất biến 5, 6, 8); `modules/pipeline/step-runner.service.ts` (T13), `step-registry.ts` (T12), `modules/diagram` (T10).
- Sửa: `flintflow_be/src/modules/pipeline/step-runner.service.ts` — nhánh S-5: `S-5.1` lấy màn `pending` đầu `screen_queue`, ghi `progress.screen_cursor`; hết queue thì vòng `@nonscreen`; `S-5.5` accept đặt `detail_status=signed_off`; hành động "để lại" đặt `placeholder` (vẫn giữ khung `functions[]`).
- Sửa: `assets/step-registry.json` nếu thiếu `reads/writes`.
### Tạo mới / điền
- `assets/skills/content/screens-and-flow/` (S-4.1 `features[]` + `screens[]` theo feature, chốt N và `screen_queue[]`, đánh dấu màn cốt lõi; S-4.2 `flow_to[]`, `is_popup`, `tabs[]`).
- `assets/skills/content/authorization-matrix/` (S-4.3 `roles[]`, `permissions[]` màn × vai trò, dòng con action).
- `assets/skills/content/non-screen-functions/` (S-4.4 `functions[screen_id=null]`: cron, webhook, engine nền).
- `assets/skills/content/entities-erd/` (S-4.5 `entities[]`, `relations[]`; gọi renderer erd).
- `assets/skills/content/function-detail/` với `references/{batching.md, validation-kinds.md, abnormal-flow-patterns.md}`: S-5.2 trigger/description/actor/mục đích/giao diện/xử lý dữ liệu; S-5.3 renderer screen_layout (chỉ màn cốt lõi); S-5.4 `normal[]`, `abnormal[]`, `validations[]{id, kind}` chia lô ≤ 6 function/lượt; Regenerate mức function.
- `assets/skills/content/nfr-quality-attributes/` (S-6.1–6.5; ngưỡng mặc định theo `stakes` × `complexity` trong `references/thresholds.md`; reliability/performance bắt buộc `metric`+`threshold`).
- `assets/skills/content/appendix-content/` (S-7.1 `business_rules[tier=detail]` suy từ `validations[kind=business]` với `source_validation_ids`; S-7.2 `common_requirements[]`; S-7.3 `messages[]` suy từ `abnormal[]` với mã; S-7.4 `other_requirements[]` từ B-1.6 + technical risk).
- `assets/skills/content/glossary/` (S-8.1 quét Spine lấy thuật ngữ/viết tắt vào `glossary[]`; user chốt ở gate).
- `fixtures/op-cases/s4-s8/*.json` cho mock; `flintflow_be/src/modules/pipeline/skills/s4-s8.e2e.test.ts` (mock + `E2E_AI=1`).
- `docs/measurements.md` mục S-4…S-8.1.

## Các bước implement
1. S-4.x skill + runner chốt N; test `screen_queue` và bất biến 8.
2. S-5 loop: runner + `function-detail` chia lô; test màn 15 function thành 3 lượt.
3. S-6, S-7, S-8.1 skill.
4. E2E mock rồi `E2E_AI=1` trên fixture minimal + kết quả T14.
5. Measurements; ghi tỉ lệ retry.

## Dependency
- Phụ thuộc: T13, T14, T10.
- Chặn: T19 (cần S-8.1 để S-9.1 chạy lại glossary), M4.
- Chạy song song với: T17, T19, T20.

## Output kỳ vọng
- 8 skill content hoàn chỉnh; runner S-5 loop; e2e S-4 đến S-8.1.

## Tiêu chí hoàn thành (DoD)
- [x] Mock e2e S-4.1 → S-8.1 xanh (`s4-s8.e2e.test.ts`); N = 4 màn + vòng `@nonscreen` ⇒ `totalSteps = 51 + 5×5 = 76`.
- [ ] `E2E_AI=1`: **chưa chạy**. Provider có sẵn, nhưng nhánh test này (khung của T14) không chạy được như đang viết: `executeAiAction` cần Mongo thật để reserve/deduct credit và ghi `AiActionLog`, mà tiến trình vitest không nối Mongo — ghi ở `docs/spec-gaps.md` (commit `bc80459`). Trên mock thì cả hai điều kiện đã xanh (0 cờ đỏ không waive được, `nfr_missing_number` = 0). Cách chạy thật đề xuất: qua API trên BE thật, như T14 đã làm cho run12–20.
- [x] Màn 15 function chia **6 + 6 + 3** lượt cho cả S-5.2 và S-5.4, các lô rời nhau và phủ đủ; màn 2 function vẫn một lượt. `5 × N` step không đổi (chia lô nằm trong step).
- [x] Màn `placeholder` giữ nguyên khung `functions[]` do S-4.1 sinh, `nextStep` bỏ qua vòng của nó, và không có cờ `screen_pending_at_baseline` nào mở.

## Ghi chú / rủi ro
- Chỉ 3–5 màn cốt lõi chi tiết (Phases §9.1) — chọn: Project Workspace, Verification & Change, Export, Project Dashboard, Plan & Pricing.

## Kết quả (2026-09-16)

- Nhánh `feat/FLF-158-skills-s4-s8`, 3 commit, 35 file (+2912 / −103).
- 8 content skill bỏ `stub`, mỗi file ≤ 150 dòng (101–128); thêm 4 file `references/` cho người đọc.
- Runner: `LOOP_BOOKKEEPING_TEMPLATES` (S-5.1/S-5.5 không gọi model), `loopCursorOps`,
  `functionBatches` + `batchContext` (lô ≤ 6 function); gate: `accept_as_is` trên S-5.1 ⇒ `placeholder`.
- 17 fixture op-case; `s-5.2/s-5.4` là template `${fn}` mở theo đúng lô runner đưa vào projection.
- BE: typecheck sạch, **570 test xanh / 14 skip** (develop trước đó 567; +3 ca của file e2e mới).
- Số đo token/lượt gọi từng step ở `docs/measurements.md`; 0/46 lượt phải retry schema (mock).
- Việc còn lại: chốt cách chạy provider thật (qua API trên BE thật — khung `E2E_AI=1` của vitest không nối Mongo), chạy S-4.1 → S-8.1, ghi số + tỉ lệ retry thật vào `docs/measurements.md`.
