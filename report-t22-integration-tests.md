# BÁO CÁO TASK T22 — Bộ test tích hợp + đo token end-to-end trên fixture · Wave 5 · Người: B · Ngày: 2026-09-16

## 1. Trạng thái
- Trạng thái: **Đang làm** — code, test, CI, tài liệu xong; còn hai việc cần provider thật / GitHub (mục 9)
- Nhánh: `feat/FLF-162-integration-tests` (BE, tách từ `develop` `30b9f19`) · Commit cuối: `9a072ff` (đã push) · PR: chưa mở
- % ước lượng hoàn thành: ~85% · Effort đã dùng / ước lượng: 6 / 6 điểm

## 2. Đã làm (theo bước trong file task)
| Bước | Mô tả ngắn | Kết quả |
|---|---|---|
| 1 | Memory server + helper; chuyển test có DB sang setup chung | **Một phần** — `test/global-setup.ts` (MongoMemoryReplSet 1 node ⇒ có transaction), `test/setup.ts` (`seedFixture(kind)`, `authAs(user)`, `resetDb`, `parseSse`), DB riêng mỗi worker. **Không chuyển** 66 file unit có sẵn: file task viết lúc BE còn 2 file test; giờ 66 file tự mock model và nhiều test dựa vào `readyState !== 1` để đi nhánh `TRANSACTION_UNAVAILABLE` — nối Mongo chung sẽ đổi hành vi test của người khác (coding-rules §3.9). Tách vitest thành 3 project `unit` / `integration` / `e2e-ai`. |
| 2 | 8 file integration qua HTTP, mock provider trả op từ `fixtures/op-cases` | Xong — đủ 8 file tên như task + 2 file phụ (`op-case-eval`, `measure-tokens`). Mock ở **tầng `callLLM`** (không mock `executeAiAction`) ⇒ reserve/trừ/hoàn credit, parse, retry, `AiActionLog`, `Usage` là code thật. 57 test. |
| 3 | E2E-AI op cases; script đo token; chạy 1 lần `--real` | **Một phần** — `test/e2e-ai/op-cases.e2e.test.ts` + bộ chấm `test/helpers/op-case-eval.ts` (kiểm bằng executor giả: 10/10 khi model trả đúng op kỳ vọng). `src/scripts/measure-tokens.ts` chạy chế độ `estimate` trên fixture 19 màn, đã ghi `docs/measurements.md`. **Chưa chạy provider thật** (cả op cases lẫn `--mode real`) — xem mục 7. |
| 4 | Coverage + CI | Xong trên máy — ngưỡng 70% lines cho `spine/**`, `pipeline/**` (đo: **95.91%** / **94.01%**; đã thử nâng lên 99% ⇒ vitest exit 1). CI BE: `typecheck:test`, cache binary mongod, `test:coverage`, upload artifact. CI **GitHub chưa chạy** (chưa push). |

## 3. File đã thay đổi (đối chiếu bảng vùng sở hữu trong coding-rules.md mục 2)
| File | Loại | Trong vùng sở hữu? | Ghi chú |
|---|---|---|---|
| `flintflow_be/vitest.config.ts` | sửa | S (T22) | 3 project, `globalSetup`/`setupFiles` chỉ cho integration + e2e-ai, coverage thresholds |
| `flintflow_be/package.json`, `package-lock.json` | sửa | S (T22) | dev dep: `mongodb-memory-server`, `supertest`, `@vitest/coverage-v8@4.1.11` (khớp vitest 4.1.11), **`@types/supertest`** (xem mục 9 Đề xuất); script `test:unit`, `test:integration`, `test:e2e-ai`, `test:coverage`, `typecheck:test`, `measure:tokens` |
| `flintflow_be/.github/workflows/ci.yml` | sửa | S (T22) | xem bước 4 |
| `flintflow_be/test/{global-setup,setup}.ts`, `test/tsconfig.json` | tạo | S (`test/`) | `test/tsconfig.json`: `tsconfig.json` gốc chỉ bao `src/` và T22 không được sửa `tsconfig*.json` gốc |
| `flintflow_be/test/helpers/{mock-llm,pipeline,op-cases,op-case-eval}.ts` | tạo | S (`test/`) | |
| `flintflow_be/test/integration/*.ts` (10 file) | tạo | S (`test/`) | |
| `flintflow_be/test/e2e-ai/op-cases.e2e.test.ts` | tạo | S (`test/`) | kết quả ghi `test/e2e-ai/results/` |
| `flintflow_be/src/scripts/measure-tokens.ts` + `.test.ts` | tạo | S* (measure-tokens) | |
| `flintflow_be/docs/testing.md` | tạo | S (T22) | |
| `flintflow_be/docs/measurements.md` | sửa (+149) | thêm mục | mục `## Threshold` (trống, nhóm điền) + khối `<!-- T22:measure-tokens -->`; không đụng mục T14/T18 |
| `flintflow_be/docs/spec-gaps.md` | sửa (+4) | thêm dòng | |
| `claude_plan/{task-22-integration-tests,plan-overview}.md`, `report-t22-integration-tests.md` | sửa/tạo | dòng của T22 | |

