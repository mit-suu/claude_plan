# BÁO CÁO TASK T21 — Data migration + xoá legacy + cập nhật docs · Wave 5 · Người: A · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Đang làm** — code, xoá legacy, docs xong; migration **dry-run** xong trên Mongo dev; **chưa chạy thật**
- Nhánh: `feat/FLF-161-migration-cleanup` ở **cả hai repo** · Commit cuối: BE `13293ed`, FE `8905c13` · PR: chưa mở
- % ước lượng hoàn thành: 90% · Effort đã dùng / ước lượng: 6 / 7 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | Migration script + dry-run trên dữ liệu dev | **Một phần** — script + 15 test; dry-run trên Mongo dev: 3 project, 16 section, 0 lỗi. Chạy thật chờ xác nhận (mục 7) |
| 2 | Xoá legacy, typecheck, sửa import vỡ | Xong — BE xoá 62 file, FE xoá 8 file; grep DoD rỗng |
| 3 | README / swagger / CLAUDE.md / architecture.md | Xong |
| 4 | Full test BE/FE | Xong — xem mục 6 |

**Migration** (`src/scripts/migrate-sections-to-spine.ts`, `npm run migrate:sections`):
- Đọc collection `sections` bằng driver thô (model cũ đã xoá). Không sửa/xoá dữ liệu cũ.
- Ghi Spine **chỉ** qua `applyTransaction` với một op hệ thống `migrate` (path `$`) ⇒ một dòng `changes` `{op: "migrate", reason: "legacy import", by: "system"}`, revert được.
- Chỉ migrate Spine chưa từng ghi (`spine_version = 1`) ⇒ chạy lại an toàn. `progress` giữ B-0.1, `steps[]` rỗng.
- Mọi section có nội dung ⇒ addendum trỏ section FPT; thêm `project.vision`, `goals[]` (mã `BG-xx` trong bảng/heading, không có thì bullet), bảng use case ⇒ `actors[]` + `use_cases[]`.
- `--dry-run` áp thử trong bộ nhớ bằng `planTransaction` (bắt lỗi schema/bất biến trước lần thật). Mỗi lần chạy thêm một khối vào `docs/migration-report.md`.
- Parser được chỉnh theo dữ liệu dev thật: bảng thiếu dòng `|---|`, ô actor kiểu "- (include từ UC-03)", mục tiêu dạng bảng `| Mã | Mục tiêu |` và heading `### BG-01 — …`.

**Xoá legacy BE**: `modules/specification/`, `modules/verification/`, `shared/constants/section-types.ts`, phần section của `document-context.service.ts`, `rollbackMessages` + handler + route `POST /chats/:chatId/rollback`, field `workspacePhase` của tin nhắn chat, 6 `ActionType` deprecated + schema/giá/nhánh parse của chúng, 4 prompt cũ, `prompts/_archive/drawtest`, `assets/_archive/diagram-skill`, `src/_archive/drawtest`, 8 script tạm (`test-task-a..d`, `seed-test-uc34`, `verify-task2c-2d`, `audit-callers`, `seed-from-md`), model `PromptTemplate`, script `seed:md`. `audit-ai-db.ts` giữ, chỉ còn soát `PricingConfig`.

**Sửa BE**: `Project` còn `name/domain/status`; gỡ `listProjects/renameProject/archiveProject` (service + controller); xoá cứng project dọn thêm `RenderedDocumentCache`, file sơ đồ GridFS, asset Cloudinary của tài liệu; `deleteProjectDocument` xoá asset Cloudinary (best-effort, `project-document.storage.ts`); gỡ mount `/specifications`, `/verification`; swagger thêm mô tả + tag theo module mới, sửa path `PATCH /projects/{id}/name`.

**Xoá legacy FE**: `lib/constants/section-types.ts`, `lib/api.ts` (import `@/lib/api` giờ trỏ `lib/api/index.ts`, không phải sửa call site), `DraftReviewCard`, `GeneratingIndicator`, `ConfirmRollbackModal`, `DiscoveryStepBar`, `SummaryReviewCard`, `StepTransitionBanner`, `rollbackChat` + `ChatRollbackResult`, `SectionItem`, `ChatActionType "chat_discovery"`. `QuestionStepperInput` **giữ** (ElicitPanel, ChatPane còn dùng) — kiểu `DiscoveryQuestion` chuyển sang `types/chat.ts`. Badge baseline ở `WorkspaceHeader` đọc `spine.baselines` thay `project.baselineVersion`.

