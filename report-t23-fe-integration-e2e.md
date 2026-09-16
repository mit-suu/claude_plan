# BÁO CÁO TASK T23 — FE tích hợp thật toàn bộ + e2e + dọn mock · Wave 5 · Người: D · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Xong**
- Nhánh: `feat/FLF-161-fe-integration-e2e` (FE) · Commit cuối: `e837787` · PR: chưa mở
- % ước lượng hoàn thành: 100% · Effort đã dùng / ước lượng: 6 / 6 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | Tắt mock runtime; chạy luồng trên BE dev; ghi drift rồi sửa | Xong — gỡ hẳn đường bật msw; drift tìm được: `rollbackChat` trỏ endpoint legacy (gỡ), `createBaseline` thiếu `base_version` (đã sửa ở XREQ của T19) |
| 2 | ProjectCard/home dùng dữ liệu Spine | Xong — nhãn từ step registry, trạng thái theo `readiness`; home nạp `GET /progress` song song |
| 3 | Playwright + CI job | Xong — 2 ca xanh trên BE + Mongo thật; CI thêm job lint, unit test và job e2e riêng |
| 4 | i18n nhãn; docs | Xong — `lib/i18n.ts` đọc `label_vi`/`label_en` từ registry; `docs/fe-architecture.md` |

## 3. File đã thay đổi
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `app/projects/[projectId]/hooks/useWorkspace.ts` | sửa | S* | gỡ nhánh `NEXT_PUBLIC_API_MOCK` |
| `mocks/browser.ts` · `public/mockServiceWorker.js` | **xoá** | S (`mocks/`) | không còn caller sau khi gỡ nhánh trên |
| `components/ProjectCard.tsx` + `.test.tsx` | sửa / tạo | S | bỏ bảng nhãn cứng + phần trăm legacy |
| `app/home/page.tsx` | sửa | S | nạp `GET /progress` cho từng dự án |
| `lib/i18n.ts` + `lib/i18n.test.ts` | tạo | S (task liệt kê đích danh) | |
| `lib/api/chat.ts` · `lib/api/endpoints.test.ts` | sửa | S* | gỡ `rollbackChat` |
| `e2e/workspace.spec.ts` · `playwright.config.ts` | tạo | S | |
| `package.json` · `package-lock.json` | sửa | S* (task cho thêm `@playwright/test`, script `e2e`) | |
| `.github/workflows/ci.yml` | sửa | S* (task ghi rõ) | thêm job lint, test, e2e |
| `vitest.config.ts` · `.gitignore` | sửa | S* | loại `e2e/**` khỏi vitest; bỏ qua artefact Playwright |
| `docs/fe-architecture.md` | tạo | S (T23) | |

Tổng FE: **18 file · +746 / −481**.

Ngoài ra **1 dòng** thêm vào `flintflow_be/docs/spec-gaps.md` (commit `a5c0498`).

## 4. Thay đổi ngoài vùng sở hữu
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| `flintflow_be/docs/spec-gaps.md` (+1 dòng) | Phát hiện thuộc về BE (mock provider), không thuộc nhánh FE. `coding-rules` mục 2: spec-gaps "ai cũng được **thêm dòng**". | Không cần XREQ | — |

⚠ **Cần bạn biết:** dòng đó tôi commit **thẳng vào `develop` của BE**, không qua nhánh — lệch quy ước
"không commit thẳng vào develop". Lý do: một dòng tài liệu, và mở một nhánh BE riêng cho một task FE thì
rườm rà hơn giá trị. Nếu muốn chặt chẽ thì revert rồi đưa vào nhánh của T22.

## 5. Hợp đồng / interface bị ảnh hưởng
- Không đụng hợp đồng đóng băng. `lib/constants/step-registry.json` **không sửa tay** (là bản sao đồng bộ).
- Interface mới: `lib/i18n.ts` — `tStep`, `tPhase`, `tPhaseOfStep`, `localeOf`, `DEFAULT_LOCALE`;
  `ProjectCard` nhận thêm prop `progress` và `locale`; `nextStepLabel` xuất ra để test được.