Tổng BE: **27 file** (6 sửa + 21 tạo) · **+3 585 / −9** dòng (trong đó `package-lock.json` +840). ⚠ Vượt khuyến nghị PR ≤ ~600 dòng — đề xuất tách 3 PR theo bước: (1) hạ tầng + `spine-api` + `op-engine` + `flags`; (2) 5 file integration còn lại; (3) e2e-ai + `measure-tokens` + CI + docs.

## 4. Thay đổi ngoài vùng sở hữu
| File | Lý do | Issue XREQ | Ai granted |
|---|---|---|---|
| Không có | | | |

`flintflow_fe/.github/workflows/ci.yml` (task ghi "test BE + FE"): **không sửa** — bảng vùng FE không có cột T22. FE CI hiện chỉ typecheck + build, thiếu `lint` + `test`. Chuyển cho T23 (sở hữu `package.json`/CI phía FE theo T07/T23) — mục 7.

## 5. Hợp đồng / interface bị ảnh hưởng
- Có đụng hợp đồng đóng băng không: **Không**.
- Interface mới cho task khác:
  - `test/setup.ts`: `seedFixture(kind, { balance, mutate, withoutSpine, email })`, `authAs(user)`, `parseSse(text)`, `resetDb()`.
  - `test/helpers/mock-llm.ts`: `mockLlmRouterModule()`, `mockCalls`, `mockOverrides.next`.
  - `test/helpers/pipeline.ts`: `runStepHttp`, `gateHttp`, `startAt(stepId, targets?)`.
  - `npm run measure:tokens -- [--mode estimate|real-draft|real] [--fixture full|minimal] [--no-write] [--usd-in X --usd-out Y]` — T24 có thể chạy trong compose; M4 dùng `--mode real` cho lượt chạy thật đang hoãn.

## 6. Kiểm chứng (output thật)
- `npm run typecheck`: pass (không lỗi). `npm run typecheck:test`: pass.
- `npm test` (3 project): `Test Files 76 passed | 2 skipped (78)` · `Tests 759 passed | 26 skipped (785)`. Mốc trước task trên `develop`: `65 passed | 1 skipped`, `691 passed | 14 skipped`. Skip mới = 12 test của `e2e-ai` (không có `E2E_AI=1`).
- `npm run test:integration`: 10 file, 57 test pass, ~21 s.
- `npm run test:coverage`: `Tests 759 passed | 14 skipped`; tổng `Lines 73.89% (5211/7052)`; `src/modules/spine/**` **95.91%** (1664/1735), `src/modules/pipeline/**` **94.01%** (973/1035). Thử ngưỡng 99%: `ERROR: Coverage for lines (95.9%) does not meet "src/modules/spine/**" threshold (99%)`, exit 1 — đã trả về 70%.
- `npm run build`: pass, `dist/scripts/measure-tokens.js` có.
- `npm run measure:tokens` (estimate, fixture 19 màn): 81 step, 108 lượt gọi (53 elicit + 54 draft + 1 review), **tokens in 381 766**, tokens out (ước lượng) 22 955, **271 credit**; ngoại suy 20 vòng S-5: 164 lượt, 599 256 in, **411 credit**; input 1/3 đầu 3 480 → 1/3 cuối 4 393 = **1.26×** (không thấy bậc hai). USD n/a, ngưỡng chưa đặt.
- Snapshot bị vitest ghi lại do CRLF: đã `git checkout` 3 file `__snapshots__`.
- DoD trong file task: **0/3 tick đầy đủ**:
  - "CI xanh; coverage lõi ≥ 70%": đạt trên máy; CI GitHub chưa chạy vì chưa push.
  - "`E2E_AI=1` ≥ 8/10": chưa chạy provider thật.
  - "`docs/measurements.md` có tổng token và chi phí fixture 19 màn, so với ngưỡng": có số **estimate** (token + credit); USD n/a (không có đơn giá GLM); ngưỡng nhóm chưa đặt; chưa có số `--real`.

