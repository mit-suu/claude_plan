# BÁO CÁO TASK T19 — S-9 gate cuối + baseline snapshot + export bản sạch + prioritization · Wave 4 · Người: C · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Xong** (chờ một XREQ để FE dùng được)
- Nhánh: `feat/FLF-159-s9-baseline` (BE `a26466c`) + `feat/FLF-159-s9-baseline` (FE `cdc547a`, XREQ) · PR: chưa mở
- % ước lượng hoàn thành: 100% code · Effort đã dùng / ước lượng: 8 / 8 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | S-9.1 + S-9.5 (tất định) + routes + test | Xong — `completeness-sweep.ts`, `baseline.service.ts`, endpoint 19/20, 10 + 5 test |
| 2 | Snapshot + export bản sạch + test bất biến "baseline không đổi" | Xong — snapshot sâu vào collection `baselines`; e2e dựng cả hai bản và so nội dung |
| 3 | S-9.4 prioritization skill + test; gỡ route UC34/35 | Xong — `content/prioritization` + `prioritization.ts`, 9 test; gỡ `generate-priority`, `generate-scope` |
| 4 | S-9.3 goal validation (cờ vàng) | Xong — `goal-validation.ts` dùng skill `review-section`, cờ `goal_not_covered` không chặn |
| 5 | Notify + swagger | Xong — `notify` khi ký baseline; swagger đủ cho hai endpoint mới |

## 3. File đã thay đổi (đối chiếu bảng vùng sở hữu trong coding-rules.md mục 2)
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `src/modules/pipeline/s9/{completeness-sweep,goal-validation,prioritization,baseline.service,run-s9-step}.ts` | tạo | S | hàng `pipeline/s9/` cột T19 = `S` |
| `src/modules/pipeline/s9/{baseline.controller,baseline.route}.ts` | tạo | S | cùng hàng |
| `src/modules/pipeline/s9/{baseline.service,prioritization,s9.e2e}.test.ts` | tạo | S | 24 test |
| `src/modules/pipeline/step-runner.service.ts` | sửa (+12) | S* | task ghi rõ "S-9.x là step deterministic… không Meter cho 9.1/9.5" |
| `src/modules/specification/specification.route.ts` | sửa (−84) | S* | task ghi rõ "Gỡ khỏi route: generate-priority, generate-scope, approve-baseline" |
| `src/modules/spine/deterministic-check.ts` | sửa (+11) | S* | thêm `MODEL_OWNED_RULES` — cần cho cờ `goal_not_covered` mà task nêu đích danh |
| `src/modules/spine/flags.service.ts` | sửa (+2/−2) | S* | `planFlagOps` bỏ qua cờ model-owned khi dọn |
| `src/modules/spine/section-registry.ts` | sửa (+7) | S* | ⚠ xem mục 5 và 7 |
| `assets/skills/content/prioritization/SKILL.md` | tạo | S* | `assets/skills/content/` cột T19 = `S* (prioritization)` |
| `assets/skills/output/srs-completeness-score/SKILL.md` | sửa | S* | `assets/skills/output/` cột T19 = `S* (completeness-score)` |
| `src/app.ts` | sửa (+2) | S* | chỉ thêm import + mount |
| `docs/spec-gaps.md` | sửa (+6) | thêm dòng | ai cũng được thêm dòng |
| `src/shared/ai/prompt-assets.test.ts` | sửa (+9/−3) | **R** | ⚠ xem mục 4 |

Tổng: **20 file · +1894 / −89**

## 4. Thay đổi ngoài vùng sở hữu
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| `src/shared/ai/prompt-assets.test.ts` (số skill 30→31, thêm 2 dir vào `WRITTEN_NON_ACTION`) | Skill mới `content/prioritization` và `output/srs-completeness-score` viết thật thì test của T03 khoá số lượng và khoá `stub` sẽ đỏ. | `[XREQ][T19→T03]` — theo đúng tiền lệ đã chốt 2026-09-15 (T10/T14) và dùng lại ở T18. | **granted 2026-09-16** |
| `flintflow_fe/lib/api/export.ts` + `lib/api/endpoints.test.ts` | `createBaseline` gửi body rỗng nên BE trả 400; đổi chữ ký thành `createBaseline(projectId, baseVersion)` gửi `{ base_version }`. | `[XREQ][T19→T16]` | **granted 2026-09-16** — commit riêng `t19(xreq→t16): …` trên nhánh FE `feat/FLF-159-s9-baseline` |