## 3. File đã thay đổi
**Backend** — 97 file · +1319 / −8551 (`git diff --stat develop...HEAD`): 62 xoá, 28 sửa, 7 tạo
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `src/modules/{specification,verification}/**` | xoá | S (xoá) | |
| `src/shared/constants/section-types.ts` | xoá | — (task ghi rõ xoá) | xem mục 4 |
| `assets/prompts/{4 file}.md`, `assets/prompts/_archive/drawtest`, `assets/_archive/diagram-skill`, `src/_archive/drawtest` | xoá | S (xoá) / task ghi rõ | |
| `src/scripts/{test-task-a..d,seed-test-uc34,verify-task2c-2d,audit-callers,seed-from-md}.ts`, `audit-ai-db.ts` | xoá / sửa | S | |
| `src/modules/admin/prompt-template.model.ts` | xoá | S | |
| `src/scripts/migrate-sections-to-spine.ts` + `.test.ts` | tạo | S | |
| `src/modules/project/{project.model,project.service,project.controller,project.route}.ts` | sửa | S | |
| `src/modules/project/chat-session.{service,controller,model}.ts` | sửa | S | |
| `src/modules/project/project-document.service.ts`, `project-document.storage.ts` + test, `project.service.test.ts` | sửa / tạo | S (`project-document.*`, `project.*`) | |
| `src/shared/ai/{ai-action.types,response-parser,credit-reservation.service,ai-action.service,ai-action.controller,document-context.service,prompt-assets,prompt-registry.service}.ts` + 2 test | sửa | S | |
| `src/shared/ai/README.md`, `assets/prompts/README.md` | sửa | S | |
| `src/modules/spine/traceability.service.ts` | sửa (1 dòng comment) | S | |
| `src/app.ts`, `src/config/swagger.ts` | sửa | S / S* (swagger) | |
| `package.json` | sửa (bỏ `seed:md*`, thêm `migrate:sections`) | S* | |
| `README.md`, `CLAUDE.md`, `docs/architecture.md`, `docs/migration-report.md` | sửa / tạo | S (task ghi rõ) | |
| `docs/spec-gaps.md` | sửa (+7 dòng) | thêm dòng | |

**Frontend** — 21 file · +148 / −1257
| File | Loại | Trong vùng sở hữu? |
|---|---|---|
| `lib/constants/section-types.ts`, `lib/api.ts`, 6 component legacy | xoá | S (xoá) |
| `lib/api/chat.ts`, `lib/api/endpoints.test.ts` | sửa | S |
| `types/{chat,document,project}.ts` | sửa | S |
| `_components/{ChatPane,ChatInput.test,QuestionStepperInput,WorkspaceHeader}.tsx` | sửa | S |
| `_components/__tests__/WorkspaceHeader.test.tsx` | tạo | S |
| `app/projects/[projectId]/page.tsx` | sửa (truyền `baselineVersion`) | S |
| `README.md`, `CLAUDE.md` | sửa | S (task ghi rõ) |

## 4. Thay đổi ngoài vùng sở hữu
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| `src/shared/constants/section-types.ts` (xoá) | Hàng `shared/constants` không có trong bảng mục 2, nhưng file task liệt kê đích danh "Xoá BE". | Không cần (task ghi rõ) | — |

Không sửa `components/ProjectCard.tsx`, `mocks/state.ts` (vùng T23) — xem mục 7.

## 5. Hợp đồng / interface bị ảnh hưởng
- Không sửa `spine.schema.ts`, `op.types.ts`, `pipeline.dto.ts`, `pipeline-contract.md`, `step-registry.json`, `rendered-document.types.ts`. Op `migrate` đã có sẵn trong `op.types.ts` từ T08.
- Gỡ 6 giá trị `ActionType` deprecated (task ghi rõ; ghi chú spec-gaps về câu "đổi phải qua contract-change"). Route `POST /chats/:chatId/rollback`, `/api/v1/specifications/*`, `/api/v1/verification/*` không còn (không có trong contract).
- `Project` API không còn `currentStep/currentPhase/workspacePhase/baselineVersion/progressPercent`.
- Interface mới: `destroyDocumentAsset(publicId)` (`project-document.storage.ts`); `WorkspaceHeader` prop `baselineVersion`; `DiscoveryQuestion` ở `types/chat.ts`.

## 6. Kiểm chứng
- Lưu ý môi trường: máy này thiếu devDependencies (không có `vitest`, typecheck develop gốc báo 206 lỗi) ⇒ đã chạy `npm ci` ở cả hai repo theo lockfile, không thêm package.
- BE `npx tsc --noEmit -p tsconfig.json`: **pass** (0 lỗi). `npx vitest run`:
  ```
  Test Files  68 passed | 1 skipped (69)
       Tests  716 passed | 14 skipped (730)
  ```
  (develop sau T20: 691 pass; +15 migration, +5 `project.service`, +4 `project-document.storage`, +1 `prompt-assets`. Module đã xoá không có test nào.)