## 7. Bị chặn / cần quyết định
| Vấn đề | Cần ai | Đề xuất của tôi | Mức khẩn |
|---|---|---|---|
| Lượt chạy provider thật (`E2E_AI=1` op cases + `measure:tokens --mode real-draft/real`) tốn credit provider; plan-overview đã quyết hoãn lượt chạy thật tới sau W5 | Nhóm (người giữ key Modal) | Chạy ngay sau khi merge T22: `E2E_AI=1 npm run test:e2e-ai` rồi `npm run measure:tokens -- --mode real-draft`. Cùng lúc gỡ được M4 | Cao (chặn DoD 2, 3 và M4) |
| Ngưỡng chi phí `## Threshold` trong `docs/measurements.md` chưa ai đặt; không có đơn giá USD cho `glm` | Nhóm | Điền `credit_per_project` (số đo estimate: 271, ngoại suy 411) và đơn giá USD/1M nếu có | Trung bình |
| FE CI chưa chạy `lint` + `test` | T23 (D) | Thêm hai step vào `flintflow_fe/.github/workflows/ci.yml` | Trung bình |
| `baseline_id` của endpoint 16/18 ≠ `id` endpoint 19/20 (mục 8) | T15/T19 (C) | Cho `getBaselineDocument` nhận cả `BLnnn` | Trung bình — sẽ lộ khi T23 nối FE |

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)
| Vị trí (file:dòng) | Vấn đề | Thuộc task nào | Đã ghi docs/spec-gaps.md? |
|---|---|---|---|
| `src/modules/render/assemble.service.ts:608-613` (`getBaselineDocument`) | Tra `Baseline._id` Mongo, trong khi `POST /baseline`/`GET /baselines` trả `id: "BL001"` ⇒ `?baseline_id=BL001` luôn 404 | T15 / T19 | Có |
| `fixtures/spine-fixture-19-screens.json` | 14/19 màn `placeholder` ⇒ chỉ 6/20 vòng S-5 chạy; số đo "19 màn" thấp hơn thực tế | T02 | Có |
| `flintflow_be/CLAUDE.md` mục Test | Còn ghi "không có `setupFiles`… chưa có `mongodb-memory-server` (T22 sẽ thêm)" và nhánh `E2E_AI=1` không chạy được — đã lỗi thời sau T22 | T21 (sở hữu README/CLAUDE.md) | Không (không phải gap tài liệu spec) |
| `src/app.ts:30` | `connectDB()` chạy ngay lúc import và `process.exit(1)` khi lỗi — test phải đặt `MONGO_URI` trước import; `morgan("dev")` luôn bật làm log test ồn | T21 / T24 | Không |
| spec-gap T18 "nhánh `E2E_AI=1` không nối Mongo" | Nguyên nhân đã được khung T22 gỡ (credit thật chạy được trong vitest); hai file `skills/*.e2e.test.ts` vẫn chú thích như cũ | T14 / T18 | Có (dòng mới) |

## 9. Bước tiếp theo
- Việc còn lại của task này:
  1. Commit (3–5 commit `t22: …`), push, mở PR (đề xuất tách 3 PR, mục 3), xem CI GitHub xanh.
  2. Chạy `E2E_AI=1 npm run test:e2e-ai` một lần, chép bảng `test/e2e-ai/results/op-cases-*.md` vào `docs/measurements.md`, ghi số thật.
  3. Chạy `npm run measure:tokens -- --mode real-draft` (hoặc `--mode real`) một lần; nếu tỉ lệ input 1/3 cuối ÷ 1/3 đầu ≥ 2× thì mở issue cho projection T11.
- Ảnh hưởng tới merge point **M5**: Có — M5 cần "CI xanh (unit + integration + e2e)"; phần integration BE đã có. **M4**: lượt chạy thật đang hoãn giờ chạy được bằng hai lệnh trên.
- Đề xuất: duyệt dependency **`@types/supertest`** (không có trong danh sách task nhưng cần để `supertest` có kiểu trong `typecheck:test`). Cập nhật `flintflow_be/CLAUDE.md` mục Test ở T21.