## 5. Hợp đồng / interface bị ảnh hưởng
- **Không sửa** `pipeline.dto.ts`, `pipeline-contract.md`, `op.types.ts`, `spine.schema.ts`, `step-registry.json`. Mọi schema cần dùng (`baselineRequestSchema`, `baselineResponseSchema`, `BASELINE_BLOCKED`) đã có sẵn.
- **Cần 4/4 xác nhận nếu nhóm coi là chạm hợp đồng**: `section-registry.ts` `matchRows` giờ trả rỗng cho `functions[].priority` và `nfrs[].priority`. **Bảng §4 (`FIELD_SECTION_MAP`) không đổi một dòng** — chỉ là ánh xạ của hai field, đúng kiểu đã làm sẵn cho `functions[].order`. Lý do bắt buộc phải sửa ở mục 7.
- Interface mới: `signOff()`, `listBaselines()`, `blockingFlags()`, `nextBaselineVersion()`, `completenessSweep()`, `validateGoals()`, `prioritize()`, `scopeWarnings()`, `runS9Step()`, `S9_FREE_STEPS`, `MODEL_OWNED_RULES`.
- Endpoint: `POST /projects/:id/baseline` (19), `GET /projects/:id/baselines` (20).

## 6. Kiểm chứng (dán output thật)
- `npx tsc --noEmit`: **pass**, không output.
- `npx vitest run`:
  ```
  Test Files  64 passed | 1 skipped (65)
       Tests  685 passed | 14 skipped (699)
  ```
  (develop trước đó: 661.) Không test nào bị skip thêm.
- Test của task: `npx vitest run src/modules/pipeline/s9/` → `3 passed (3) · 24 passed (24)`.
- **Chạy thật trên BE + Mongo** (dev server, tài khoản `fixture@flintflow.io`, project run20 — dự án mới đi tới S-3.6 nên §3.1.x còn trống):
  ```
  GET  /projects/:id/baselines              200 · []          ← đóng spec-gap "/baselines 404" của M3
  POST /projects/:id/baseline {base_version:65}  422 BASELINE_BLOCKED
       meta.flags: FL005 section_empty fixed:3.1.1 (S-4.2), FL006 fixed:3.1.2 (S-4.1), FL008 fixed:3.1.4 …
  POST /projects/:id/baseline {base_version:1}   409 SPINE_VERSION_CONFLICT
  POST /projects/:id/baseline (không body)       400 VALIDATION_ERROR
  GET  /projects/:id/flags?level=red&open=true → 35 cờ đỏ mở
  ```
  Đúng thiết kế: chưa làm §3 thì chưa ký được, và thông báo chỉ thẳng step phải quay lại.
- DoD: **5/5 tick**.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| **S-9.4 tự chặn baseline.** Ghi `priority` cho mọi function/NFR làm 62 section thành `stale` (đo trên fixture 19 màn), rồi `section_stale_at_baseline` chặn đúng cái baseline ngay sau đó. | Cả 4 | Đã sửa: `priority` không ánh xạ section, y như `order`. Cần 4/4 xác nhận vì đụng `section-registry.ts`. Không sửa thì S-9 không bao giờ ký được. | **Cao** |
| ~~FE chưa gửi `base_version`~~ | — | **Đã làm** (XREQ granted): `createBaseline(projectId, baseVersion)`. Lưu ý: **chưa component nào gọi hàm này** — `ExportPanel` mới chỉ liệt kê baseline, nút "Ký baseline" thuộc T16/T23. Tầng API đã đúng để người thêm nút không gọi sai. | — |
| ~~XREQ T19→T03~~ | — | **granted** | — |
| Task viết `409 RED_FLAGS_OPEN`, hợp đồng có sẵn `422 BASELINE_BLOCKED` | Cả 4 | Theo hợp đồng. Sửa câu trong task-19 cho khớp. | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa thêm, chỉ ghi)
| Vị trí | Vấn đề | Thuộc task nào | Đã ghi spec-gaps? |
|---|---|---|---|
| `flags.service.planFlagOps` | Cờ `accepted_as_is` của gate (T13) bị lượt recompute kế tiếp tự đóng — lỗi có từ T13, chưa ai gặp vì chưa ai recompute ngay sau `accept_as_is`. Đã sửa cùng cơ chế `MODEL_OWNED_RULES` vì T19 cần nó cho `goal_not_covered`. | T13 | Có |
| `assets/prompts/priority_ranking.md`, `scope_out_of_scope.md` | Prompt của UC34/UC35 giờ không còn route nào gọi | T21 (xoá `assets/prompts` cũ) | Không (đã nằm trong kế hoạch T21) |

## 9. Bước tiếp theo
- Việc còn lại của task này: **không còn việc code**. Cần 4/4 xác nhận cho `priority` không ánh xạ section (mục 5). Nút "Ký baseline" trên UI vẫn thiếu — thuộc T16/T23, không phải T19.
- Ảnh hưởng tới merge point M4: **Có** — M4 cần một lượt đi trọn tới `S-9.5 → Word baseline`; endpoint đã sẵn sàng, còn thiếu lượt chạy thật và hai dòng FE.
- Đề xuất: merge T19 sau T18 (S-9.1 quét lại glossary của S-8.1) và trước T21.
