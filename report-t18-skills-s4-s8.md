# BÁO CÁO TASK T18 — Content skill S-4, S-5 (loop theo màn), S-6, S-7, S-8.1 · Wave 4 · Người: B · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Đang làm** — toàn bộ code + e2e mock đã xong; chỉ còn lượt chạy `E2E_AI=1` (phiên này không có API key provider)
- Nhánh: `feat/FLF-158-skills-s4-s8` · Commit cuối: `27c0008` · PR: chưa mở (chờ phương án merge/push)
- % ước lượng hoàn thành: 90% · Effort đã dùng / ước lượng: 9 / 10 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | S-4.x skill + runner chốt N; test `screen_queue` và bất biến 8 | Xong — 4 skill S-4; e2e kiểm `screen_queue`, `totalSteps = 51 + 5×5`; bất biến 8 không bị chạm vì màn thêm ở phase S-4 |
| 2 | S-5 loop: runner + `function-detail` chia lô; test màn 15 function thành 3 lượt | Xong — `FUNCTION_BATCH_SIZE = 6`, màn 15 function chia **6 + 6 + 3** cho cả S-5.2 và S-5.4, lô rời nhau |
| 3 | S-6, S-7, S-8.1 skill | Xong — `nfr-quality-attributes` (+ `references/thresholds.md`), `appendix-content`, `glossary` |
| 4 | E2E mock rồi `E2E_AI=1` trên fixture minimal + kết quả T14 | **Một phần** — mock xanh; `E2E_AI=1` đã viết, `skipIf` đúng biến môi trường, **chưa chạy** (thiếu key) |
| 5 | Measurements; ghi tỉ lệ retry | Xong cho mock — bảng token/lượt gọi từng step trong `docs/measurements.md`, retry **0/46**; số thật chờ lượt `E2E_AI=1` |

## 3. File đã thay đổi (đối chiếu bảng vùng sở hữu trong coding-rules.md mục 2)
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `assets/skills/content/screens-and-flow/SKILL.md` | sửa | S* | `assets/skills/content/` cột T18 = `S*` |
| `assets/skills/content/authorization-matrix/SKILL.md` | sửa | S* | |
| `assets/skills/content/non-screen-functions/SKILL.md` | sửa | S* | |
| `assets/skills/content/entities-erd/SKILL.md` | sửa | S* | |
| `assets/skills/content/function-detail/SKILL.md` | sửa | S* | |
| `assets/skills/content/function-detail/references/{batching,validation-kinds,abnormal-flow-patterns}.md` | tạo | S* | tài liệu cho người, không nạp runtime |
| `assets/skills/content/nfr-quality-attributes/SKILL.md` | sửa | S* | |
| `assets/skills/content/nfr-quality-attributes/references/thresholds.md` | tạo | S* | |
| `assets/skills/content/appendix-content/SKILL.md` | sửa | S* | |
| `assets/skills/content/glossary/SKILL.md` | sửa | S* | |
| `src/modules/pipeline/step-runner.service.ts` | sửa | S* | task ghi rõ "Sửa … step-runner.service.ts — nhánh S-5" |
| `src/modules/pipeline/gate.service.ts` | sửa | S* | hành động "để lại" ⇒ `placeholder` (task ghi rõ, cùng hàng bảng vùng sở hữu) |
| `src/modules/pipeline/skills/s4-s8.e2e.test.ts` | tạo | S | hàng `pipeline/skills/*.e2e.test.ts` cột T18 = `S` |
| `fixtures/op-cases/s4-s8/*.json` (17 file) | tạo | S* | hàng `fixtures/` cột T18 = `S* (op-cases/s4-s8)` |
| `docs/measurements.md` | sửa | thêm mục | T18 được ghi thêm mục, không xoá mục người khác |
| `docs/spec-gaps.md` | sửa | thêm dòng | ai cũng được thêm dòng |
| `src/shared/ai/prompt-assets.test.ts` | sửa | **R** | ⚠ xem mục 4 |

Tổng: **35 file · +2912 / −103** (`git diff --stat develop...HEAD`)

## 4. Thay đổi ngoài vùng sở hữu (phải rỗng, hoặc có XREQ được granted)
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| `src/shared/ai/prompt-assets.test.ts` (+11 / −3, chỉ danh sách `WRITTEN_NON_ACTION`) | Test của T03 khoá "mọi skill không phải action = `stub: true`". 8 skill của T18 viết thật thì **bắt buộc** bỏ `stub`, nếu không `draftOps#contentGuidance` gửi câu "skill chưa viết" thay cho nội dung skill — tức là cả task không có tác dụng. | `[XREQ][T18→T03]` — **chưa mở issue**, làm theo đúng tiền lệ đã chốt 2026-09-15 cho T10 (5 renderer) và T14 (4 content skill), cùng một dòng danh sách, cùng một lý do. | **Chưa có** — cần B/T03 xác nhận trước khi merge |

