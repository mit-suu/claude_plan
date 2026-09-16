# BÁO CÁO TASK T20 — Discovery B-0 → B-2 + S-1 qua step runner (BE + FE) · Wave 4 · Người: D · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Đang làm** — code xong cả BE và FE, DoD xanh trên mock; còn lượt chạy thật trên dev (bước 5)
- Nhánh: `feat/FLF-160-discovery-brief` ở **cả hai repo** · Commit cuối: BE `25422fd`, FE `5a60dd2` · PR: chưa mở
- % ước lượng hoàn thành: 90% · Effort đã dùng / ước lượng: 8 / 9 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | Skill `product-brief` (tái dùng nội dung `chat_discovery`) + `brief-analysis` | Xong — `product-brief` 124 dòng + 4 `references/` + `assets/brief-template.md`; `brief-analysis` mới cho S-1.1/1.3/1.4 |
| 2 | Runner: Fast/Coaching cho B-*, B-2.1 duyệt lô, approve B-2.3 mở S-1 | Xong — trần 2 lượt elicit/phase đã có sẵn ở `step-runner` (T13); duyệt lô ở panel FE; accept B-2.3 ⇒ `nextStep` = S-1.1 |
| 3 | Gỡ `CHAT_DISCOVERY` khỏi chat-session; gỡ `advance-to-generation` | Xong — bỏ cả `buildCompletedStepsSummary`; thêm đường tin nhắn → `submitAnswer` |
| 4 | FE refit (xoá parse transcript), 3 panel mới | Xong — ChatBubble dọn sạch; `BriefSummaryCard`, `AssumptionSweepPanel`, `AddendumTriagePanel` |
| 5 | E2E mock + `E2E_AI=1`; project mới trên dev đi trọn B-0 → S-1.4 rồi S-2.1 | **Một phần** — e2e mock xanh (6 ca); lượt chạy thật trên dev **chưa làm** |

`DiscoveryStepBar` → `StepProgressBar`, `SummaryReviewCard` → `BriefSummaryCard`, `StepTransitionBanner` → `GateCard`: ba component cũ **đã không còn được import** từ T12/T16; T20 chỉ cần thêm `BriefSummaryCard`. Ba file cũ để T21 xoá.

## 3. File đã thay đổi
**Backend** (31 file · +1266 / −80)
| File | Loại | Trong vùng sở hữu? |
|---|---|---|
| `assets/skills/content/product-brief/SKILL.md` + `references/*.md` (4) + `assets/brief-template.md` | sửa/tạo | S* (`product-brief`) |
| `assets/skills/content/brief-analysis/SKILL.md` | tạo | S* (`brief-analysis`) |
| `src/modules/project/chat-session.service.ts` + `.test.ts` | sửa | S* (task ghi rõ) |
| `src/modules/specification/specification.route.ts` | sửa | S* (task ghi rõ "Gỡ khỏi route: advance-to-generation") |
| `fixtures/op-cases/b0-s1/*.json` (17) | tạo | S* (`op-cases/b0-s1`) |
| `src/modules/pipeline/skills/brief.e2e.test.ts` | tạo | S |
| `docs/spec-gaps.md` | sửa (+5) | thêm dòng |
| `src/modules/pipeline/context-projection.ts` | sửa (+4/−1) | **R** ⚠ mục 4 |
| `src/shared/ai/prompt-assets.test.ts` | sửa (+6/−3) | **R** ⚠ mục 4 |

**Frontend** (7 file · +539 / −122)
| File | Loại | Trong vùng sở hữu? |
|---|---|---|
| `_components/{AssumptionSweepPanel,AddendumTriagePanel,BriefSummaryCard}.tsx` | tạo | S (hàng ba component này, cột T20 = `S`) |
| `_components/__tests__/BriefPanels.test.tsx` | tạo | S |
| `_components/ChatBubble.tsx` | sửa | S (hàng `ChatPane/ChatBubble…`, cột T20 = `S`) |
| `app/projects/[projectId]/page.tsx` | sửa (+18) | S* |
| `lib/constants/section-types.ts` | sửa (+6, chỉ ghi chú) | S* (`DISCOVERY_STEPS`) |

## 4. Thay đổi ngoài vùng sở hữu
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| `src/modules/pipeline/context-projection.ts` (2 dòng `STEP_SKILLS`) | S-1.1/S-1.3/S-1.4 đang trỏ `project-classifier` — skill phân loại dự án, không phải skill đọc lại Brief. Không đổi ánh xạ thì `brief-analysis` không bao giờ tới model, tức là nửa task vô nghĩa. | `[XREQ][T20→T11]` | **granted 2026-09-16** |
| `src/shared/ai/prompt-assets.test.ts` (số skill 30→31, thêm 2 dir vào `WRITTEN_NON_ACTION`) | Skill `product-brief` bỏ `stub` và `brief-analysis` là skill mới. | `[XREQ][T20→T03]` — theo tiền lệ T10/T14/T18/T19 | **granted 2026-09-16** |