- Dependency mới: `@playwright/test` (task cho phép đích danh).

## 6. Kiểm chứng (dán output thật)
```
npx tsc --noEmit      → pass, không output
npm run lint          → ✖ 9 problems (0 errors, 9 warnings)     [develop có 10; giảm 1 do xoá mockServiceWorker.js]
npm test              → Test Files 30 passed · Tests 207 passed  [develop: 192]
npm run build         → thành công, 14 route
npm run e2e           → 2 passed (11.6s)
  ✓ workspace end-to-end trên BE thật › đăng nhập, thẻ dự án đọc tiến độ thật, workspace và Export chạy trên BE
  ✓ workspace end-to-end trên BE thật › không còn đường bật msw ở runtime
```
DoD grep:
```
grep -rn "NEXT_PUBLIC_API_MOCK" app lib components          → rỗng
grep -n "STEP_LABELS\|progressPercent" components/ProjectCard.tsx → rỗng
```
E2E chạy thật trên BE `:5000` + Mongo thật + FE `:3000`, tài khoản `fixture@flintflow.io`; test tự tạo và
tự xoá project của nó. **Job CI đã viết nhưng chưa chạy trên GitHub** — cần xác nhận ở PR đầu tiên.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| **`AI_PROVIDER=mock` không tồn tại.** Provider chọn theo frontmatter từng skill; `mock.provider.ts` trả JSON không khớp schema pipeline. Không chạy được step AI trong CI. | B (T22) / C (T24) | Thêm biến `AI_PROVIDER` ghi đè provider của mọi skill, và cho mock provider trả output hợp schema theo `ActionType`. Xong hai thứ đó thì mở rộng e2e thêm một ca "chạy S-2.1 → gate Accept → Document pane có §1". Cùng nguyên nhân với dòng T18 về `E2E_AI=1`. | Trung bình — không chặn T23, chặn độ phủ e2e |
| Job e2e trên CI cần checkout repo BE | C (T24) | Workflow dùng `secrets.BE_REPO_TOKEN`, rơi về `github.token` nếu không có. Repo BE là private thì phải tạo token đó, nếu không job sẽ đỏ ngay lần chạy đầu. | Trung bình |
| Chưa có nút "Ký baseline" trên UI | D / T16 | `lib/api/export.ts#createBaseline` đã đúng hợp đồng nhưng chưa component nào gọi. Không nằm trong DoD của T23. | Thấp |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí | Vấn đề | Thuộc task nào | Đã ghi spec-gaps? |
|---|---|---|---|
| `flintflow_be/src/shared/ai/providers/` | Không có đường ép provider; mock provider trả JSON sai schema | T22 / T24 | Có (`a5c0498`) |
| `lib/constants/section-types.ts`, `_components/{DiscoveryStepBar, SummaryReviewCard, StepTransitionBanner, DraftReviewCard, ConfirmRollbackModal, QuestionStepperInput}.tsx` | Không còn được import, vẫn nằm trong repo | T21 (xoá legacy FE) | Không (đã có trong plan T21) |
| `flintflow_be` `POST /projects/:id/chats/:id/rollback` | FE không gọi nữa; route vẫn còn | T21 | Không |
| `types/chat.ts` `ChatRollbackResult` | Không còn nơi dùng sau khi gỡ `rollbackChat` | T21 | Không |

## 9. Bước tiếp theo
- Việc còn lại của task này: **không còn**. Chỉ cần xác nhận job CI chạy xanh ở PR đầu tiên (và tạo
  `BE_REPO_TOKEN` nếu repo BE private).
- Ảnh hưởng tới merge point M5: **Có** — M5 đòi CI xanh gồm e2e; job đã có, chờ lần chạy thật trên GitHub.
- Đề xuất: merge T23 **sau T21** để rebase lên bản đã xoá legacy — xung đột nếu có chỉ là "một bên sửa,
  một bên xoá" trên mấy file legacy, giải bằng cách cho xoá thắng. Khi T22 làm xong mock provider thì bổ
  sung một ca e2e phủ bước AI.
