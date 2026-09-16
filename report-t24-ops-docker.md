# BÁO CÁO TASK T24 — Vận hành: docker compose BE/FE, PlantUML, env, health, hardening · Wave 5 · Người: C · Ngày: 2026-09-16

## 1. Trạng thái

**Xong.** DoD 4/4. Nhánh `feat/FLF-163-ops-docker` ở cả `flintflow_be` và `flintflow_fe`.

Task này kéo theo mốc **M4** (lượt chạy provider thật, hoãn từ Wave 4) vì cùng một buổi và vì M4 lộ ra
hai lỗi mà T24 phải sửa. Số liệu M4 ở `flintflow_be/docs/measurements.md`.

## 2. Đã làm (theo bước trong file task)

1. **Compose + replica set + PlantUML ghim tag.** `docker compose up -d --wait` dựng mongo (`--replSet rs0`)
   → `mongo-init` (`rs.initiate` + chờ PRIMARY, idempotent) → plantuml (`jetty-v1.2025.4`) → backend →
   frontend. Mỗi service có healthcheck; `depends_on` theo điều kiện nên `--wait` là đủ để biết hệ thống
   lên hay không. Transaction đa document kiểm bằng `mongosh` qua chính service đó.
2. **Env + gỡ hardcode.** URL endpoint Modal cá nhân bị gỡ khỏi 4 chỗ; `MODAL_BASE_URL` không còn default,
   thiếu là `AI_PROVIDER_NOT_CONFIGURED` ngay lượt gọi đầu (`shared/ai/providers/modal.config.ts`). Thêm
   `AI_PROVIDER_OVERRIDE`, `REVIEW_LLM_ENABLED` vào env đã validate. `.env.example` của cả hai repo viết lại
   đầy đủ kèm lý do.
3. **Health + startup check.** `/health` trả `{status, mongo, plantuml, version, assets}`; mongo hỏng ⇒ 503,
   plantuml hỏng ⇒ `degraded` + 200. Cảnh báo cấu hình lúc khởi động (`warnStartupConfig`) chạy **sau** khi
   nghe cổng và không giết tiến trình.
4. **CI docker + tài liệu.** Workflow `docker` ở cả hai repo; `docs/ops.md` 8 mục.
5. **Chạy e2e T23 trong compose.** 2/2 xanh.

Ngoài ra, làm nốt hai vế mà spec-gap của T23 ghi là còn thiếu: `AI_PROVIDER_OVERRIDE` ghi đè provider mọi
skill, và `mock.provider.ts` trả output **hợp schema** theo `ActionType`. Giờ chạy được một step pipeline
trong CI mà không gọi model thật.

## 3. File đã thay đổi

**flintflow_be** — `docker-compose.yml`, `Dockerfile`, `.dockerignore`, `.gitignore`, `.env.example`,
`scripts/mongo-init-rs.sh` (mới), `src/config/{env,startup-checks}.ts`, `src/config/health.ts` + test (mới),
`src/app.ts`, `src/server.ts`, `src/shared/ai/providers/{modal.config (mới),glm,ai-sdk,llm.router,mock}.ts`,
`mock.provider.test.ts` (mới), `src/shared/ai/{ai-action.service,ai-action.types}.ts`,
`src/modules/pipeline/{context-projection,step-runner.service}.ts` + test,
`src/scripts/{run-full-pipeline,seed-e2e-user}.ts` (mới), `test/integration/health.int.test.ts` (mới),
`.github/workflows/docker-build.yml`, `docs/{ops (mới),measurements,spec-gaps}.md`, `package.json`,
`package-lock.json`.

**flintflow_fe** — `Dockerfile`, `.dockerignore`, `.env.example`, `CLAUDE.md`, `app/layout.tsx`,
`components/GoogleButton.tsx` + test (mới), `lib/google-auth.ts` (mới), `e2e/workspace.spec.ts`,
`docs/fe-architecture.md`, `.github/workflows/{ci,docker-build}.yml`, `package-lock.json`.

## 4. Thay đổi ngoài vùng sở hữu

Ba chỗ, đều nằm trong phần "gỡ hardcode / env / ops" mà chính file task giao cho T24:

- `src/modules/pipeline/context-projection.ts` (vùng T11) — sửa điều kiện thêm danh sách id
  `assumptions`. Không sửa được thì M4 không chạy qua nổi B-2.3.
- `src/shared/ai/{ai-action.service,ai-action.types}.ts` (vùng T03) — thêm `actionType` **tuỳ chọn** vào
  `AiProviderConfig` để mock biết trả schema nào. Không đổi giá trị `ActionType` nào ⇒ không cần
  `contract-change`.
- `src/modules/pipeline/step-runner.service.ts` (vùng T13) — một dòng: `process.env.REVIEW_LLM_ENABLED`
  → `env.REVIEW_LLM_ENABLED`.

## 5. Hợp đồng / interface bị ảnh hưởng

`GET /health` đổi từ `{status: "ok"}` sang `{status, mongo, plantuml, version, assets}` và có thể trả 503.
Endpoint này **không** nằm trong `docs/api/pipeline-contract.md` (nó ngoài `/api/v1`), nên không phải
`contract-change`. Không client nào đang đọc nó ngoài healthcheck của compose.