## 5. Hợp đồng / interface bị ảnh hưởng
- **Không sửa** `step-registry.json`, `pipeline.dto.ts`, `pipeline-contract.md`, `spine.schema.ts`, `op.types.ts`. 13 step B-* + 4 step S-1 đã có sẵn trong registry từ T12.
- `product-brief` frontmatter đổi `output_schema: discoveryStep` → `opTransaction` cho khớp cách skill content thực sự được dùng (ghi spec-gaps).
- Interface mới: `tryAnswerRunningStep` (nội bộ chat-session); FE xuất `buildDecisionOps`, `needsSingleReview`, `buildRetargetOp`, `buildDropOp`, `groupByTarget`, `PARKED_SECTION`, `TARGET_OPTIONS`.

## 6. Kiểm chứng (dán output thật)
- BE `npx tsc --noEmit`: **pass**. `npx vitest run`:
  ```
  Test Files  62 passed | 1 skipped (63)
       Tests  667 passed | 14 skipped (681)
  ```
  (develop trước đó: 661; +6 ca của `brief.e2e.test.ts`.)
- FE `npx tsc --noEmit`: **pass**. `npm run lint`: `✖ 10 problems (0 errors, 10 warnings)` — 10 warning có sẵn trên develop, không phải của T20. `npm test`:
  ```
  Test Files  28 passed (28)
       Tests  192 passed (192)
  ```
  (develop trước đó: 184; +8 ca `BriefPanels.test.tsx`.)
- DoD grep: `grep -rn "evaluation" flintflow_fe/app/projects` → **rỗng**.
- Test chạy thủ công trên dev: **chưa** — xem mục 7.
- DoD trong file task: **4/4 tick trên mock**; ô 1 ghi rõ lượt chạy thật chưa làm.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| Bước 5: chưa tạo project mới trên dev đi trọn B-0.1 → S-1.4 → S-2.1 với provider thật | Người quyết chi tiêu | Lượt này tốn credit (17 step × 2 lượt gọi) và tạo dữ liệu mới trên dev. Chờ quyết định, không tự chạy. Đây cũng chính là nửa đầu của lượt M4. | **Cao** — chặn M4 |
| ~~XREQ T20→T11 và T20→T03~~ | — | **granted 2026-09-16**; thay đổi đã nằm sẵn trong nhánh | — |
| Task nói "để dành" ở B-2.2 = chuyển sang `other_requirements[]`, nhưng registry chỉ cho B-2.2 ghi `addendum`/`assumptions` | Cả 4 | "Để dành" = đổi `target_section` sang `fixed:5.4`; S-7.4 (T18) đã đọc addendum nhắm `fixed:5.4`. Không mất thông tin, không phải đổi registry. Sửa câu trong task-20 cho khớp. | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí | Vấn đề | Thuộc task nào | Đã ghi spec-gaps? |
|---|---|---|---|
| `assets/prompts/chat_discovery.md` | Không còn đường gọi nào sau khi bỏ `ActionType.CHAT_DISCOVERY` khỏi chat-session | T21 | Có |
| `shared/ai/ai-action.types.ts` `CHAT_DISCOVERY` | Vẫn còn trong enum (đã đánh dấu `@deprecated`); không xoá vì `ai-action.types.ts` ngoài vùng T20 | T21 | Có |
| `_components/{DiscoveryStepBar,SummaryReviewCard,StepTransitionBanner}.tsx` | Không còn được import từ T12/T16; file vẫn nằm đó | T21 (xoá legacy FE) | Không (đã có trong plan T21) |

## 9. Bước tiếp theo
- Việc còn lại của task này: chạy thật trên dev một project mới B-0.1 → S-1.4 rồi S-2.1, ghi số vào `docs/measurements.md` → tick nốt DoD 1 và chuyển trạng thái "Xong". Hai XREQ đã granted.
- Ảnh hưởng tới merge point M4: **Có** — M4 đòi một project mới đi trọn B-0.1 → S-9.5 với provider thật; lượt của T20 là đoạn đầu (B-0 → S-1), nối tiếp với T18 (S-4 → S-8.1) và T19 (S-9).
- Đề xuất: merge T20 cùng đợt với T18/T19 rồi chạy **một** lượt thật xuyên suốt cho cả ba task thay vì ba lượt rời — rẻ hơn và đúng với cái M4 cần.