- FE `npx tsc --noEmit`: **pass** với mã nguồn; còn 6 lỗi trong `.next/types/validator.ts` và `.next/dev/types/validator.ts` trỏ tới page đã xoá từ T06/T07 (`admin/prompt-templates`, `draw-test`, `logo-preview`) — file build cache không track, hết khi `next build` lại. `npm run lint`: `✖ 10 problems (0 errors, 10 warnings)` — đúng 10 warning có sẵn như T20. `npx vitest run`:
  ```
  Test Files  29 passed (29)
       Tests  193 passed (193)
  ```
  (develop: 192; −1 ca `rollbackChat`, +2 `WorkspaceHeader`.)
- DoD grep `SectionType|workspacePhase|SectionVersion|RequirementSourceLink|VerificationContext` trên `flintflow_be/src`, `flintflow_fe/app`, `flintflow_fe/lib`: **0 dòng**.
- Migration dry-run trên Mongo dev (`npm run migrate:sections -- --dry-run`):
  ```
  - Project có section cũ: 3 · section đọc được: 16
  - Sẽ migrate: 3 · bỏ qua: 0 · lỗi: 0
  - Addendum: 16 · actor: 2 · use case: 8 · project có vision: 2
  | 6a74b215… | FlintFlow E-Commerce            | would_migrate | 3 | không | 0 | 0 | 0 | 3 |
  | 6aa13da3… | Bán đồ điện tử Online           | would_migrate | 7 | có    | 5 | 2 | 8 | 7 |
  | 6aa160d3… | Bán hàng mô hình đồ chơi trẻ em | would_migrate | 6 | có    | 6 | 0 | 0 | 6 |
  ```
  Cả ba project chưa có Spine (tạo trước T01).
- DoD trong file task: **3/4**; ô 2 thiếu lượt chạy thật + mở project ở B-0.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| Chạy migration **thật** trên Mongo dev (tạo Spine + 1 change cho 3 project) | Người quản DB dev | Chạy `npm run migrate:sections` rồi mở 1 project ở B-0 để kiểm addendum. Revert được (op `migrate`), không xoá `sections`. Chưa tự chạy vì ghi vào DB dùng chung. | **Cao** — chặn DoD 2 |
| `components/ProjectCard.tsx`, `mocks/state.ts` còn đọc `currentStep/progressPercent` mà BE đã gỡ | T23 | XREQ T21→T23: card lấy tiến độ từ `GET /projects/:id/progress` (hoặc bỏ badge), mock bỏ hai field; rồi xoá hai field `@deprecated` còn giữ tạm trong `types/project.ts`. Hiện card rơi về nhãn "Bản nháp" + dòng bước rỗng, không vỡ. | Trung bình |
| Document `projects` cũ vẫn còn giá trị 5 field đã gỡ; collection `sections`/`sectionversions`/`verificationcontexts`/`requirementsourcelinks`/`prompttemplates` vẫn còn | Cả nhóm | Giữ tới khi migration thật được đối chiếu; sau đó dọn thủ công một lần (`$unset` + `drop`). | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí | Vấn đề | Thuộc task nào | Đã ghi spec-gaps? |
|---|---|---|---|
| `flintflow_fe/mocks/handlers.ts:476` | Mock còn `GET /verification/projects/:projectId` — BE đã gỡ | T23 (dọn mock) | Không (mock) |
| `flintflow_be/src/modules/project/chat-session.model.ts` | `messages[].step` mặc định `"vision_problem"` (tên section cũ); `discoveryStep` chỉ còn là nhãn | T23 / sau M5 | Không |
| `flintflow_fe/.next/types/validator.ts` | Build cache trỏ page đã xoá ⇒ `tsc` báo lỗi tới khi build lại | T24 (CI/Docker nên build sạch) | Không |
| Máy dev hiện tại | `node_modules` thiếu devDependencies ở cả hai repo | — | Không |

## 9. Bước tiếp theo
- Việc còn lại: chạy migration thật khi được xác nhận, mở 1 project cũ ở B-0.1 kiểm addendum/actor/use case, ghi khối "thật" vào `docs/migration-report.md`, tick DoD 2; mở PR BE + FE `[T21][Wave 5] Data migration + xoá legacy + docs`.
- Ảnh hưởng tới M5: T21 phải merge trước T22/T23/T24; code đã sẵn trên nhánh, không chặn T22/T23 bắt đầu.
- Đề xuất: sau T22 chạy lượt provider thật của M4 như plan; T23 nhận XREQ ProjectCard.
