# BÁO CÁO TASK T17 — Change flow: impact, 3 nhánh, preview diff, stale, hoà giải, undo, traceability · Wave 4 · Người: A · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Xong** (còn một vế DoD phụ thuộc T19)
- Nhánh: `feat/FLF-157-change-flow` · Commit cuối: `edf50f8` · PR: chưa mở (chờ phương án merge/push)
- % ước lượng hoàn thành: 95% · Effort đã dùng / ước lượng: 9 / 9 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | `impact.service` + test trên fixture (đổi `actors[id=A01].name`) | Xong — `impactOf`, `impactOfChanges`, `referrersOf`, `diagramsToRerender`; 10 test |
| 2 | `change.service` preview/apply ops thuần, rồi nhánh instruction qua skill (mock) | Xong — 3 nhánh `silent/dependent/post_baseline`, `clarification`, kho `preview_id`; 36 test |
| 3 | `undo`, `traceability` + routes | Xong — `undoLast`/`findUndoTarget` (10 test), `trace`/`buildTraceLinks` (9 test), 6 endpoint |
| 4 | `reconcile` + render lại diagram | Xong — preview gộp → apply → vẽ lại hình lệch `source_hash` → `awaiting_reaccept`; 11 test |
| 5 | Nối FE ChangePanel (T16) bỏ mock; kiểm cùng D | **Không phải sửa FE** — panel T16 đã viết theo contract, msw chỉ bật khi `NEXT_PUBLIC_API_MOCK=1`. Kiểm trình duyệt với BE thật còn treo (M4, cùng D) |

## 3. File đã thay đổi (đối chiếu bảng vùng sở hữu trong coding-rules.md mục 2)
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `src/modules/spine/impact.service.ts` | tạo | S | hàng `{impact,change,reconcile,undo,traceability}.service.ts` |
| `src/modules/spine/impact.test.ts` | tạo | S | |
| `src/modules/spine/change.service.ts` | tạo | S | |
| `src/modules/spine/change.service.test.ts` | tạo | S | |
| `src/modules/spine/reconcile.service.ts` | tạo | S | |
| `src/modules/spine/reconcile.test.ts` | tạo | S | |
| `src/modules/spine/undo.service.ts` | tạo | S | |
| `src/modules/spine/undo.test.ts` | tạo | S | |
| `src/modules/spine/traceability.service.ts` | tạo | S | file mới; bản legacy ở `modules/specification/` **không đụng**, T21 xoá |
| `src/modules/spine/traceability.test.ts` | tạo | S | |
| `src/modules/spine/changes.controller.ts` | sửa | S* | task ghi rõ "route `changes` (T08) thêm nhánh instruction, reconcile, undo, traceability" |
| `src/modules/spine/changes.route.ts` | sửa | S* | 4 route mới + swagger |
| `src/modules/spine/changes.controller.test.ts` | sửa | S* | viết lại theo kiến trúc mới (controller mỏng) — 19 test, nhiều hơn 9 test cũ |
| `src/modules/project/chat-session.service.ts` | sửa | S* | task ghi rõ "tin nhắn dạng lệnh sửa chuyển vào `change.service`" |
| `src/modules/project/chat-session.service.test.ts` | sửa | S* | +5 test cho nhánh trên; 7 test cũ giữ nguyên |
| `assets/skills/action/apply-change-op/SKILL.md` | sửa | S* | `assets/skills/action/` cột T17 = `S* (apply-change-op)` |
| `docs/spec-gaps.md` | sửa | thêm dòng | ai cũng được thêm dòng |

Tổng: **17 file · +3081 / −137** (`git diff --stat develop...HEAD`)

## 4. Thay đổi ngoài vùng sở hữu (phải rỗng, hoặc có XREQ được granted)
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| Không có | | | |