## 5. Hợp đồng / interface bị ảnh hưởng
- Có đụng hợp đồng đóng băng không: **Không**. `assets/step-registry.json` **không sửa** (đã có đủ `reads`/`writes`/`renders` cho S-4…S-8.1); `pipeline.dto.ts`, `pipeline-contract.md`, `op.types.ts` không đụng. Hai chỗ lẽ ra muốn sửa registry đều xử lý trong code và ghi `docs/spec-gaps.md` (xem mục 7).
- Interface mới mà task khác sẽ dùng (`step-runner.service.ts`):
  - `LOOP_BOOKKEEPING_TEMPLATES`, `FUNCTION_BATCH_SIZE`
  - `loopCursorOps(spine, stepId)`, `nextPendingScreen(spine)`, `functionBatches(spine, stepId)`, `batchContext(ctx, functionIds)`
  - `gate.service`: `accept_as_is` trên `S-5.1@<màn>` ⇒ `screens[].detail_status = "placeholder"` (T19 cần biết khi đếm màn ở baseline)

## 6. Kiểm chứng (dán output thật, không mô tả)
- `npx tsc --noEmit` (BE): **pass**, không output.
- `npm run lint` (FE): không chạy — T18 không sửa file FE nào.
- `npx vitest run` (BE):
  ```
  Test Files  56 passed | 1 skipped (57)
       Tests  570 passed | 14 skipped (584)
    Duration  9.50s
  ```
  (develop trước đó: 567 test; +3 ca của `s4-s8.e2e.test.ts`). Test skip thêm **1** so với develop: ca `E2E_AI=1` của T18 — lý do: **không có API key provider trong phiên này**, không phải skip để né lỗi.
- File e2e của task: `npx vitest run src/modules/pipeline/skills/s4-s8.e2e.test.ts` → `3 passed | 1 skipped (4)`.
- Bằng chứng chia lô (in ra từ chính lượt chạy):
  ```
  [T18 draft calls per step] [... {"step":"S-5.2@S91","n":6},{"step":"S-5.2@S91","n":6},{"step":"S-5.2@S91","n":3},
                                  {"step":"S-5.4@S91","n":6},{"step":"S-5.4@S91","n":6},{"step":"S-5.4@S91","n":3},
                                  {"step":"S-5.2@S92","n":2},{"step":"S-5.4@S92","n":2}, ...]
  ```
- Số dòng từng SKILL.md (trần 150): screens-and-flow 111 · authorization-matrix 101 · non-screen-functions 103 · entities-erd 102 · function-detail 128 · nfr-quality-attributes 111 · appendix-content 120 · glossary 102.
- DoD trong file task: **3/4 tick**. Mục chưa tick: `E2E_AI=1` — chưa chạy, thiếu key.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| Không có API key provider ⇒ chưa chạy `E2E_AI=1` (DoD 2 và mục "tỉ lệ retry thật") | Người có key | Chạy `E2E_AI=1 npx vitest run src/modules/pipeline/skills/s4-s8.e2e.test.ts`, dán số vào `docs/measurements.md` rồi tick DoD | **Cao** — chặn M4 |
| `WRITTEN_NON_ACTION` trong test của T03 (mục 4) | B / chủ T03 | Xác nhận `granted` theo đúng tiền lệ T10/T14 | Trung bình — chặn merge |
| Registry để S-5.1/S-5.5 `deterministic: false` nhưng hai step này không cần model | Cả nhóm | Giữ registry (đóng băng), bỏ qua Draft trong runner (`LOOP_BOOKKEEPING_TEMPLATES`). Nếu mở PR `contract-change` sau này thì đổi `deterministic: true` và gỡ hằng số | Thấp |
| Không có hành động gate nào nghĩa là "để lại màn" | Cả nhóm | Dùng `accept_as_is` trên S-5.1 (bắt buộc `note`, mở cờ vàng `accepted_as_is` — đúng là lời giải thích) thay vì thêm hành động mới vào `gateRequestSchema` đã đóng băng | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí (file:dòng) | Vấn đề | Thuộc task nào | Đã ghi docs/spec-gaps.md? |
|---|---|---|---|
| `src/modules/spine/op-engine.ts` `applySet` (mảng phần tử có `id`) | `set` cả mảng `functions[].validations` bị từ chối (`op_not_allowed`) — task-18 và bản nháp skill đều viết `set`. Đúng của engine; đã sửa **skill + fixture**, không sửa engine | T08 (engine đúng) | Có |
| `assets/step-registry.json` S-7.1 `writes` | S-7.1 chỉ được ghi `business_rules`, `assumptions` ⇒ không thể nối `functions[].business_rule_ids`; cạnh truy vết chỉ một chiều qua `source_validation_ids` | T12 (registry) | Có |
| `src/modules/pipeline/draft-to-ops.ts#contentGuidance` | Vẫn không nạp `references/` cho skill content (quyết định Wave 3). Hệ quả: mọi luật phải nén vào `SKILL.md` ≤ 150 dòng — làm được cho cả 8 skill, nên **không đề nghị mở lại** | T11 | Đã có từ T14 |

## 9. Bước tiếp theo
- Việc còn lại của task này: (1) xin `granted` cho XREQ T18→T03; (2) chạy `E2E_AI=1`, ghi số thật + tỉ lệ retry vào `docs/measurements.md`, tick DoD 2 → trạng thái "Xong".
- Ảnh hưởng tới merge point M4: **Có** — M4 đòi một lượt đi trọn B-0.1 → S-9.5 với provider thật; lượt `E2E_AI=1` của T18 là mảnh S-4→S-8.1 của lượt đó.
- Đề xuất: merge T18 trước T19 (T19 cần S-8.1 để S-9.1 quét lại glossary, và cần biết `placeholder` để đếm màn ở baseline).
