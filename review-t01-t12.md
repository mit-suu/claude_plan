# Review code T01–T12 — 2026-09-14

Code review: BE và FE trên nhánh `wave2/review`, tức `develop` (wave 1) cộng các nhánh wave 2.

Cách chấm:
- **Xong** = DoD đạt và không có lỗi mức cao.
- **Đang làm** = còn DoD mở, hoặc có lỗi cao.

Mọi lỗi dưới đây đã được đối chiếu với code. Ô "Tin cậy" ghi độ chắc chắn khi kịch bản chỉ suy ra từ code mà chưa chạy thật.

Test lúc review:
- BE: 40 file, 406 test xanh, typecheck sạch.
- FE: 13 file, 101 test xanh, typecheck sạch, lint 0 lỗi.
- Chưa chạy `next build`; chưa mở file docx bằng LibreOffice.

> **Cập nhật sau lượt sửa (cùng ngày):** mọi lỗi cao và trung đã sửa hoặc đóng theo quyết định của nhóm — xem mục [Kết quả sửa](#kết-quả-sửa-2026-09-14). Test sau sửa: BE 41 file / 428 test, FE 13 file / 102 test, typecheck hai repo sạch, FE lint 0 lỗi.

## Tổng quan

Cột "Sau sửa" là trạng thái hiện tại; cột lỗi giữ số lúc review.

| Task | Lúc review | Sau sửa | Lỗi cao | Lỗi trung | Việc còn mở |
|---|---|---|---|---|---|
| T01 | Xong | Xong | 0 | 1 | Test repository mới chạy trên model mock |
| T02 | Đang làm | Xong | 1 | 0 | Chưa chạy seed trên Mongo thật |
| T03 | Đang làm | Xong | 1 | 1 | — (bỏ prompt-template theo quyết định) |
| T04 | Xong | Xong | 0 | 3 | Lỗi thấp phía `payment_service` (xem dưới) |
| T05 | Đang làm | Đang làm | 0 | 1 | Mở file docx bằng Word/LibreOffice |
| T06 | Xong | Xong | 0 | 1 | ai-cost chưa đối chiếu DB |
| T07 | Xong | Xong | 0 | 0 | `next build` chưa kiểm |
| T08 | Đang làm | Đang làm | 0 | 6 | Contract cần 4/4 người approve |
| T09 | Xong | Xong | 0 | 1 | — |
| T10 | Xong | Xong | 0 | 4 | — |
| T11 | Đang làm | Xong | 1 | 1 | — |
| T12 | Xong | Xong | 0 | 3 | Chưa chạy tay trên trình duyệt |

## Lỗi mức cao (sửa trước M1/M2)

| # | Task | Vị trí | Vấn đề | Kịch bản |
|---|---|---|---|---|
| H1 | T02 | `flintflow_be/src/scripts/seed-fixture.ts:90-96` | Ghi `project_id`, trong khi model và repository dùng `projectId`. Script không `spineSchema.parse`, không đi qua repository. | Seed xong gọi `GET /spine` thì không thấy dữ liệu fixture, `getOrCreate` còn tạo thêm một Spine rỗng. Seed project thứ hai dễ gặp E11000, vì unique index gặp `projectId` null. |
| H2 | T03 | `src/modules/admin/` (commit `1feac24` FLF-146) | Commit này xoá `prompt-template.controller.ts`/`route.ts` do T03 thêm (PUT trả 410), xoá cả GET. | `PUT /api/v1/admin/prompt-templates/chat` trả 404 thay vì 410; admin không còn cách nào xem prompt. |
| H3 | T11 | `src/modules/pipeline/draft-to-ops.ts:229` | Lấy `txn` do model trả. Ví dụ trong SKILL.md có chuỗi mẫu cho `txn`, model dễ chép lại nguyên văn. | Hai draft mang cùng một txn. Nếu `saveWithVersion` trả 409, bước bù trừ `deleteMany({txn})` xoá luôn change của txn cũ đã commit. **Sửa:** engine luôn tự sinh txn, bỏ qua giá trị của model. |

## Lỗi mức trung

### T01
- `project.service.ts:58-66`: `deleteProject(hard)` không xoá `Spine`, `Change`, `Baseline`, `Usage`, để lại document mồ côi (baseline chứa snapshot lớn).

### T03
- `src/config/startup-checks.ts:15-25`: không gọi `getSkillIndex()`, nên skill sai frontmatter vẫn boot được và chỉ lỗi khi được gọi tới.

### T04
- `shared/ai/credit-reservation.service.ts:203-242`: `deductCredit` đổi `state=deducted` trước rồi mới trừ ví, không nằm trong transaction. Nếu trừ ví thất bại (`balance < cost`) thì release không chạy, `reserved` bị treo vĩnh viễn.
- `credit-reservation.service.ts:219-229`: nhánh reservation hết hạn không xét `reserved`, nên số khả dụng có thể âm; gọi lại có thể trừ hai lần.
- `billing.service.ts:383-417` + `billing.route.ts:172`: `POST /billing/upgrade` cho tự lên Pro không cần thanh toán.

### T05
- `src/app.ts:84`: `express.json()` giới hạn 100KB, nên `POST /export/word/preview` có ảnh base64 lớn hơn khoảng 75KB trả 413.

### T06
- `shared/auth/admin.middleware.ts:18-25`: role đọc từ JWT, `authMiddleware` không kiểm `isActive`. Admin bị hạ quyền hoặc bị khoá vẫn gọi được `/admin/*` cho tới khi token hết hạn.

### T08
- `spine.repository.ts:219-237` — hai tab cùng ghi để lại lỗ seq:
  - Hai tab cùng base. Bên thua đã chèn seq 5,6 rồi mới xoá; bên thắng đã lấy seq 7. `changes[]` còn …4, 7, nên `revertRange` qua dải này trả 422.
  - `insertMany` ordered lỗi ở vị trí i>0 thì các phần tử đã chèn trước đó không được dọn.
- `op-engine.ts:340-347` — revert một op xoá không kiểm id trùng. Kịch bản: xoá A1, thêm A1 mới, revert op xoá ⇒ hai A1; từ đó mọi path `actors[id=A1]` báo `path_ambiguous`.
- `op-engine.ts:376-380` — revert không so giá trị hiện tại với `change.value`, nên ghi đè im lặng sửa đổi sau (X→Y→Z, revert bước đầu ⇒ X). Rủi ro cho resume/regenerate ở T13.
- `invariants.ts:203-226` — undo việc xoá một màn đã làm xong trong S-5 bị chặn bởi `invariant_8_screen_not_pending`.
- `invariants.ts:31-37,133` — `onlyNew` so theo message, mà message bất biến 5/6 chứa danh sách order. Spine legacy lỗi sẵn thêm một feature thì message đổi, bị tính là vi phạm mới, lô bị từ chối.
- `changes.controller.ts` — `POST /changes` không có whitelist path:
  - User `set flags[id].waived_by_user` được, qua mặt luật cấm waive và yêu cầu lý do ≥ 20 ký tự.
  - User sửa được `steps[].status`, `progress`, `baselines[]`.
  - **Sửa:** chặn các gốc `flags`, `steps`, `progress`, `baselines`, `sections` trên route của user.

### T09
- `flags.service.ts:106-111` — recompute thường (không `atBaseline`) đóng cờ của các luật chỉ chạy ở S-9, dù điều kiện vẫn còn. Lần recompute baseline sau mở cờ mới chưa waive, nên waiver cũ bị mất. **Sửa:** không đóng cờ thuộc luật `at_baseline` khi đang recompute thường.

### T10
- `diagram.service.ts:143,185` — `unchanged()` so `puml` gốc với bản đã auto-fix đang lưu, nên mỗi lần `renderAll` đều ghi lại và tăng `spine_version`. Hình ở trạng thái `error` cũng bị ghi lại mỗi lần.
- `diagram.service.ts:103-106,194,210` — ghi GridFS trước `applyTransaction`. Hai lần render song song cùng cấp id `D05`: một bên nhận 409 nhưng file của nó có thể đã đè file của bên kia.
- `shared/diagram/compile-check.ts:34-39,66` — quét marker `syntax error` trong SVG, mà SVG chứa nguyên văn nhãn. Use case tên "Report syntax error" bị đánh lỗi oan.
- `diagram.service.ts:55` — `dropErrorLine` đoán cách đánh số dòng. Bỏ dòng có thể làm hình thành `ok` nhưng mất phần tử (không báo gì), hoặc làm vỡ `}` của composite state.

### T11
- `op-validator.ts:55` — `ops: []` được coi là hợp lệ, nhưng `transactionSchema` yêu cầu `ops.min(1)`, nên T13 gọi `applyTransaction` sẽ nhận 422 mà không được retry.

### T12
- `useStepRunner.ts:135-141` — stream SSE đóng mà không có `gate_ready`/`error` thì UI kẹt ở `drafting` với `busy=true`, không có nút thoát.
- `useStepRunner.ts:135` — không dùng `AbortController`; unmount hoặc đổi project không huỷ stream, event cũ vẫn được dispatch.
- `useSpine.ts` + `page.tsx:57-59` — `reload` không bỏ response cũ, nên GET v10 về muộn đè `versionRef` sau khi đã `replaceSpine` v11, dẫn tới 409 ở lần ghi sau.

## Lỗi mức thấp (gom)

- **T01**
  - Project và Spine được tạo ngoài transaction.
  - `baseline.model` không có unique `(projectId, version)`.
- **T02**
  - Case-05 dùng `set` thay vì op `renumber`.
  - Test fixture không cho phép `renumber`.
- **T03**
  - `DIAGRAM_CLASSIFY` và `DIAGRAM_GENERATE` còn trong enum; gọi tới trả 500.
  - Còn code chết Excalidraw ở `response-parser.ts:29,192`.
- **T04**
  - Order bị timeout vẫn có thể đã tạo phía `payment_service`.
  - Intent `failed` không nhận tiền về muộn.
  - `balanceAfter` tính không nhất quán.
  - Notify gọi ngay trong transaction.
- **T05**
  - `markdown-to-blocks.ts:19,57`: dòng `---` sau dòng có `|` bị hiểu thành bảng.
  - Đoạn bắt đầu bằng "2026." bị hiểu thành numbered list.
- **T06**
  - Điều kiện `state` trong ai-cost là code chết.
  - `lib/api/admin.ts` không `encodeURIComponent` id.
- **T07**
  - Google client ID thật trong `.env.local` (file không track).
  - Thư mục `packages/excalidraw/` rác.
  - 6 trang `(auth)` gọi `fetch` thô.
- **T08**
  - Phần tử vô hướng chứa `[` hoặc `]` không remove/revert được.
  - Mảng khoá có id lặp làm cascade `path_ambiguous`.
  - Txn không đổi gì vẫn tăng version.
  - Preview gọi `getOrCreate`, tức là có ghi DB.
- **T09**
  - Hiệu năng O(n²) theo lịch sử change.
  - Phần tử đã xoá làm mất ngữ cảnh ánh xạ section.
  - `stepsOf` vẫn dùng bảng tạm dù step registry T12 đã merge.
- **T10**
  - GridFS `delete` chạy song song gây 500.
  - File SVG/PNG cũ còn khi hình chuyển sang `error` hoặc id bị gỡ.
  - `saltCell` không lọc `^()`.
- **T11**
  - Retry không gửi lại op của lượt trước.
  - Lượt lỗi parse không vào `usage[]`.
  - `STEP_SPECS` lệch step registry (`documents`, `writes`).
- **T12**
  - Mock `produce` ném lỗi thì stream không đóng.
  - FE nhận `S-5.1@` (loop rỗng) còn BE từ chối.

## Thiếu test quan trọng

- **T01:** repository trên Mongo thật; `deleteProject` dọn Spine.
- **T02:** test cho `seed-fixture.ts`.
- **T03:** route 410; startup nạp skill.
- **T04:**
  - `deductCredit` khi ví không đủ.
  - Nhánh reservation hết hạn khi `reserved > 0`.
  - Chưa có test FE nào cho billing và notification.
- **T05:** body preview lớn; watermark hợp lệ khi mở bằng Word/LibreOffice.
- **T06:** pipeline Mongo thật; trang FE admin.
- **T08:**
  - Race gây lỗ seq.
  - Revert khi phía sau đã có sửa đổi.
  - Undo xoá màn trong S-5.
  - `onlyNew` trên Spine legacy.
  - Chặn sửa `flags`/`steps` qua `/changes`.
- **T09:** recompute thường sau atBaseline; `remediation_step` trên fixture biến thể.
- **T10:** nhãn chứa "syntax error"; render lặp lại sau auto-fix; render đồng thời.
- **T11:** `ops: []`; txn trùng do model trả.
- **T12:** stream đóng sớm; unmount giữa lúc stream; `useWorkspace`.

## Việc cần chốt với nhóm

1. ~~**T04:** giữ `payment_service` thật hay quay lại mock + HMAC.~~ **Đã chốt: giữ `payment_service` thật.**
2. ~~**T06/T03:** có khôi phục route prompt-template read-only không.~~ **Đã chốt: bỏ tính năng prompt-template.** H2 không còn là lỗi.
3. **T05:** giữ `projectId` bắt buộc trong `RenderedDocument` và custom property của docx — `context/business-flow.md` I-1 (Preflight) kiểm "dấu version FlintFlow trong file" khi import Word, cần id project để đối chiếu. FE `types/document.ts` đã thêm. Cần báo C (T15) và D (T16).
4. **T08:** 4 người approve `pipeline-contract.md` để đóng băng tại M2. Trước khi approve, contract đã bổ sung (theo `context/srs-spine.md` §3, §6 và Phases §4.1): whitelist path user ghi được, `revert_conflict`, lô không đổi (`txn: null`), cách ghi Spine + changes, `force` cho render.

## Kết quả sửa (2026-09-14)

Nhánh `wave2/review`, commit theo task. Test mới đi kèm từng mục.

| Lỗi | Kết quả | Commit |
|---|---|---|
| H1 T02 seed ghi `project_id` | Ghi qua model `Spine` (khoá `projectId`), `spineSchema.parse`, xoá `changes` cũ, `is_pipeline` qua model | BE `1f27ec4` |
| H2 T03 route prompt-template 410 | **Không sửa** — nhóm bỏ tính năng prompt-template | — |
| H3 T11 `txn` do model trả | Server luôn sinh `randomUUID`; test hai lượt khác txn | BE `2fefd2b` |
| T01 `deleteProject` bỏ sót | Dọn `Spine/Change/Baseline/Usage`; thêm unique `(projectId, version)` cho baseline | BE `c1ec448` |
| T03 startup không nạp skill | `validateStartupAssets` gọi `getSkillIndex()` + `loadStepRegistry()`; controller từ chối `DIAGRAM_CLASSIFY/GENERATE` | BE `70e807e` |
| T04 `deductCredit` không atomic | Claim `reserved→deducted` (hoặc `expired→deducted`); ví lệch ⇒ hoàn state để release/cron xử lý; nhánh expired lọc theo `balance - reserved ≥ cost`; `balanceAfter` = khả dụng | BE `85b8918` |
| T04 lên Pro miễn phí | `upgradePlan` gói trả phí ⇒ 402 `PAYMENT_REQUIRED`; mua qua `POST /billing/checkout {packageId:"plan:pro"}`, tiền về mới ghi Subscription + credit kỳ đầu; FE nút "Mua gói" đi checkout | BE `85b8918`, FE `befce78` |
| T05 `express.json` 100KB | Parser 15mb riêng cho `/api/v1/export` | BE `2efb86d` |
| T06 role từ JWT | Middleware đọc `role` + `isActive` từ DB mỗi request | BE `f3680a8` |
| T08 lỗ seq khi race | Có replica set: Spine + changes trong một Mongo transaction. Standalone: `saveWithVersion` trước, trùng seq thì cấp lại dải — bỏ bù trừ `deleteMany({txn})` | BE `f7ccb3d` |
| T08 revert chèn trùng id / đè sửa đổi sau | Mỗi change chỉ revert khi giá trị hiện tại đúng là `change.value`, khác ⇒ `revert_conflict` | BE `f7ccb3d` |
| T08 undo xoá màn trong S-5 | Màn do revert khôi phục không bị bất biến 8 coi là màn mới | BE `f7ccb3d` |
| T08 `onlyNew` theo message | Khoá `rule + path` | BE `f7ccb3d` |
| T08 `/changes` sửa `flags/steps/…` | Chặn gốc `flags, steps, progress, baselines, sections, diagrams` ⇒ `path_not_writable`; kiểm quyền trước validate body | BE `f7ccb3d` |
| T09 recompute thường đóng cờ S-9 | Không đóng cờ thuộc luật `at_baseline` khi không chạy `atBaseline` | BE `e9583a5` |
| T10 render lại luôn tăng version | So theo `source_hash`; `force` để compile lại chủ động | BE `7a2c1e5` |
| T10 GridFS trước transaction | Lưu file sau `applyTransaction`; lô 409 không để lại file | BE `7a2c1e5` |
| T10 marker "syntax error" | Chỉ giữ `Syntax Error?` (câu PlantUML) + `[From string (line` + HTTP ≥ 400 | BE `7a2c1e5` |
| T10 `dropErrorLine` | Bỏ; mặc định không tự sửa (`noAutoFix`), T13 cắm skill `render_fix` | BE `7a2c1e5` |
| T11 `ops: []` coi là thành công | `draftOps` trả `txn: null` kèm `notes`, không tạo Transaction rỗng | BE `2fefd2b` |
| T12 luồng đóng sớm / không huỷ | `AbortController` (huỷ khi unmount, chạy lại, reset); luồng đóng không có `gate_ready`/`error` ⇒ `STREAM_CLOSED`; bỏ sự kiện của step khác | FE `a425983` |
| T12 reload Spine lệch thứ tự | `useSpine` chỉ áp response của lần gọi mới nhất và không lùi version; `versionRef` chỉ tăng | FE `a425983` |

Lỗi thấp đã sửa cùng lượt:
- T01 unique baseline;
- T02 test fixture cho phép `renumber`;
- T03 action diagram archive trả 400;
- T04 `balanceAfter`;
- T05 `---` sau dòng có `|`, đoạn "2026.";
- T08: selector vô hướng trùng lấy bản đầu, txn không đổi không tăng version, preview không `getOrCreate`;
- T09 spread O(n²) trong `feedingChanges`;
- T10: GridFS `delete` bỏ qua file đã mất, xoá file khi hình lỗi/bị gỡ, `saltCell` lọc `^()`;
- T11: retry gửi lại ops cũ (`{{previous_ops}}`), `STEP_SPECS` thay bằng step registry;
- T12: mock đóng luồng khi lỗi, FE từ chối `S-5.1@`.

Còn mở (thấp, chưa sửa):
- T01 tạo Project + Spine ngoài transaction;
- T02 case-05 dùng `set` thay `renumber`;
- T03 code chết Excalidraw trong `response-parser.ts`;
- T04 order timeout đã tạo phía `payment_service`, intent `failed` nhận tiền muộn;
- T06 điều kiện `state` chết trong ai-cost, `encodeURIComponent` id;
- T07 `.env.local`, `packages/excalidraw/`, `fetch` thô;
- T08 phần tử vô hướng chứa `[`/`]`, mảng khoá id lặp;
- T09 phần tử đã xoá mất ngữ cảnh section, `stepsOf` còn bảng tạm;
- T11 lượt lỗi parse không vào `usage[]`.

Fixture T02 `case-06` ghi `step_id: "S-5.1"` (id template); step registry đòi `S-5.1@<màn>` — test draft-to-ops tự thêm `@screen_cursor`, nên sửa fixture khi làm T13.