## 5. Hợp đồng / interface bị ảnh hưởng
- Có đụng hợp đồng đóng băng không: **Không**. `pipeline.dto.ts` và `pipeline-contract.md` **không sửa một dòng nào** — mọi schema T17 cần (`changesRequestSchema` có `instruction`/`preview_id`, `impactSchema`, `reconcileRequestSchema`, `undoRequestSchema`, `changesQuerySchema`, `traceabilityQuerySchema/ResponseSchema`) đã có sẵn từ T08. Mã lỗi cũng dùng lại bảng §0.3 (kể cả `preview_id` hết hạn → `422 CHANGE_RANGE_INVALID`, đúng mã mock FE đang trả).
- Interface mới mà task khác sẽ dùng:
  - `impact.service.ts`: `impactOf(spine, paths, {hints?, after?})`, `impactOfChanges(spine, changes, after?)`, `referrersOf`, `diagramsToRerender` — T19 dùng cho impact analysis trước baseline.
  - `change.service.ts`: `preview()`, `apply()`, `branchOf()`, `isChangeInstruction()`, `NeedsClarificationError`, `ChangeDeps` (DI cho executor + recompute cờ).
  - `reconcile.service.ts`: `reconcile()`, `staleSections()`, `changesMakingStale()`, `isReconcileApplied()`.
  - `undo.service.ts`: `undoLast()`, `findUndoTarget()` (hàm thuần).
  - `traceability.service.ts`: `trace(spine, {entity, id}, {depth?})`, `buildTraceLinks(spine)`.
  - Endpoint: `POST /changes` (nhánh `instruction`), `POST /reconcile`, `POST /undo`, `GET /changes`, `GET /traceability`.

## 6. Kiểm chứng (dán output thật, không mô tả)
- `npx tsc --noEmit` (BE): **pass**, không output.
- `npm run lint` (FE): không chạy — T17 không sửa file FE nào.
- `npx vitest run` (BE):
  ```
  Test Files  60 passed | 1 skipped (61)
       Tests  658 passed | 13 skipped (671)
    Duration  12.30s
  ```
  (develop trước đó: 567 test). Test skip là các ca `E2E_AI=1`/PlantUML thật đã có từ trước, không phải của T17.
- Test theo module T17: `npx vitest run src/modules/spine/` → `21 passed (21) · 257 passed (257)`.
- Test chạy thủ công (curl/UI/docker): **chưa** — chưa có BE chạy + Mongo trong phiên này.
- DoD trong file task: **4/5 tick**. Mục chưa tick trọn: "sau baseline… còn cờ đỏ thì không tạo baseline mới" — vế `reason` bắt buộc → 400 đã kiểm; vế baseline thuộc **T19** (`POST /baseline` chưa tồn tại).

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| Undo ghi `op: "revert"` chứ không phải `op: "undo"` như task-17 mô tả | Cả nhóm | Giữ `"revert"`: `op.types.ts` đã đóng băng và contract §0.2 quy định revert. Nhận diện lô undo qua `reason: "Revert seq N"`, đã test. Nếu muốn `"undo"` thì phải mở PR `contract-change`. | Thấp |
| DoD "3 section + 2 diagram" khi đổi tên actor | A + D | Với actor `kind=human` chỉ có **1** hình vẽ lại (hình ngữ cảnh chỉ chiếu actor phi-human). Đã kiểm ca 2 hình bằng actor `kind=system`. Đề xuất sửa câu DoD cho khớp thực tế. | Thấp |
| Kho `preview_id` nằm trong bộ nhớ tiến trình | C (T24) | Chấp nhận giới hạn single-instance như `pendingAnswers`/khoá step của T13; hết hạn chỉ tốn một lần xem lại diff. Nếu scale ngang thì chuyển sang Redis cùng lúc với khoá step. | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí (file:dòng) | Vấn đề | Thuộc task nào | Đã ghi docs/spec-gaps.md? |
|---|---|---|---|
| `src/modules/spine/section-status.ts:66` (`computeSectionStates`) | User sửa thẳng một field qua chat làm **cả section sở hữu** field đó thành `stale`, không chỉ section "đọc" — đúng §5 nhưng dễ bị hiểu là lỗi | T09 (đúng thiết kế, không sửa) | Có |
| `src/modules/pipeline/pipeline.dto.ts:27` (`PIPELINE_ERROR_STATUS`) | Không có mã cho "`preview_id` hết hạn"; phải mượn `CHANGE_RANGE_INVALID` (tên lệch nghĩa) | T08 (contract) | Có |
| `src/modules/specification/traceability.service.ts` | Bản traceability cũ (source link tài liệu upload) vẫn còn, khái niệm khác hẳn bản mới | T21 (xoá legacy) | Không (đã có trong plan T21) |

## 9. Bước tiếp theo
- Việc còn lại của task này: kiểm trên trình duyệt với BE thật + Mongo (Change panel: preview → apply → undo → reconcile → traceability), cùng D. Không còn việc code.
- Ảnh hưởng tới merge point M4: **Có** — M4 đòi "FE ChangePanel chạy thật (không mock)", cần một lượt kiểm trình duyệt sau khi merge.
- Đề xuất: merge T17 trước T19 để T19 dùng lại `impactOf` cho Impact Analysis trước baseline; sau khi T19 có `POST /baseline` thì bổ sung một test đóng nốt DoD 4.