`assets/step-registry.json` và `pipeline.dto.ts` **không đổi**.

## 6. Kiểm chứng

```
$ docker compose up -d --wait
 Container flintflow-mongo-1 Healthy
 Container flintflow-mongo-init-1 Exited
 Container flintflow-plantuml-1 Healthy
 Container flintflow-backend-1 Healthy
 Container flintflow-frontend-1 Healthy

$ curl -s http://localhost:5050/health
{"data":{"status":"ok","mongo":"ok","plantuml":"ok","version":"1.0.0",
 "assets":{"prompts":2,"skills":32,"steps":56}},"error":null}

$ docker compose exec -T mongo mongosh --quiet --eval '<startSession/commitTransaction>'
transaction ok

$ curl -s -o /dev/null -w "%{http_code}" http://localhost:3050/login
200

$ E2E_BASE_URL=http://localhost:3050 E2E_API_URL=http://localhost:5050/api/v1 npx playwright test
  ✓ workspace end-to-end trên BE thật › đăng nhập, thẻ dự án đọc tiến độ thật... (2.9s)
  ✓ workspace end-to-end trên BE thật › không còn đường bật msw ở runtime (1.4s)
  2 passed (5.3s)

$ AI_PROVIDER_OVERRIDE=mock ... npm run run:pipeline -- --until B-0.2
[1] B-0.1   v accept (2 lượt gọi, 5 credit, 0.4s) -> B-0.2
[2] B-0.2   v accept (2 lượt gọi, 5 credit, 0.3s) -> B-0.3

$ grep -rn "modal.direct" src/ .env.example      → rỗng
$ git ls-files | grep -E '^\.env' | grep -v example   → rỗng (cả hai repo)

$ npm run test:coverage        (BE, đúng lệnh của backend-ci)
 Test Files  82 passed | 1 skipped (83)
      Tests  809 passed | 12 skipped (821)
Lines        : 80.71% ( 5203/6446 )

$ npm test                     (FE)
 Test Files  32 passed (32)
      Tests  211 passed (211)

$ npm run typecheck && npm run typecheck:test && npm run lint    → sạch
```

**Lượt chạy M4** (`test/e2e-ai/results/m4-full-run.md`, bảng đầy đủ ở `docs/measurements.md`):
66/81 step accepted · 72 lượt gọi · 292 572 token in / 46 629 out · 225 credit · 6/6 diagram `ok` ·
assemble 33 section · Word 93 KB · baseline `v1.0-conditional` (59 cờ waive).

## 7. Bị chặn / cần quyết định

- **Kết quả CI trên GitHub chưa xem được** — máy này không có `gh`. Từng bước của cả bốn workflow đã chạy
  ở local, gồm `npm ci` trên Linux (trong container) và `docker compose up` thật. Nhờ người có `gh` xác
  nhận lượt chạy đầu sau khi push.
- **`progress.current_step` không tiến sau gate accept** (spec-gaps). T24 không đổi hợp đồng; cần nhóm chốt
  (a) gate ghi luôn `current_step`, hay (b) `buildProgressReport` trả `nextStep(spine)?.id`. Đề xuất (a) —
  (b) kéo `spine/section-status.ts` phụ thuộc `pipeline/step-registry.ts`.
- **`deploy.yml`** vẫn chỉ deploy BE lên Azure, chưa có đường deploy FE. Ngoài phạm vi task.

## 8. Phát hiện ngoài phạm vi (KHÔNG sửa, chỉ ghi)

- Model dùng **hai kiểu id function trong cùng một project** (`FN01` rồi `FN002`). Không vi phạm bất biến
  nào nhưng nhìn lệch trong tài liệu.
- 55/59 cờ chặn baseline ở lượt M4 là `unconfirmed_assumption` — hệ quả của cách trả lời tự động
  ("thiếu thì tự chọn rồi ghi thành assumption") chứ không phải lỗi sản phẩm. Người dùng thật trả lời
  B-2.1 Assumption Sweep sẽ đóng phần lớn số này.
- Image FE **943 MB** vì cố ý không dùng `output: "standalone"` (Next tuyên bố `next start` không chạy với
  cấu hình đó, mà `npm run start` là lệnh Playwright dùng ở CI). Muốn nhỏ hơn thì phải đổi cả
  `playwright.config.ts#webServer` và lệnh `start` — một quyết định riêng, không gộp vào T24.

## 9. Bước tiếp theo

1. Push hai nhánh, xem lượt chạy CI đầu tiên trên GitHub.
2. Merge vào `develop`, tag `v1.0-refactor`, tick M5.
3. Trước khi lên production: đặt `MODAL_BASE_URL`, `JWT_*` (≥ 32 ký tự), `APP_PUBLIC_URL` domain thật,
   `NEXT_PUBLIC_GOOGLE_CLIENT_ID` nếu muốn có đăng nhập Google; chạy smoke
   `npm run run:pipeline -- --api https://<api>/api/v1 --until S-1.4`.
