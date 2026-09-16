# Task 20 — Discovery B-0 → B-2 + S-1 qua step runner (BE + FE)

**Wave:** 4 · **Người phụ trách:** D · **Effort:** 9 điểm · **Trạng thái:** [ ] Chưa làm  [x] Đang làm (mock xong, chờ lượt chạy thật trên dev)  [ ] Xong · nhánh `feat/FLF-160-discovery-brief` (BE + FE)

## Mục tiêu
Đưa Product Brief về đúng Phases §5: 13 step (B-0 4 step, B-1 6 step, B-2 3 step) + S-1 (4 step mềm) chạy qua step runner, ghi `project{}`, `addendum[]`, `assumptions[]`, `other_requirements[]` bằng op; gate do user, không do LLM tự đánh giá; approve mở S-2. FE refit chat pane theo mô hình step.

## Lệch hướng audit cần đóng
B2 (chỉ 6 step B-1, LLM tự quyết hoàn thành, Brief chỉ nằm trong JSON tin nhắn), B3 (không có S-1), E4 phần Discovery, một phần A8 (form_factor/stakes/working_mode).

## File / module liên quan
### Hiện có (đọc / sửa / xoá)
- Đọc: `context/Product-Brief-to-SRS-Phases.md` §5.1–5.4, §6.4 (S-1.x); `assets/prompts/chat_discovery.md` (nội dung 6 step đúng, tái dùng); `modules/pipeline/*` (T11, T13); `assets/skills/content/project-classifier` (T14, S-1.2).
- Sửa: `flintflow_be/src/modules/project/chat-session.service.ts` (bỏ nhánh `CHAT_DISCOVERY` + `buildCompletedStepsSummary`; tin nhắn trong session pipeline khi step B-* đang Elicit chuyển vào `POST /steps/:id/answer`).
- Sửa: `flintflow_be/src/modules/pipeline/step-runner.service.ts` (Fast path B-*: gộp câu hỏi ≤ 2 lượt/phase; B-2.1 duyệt lô theo `target_section`, vẫn duyệt lẻ giả định chạm bất biến hoặc nuôi §4.2.2/§4.2.3).
- Gỡ khỏi route: `POST /specifications/projects/:id/advance-to-generation` (xoá file T21).
- Sửa FE: `flintflow_fe/app/projects/[projectId]/_components/{ChatPane.tsx, DiscoveryStepBar.tsx, SummaryReviewCard.tsx, StepTransitionBanner.tsx, ChatBubble.tsx}`: bỏ `lastEvaluation`, `latestAiQuestions` regex, `discoverySummaryData` map cứng, `completedSteps` parse transcript; dữ liệu từ `useProgress`/`useSpine`; DiscoveryStepBar thay bằng `StepProgressBar` (T12) cho B-*; SummaryReviewCard hiển thị `project{}` + addendum nhóm theo `target_section` từ Spine; StepTransitionBanner thay bằng GateCard; ChatBubble bỏ `parseAiMessage` 4 tầng (BE trả `reply` sạch qua SSE).
- Sửa FE: `flintflow_fe/lib/constants/section-types.ts` `DISCOVERY_STEPS` lấy từ step registry (giữ `sampleQuestions` chuyển vào skill).
### Tạo mới / điền
- `assets/skills/content/product-brief/` với `references/{b0-intake.md, b1-steps.md, b2-finalize.md, three-lens.md}` và `assets/brief-template.md` (BMAD, giữ notice): B-0.1 Brain Dump (đọc tài liệu upload qua `document-context`, bóc tách, tóm tắt xác nhận, ghi `addendum[]` ngay), B-0.2 `project.form_factor`, B-0.3 `project.stakes`, B-0.4 `project.working_mode`; B-1.1 ghi `project.vision`, `project.goals[]`, addendum problem; B-1.2 addendum personas/JTBD/stakeholders (`target_section: fixed:2.1`); B-1.3 addendum value prop (`fixed:1`); B-1.4 addendum MVP scope/feature hypotheses (`fixed:1`, `fixed:3.1.2`); B-1.5 addendum metrics (`fixed:4.2.x`); B-1.6 `other_requirements[kind=risk|assumption|open_question]` + `assumptions[]`; B-2.1 Assumption Sweep; B-2.2 Addendum Triage (giữ/để dành/bỏ); B-2.3 Three-Lens (Skeptic/Opportunity/Contextual, tự định nghĩa) rồi gate Approve (UC 2.5) mở S-1.
- `assets/skills/content/brief-analysis/` (S-1.1 Brief Extraction: B-1 thành `project` + addendum cấu trúc; S-1.3 Conflict & Assumption Review: trình `assumptions[]`/xung đột cho user; S-1.4 Gap List: addendum `kind=gap`).
- `fixtures/op-cases/b0-s1/*.json`; `flintflow_be/src/modules/pipeline/skills/brief.e2e.test.ts` (project trống đi B-0.1 … S-1.4).
- FE: `_components/AssumptionSweepPanel.tsx` (B-2.1 duyệt lẻ/lô), `_components/AddendumTriagePanel.tsx` (B-2.2), `_components/BriefSummaryCard.tsx` (thay SummaryReviewCard).

## Các bước implement
1. Skill product-brief (tái dùng nội dung chat_discovery) + brief-analysis.
2. Runner: Fast/Coaching cho B-*, B-2.1 duyệt lô, approve B-2.3 mở S-1.
3. Gỡ CHAT_DISCOVERY khỏi chat-session; gỡ advance-to-generation.
4. FE refit (xoá parse transcript), 3 panel mới.
5. E2E mock + `E2E_AI=1`; tạo project mới trên dev đi trọn B-0 đến S-1.4 rồi S-2.1.

## Dependency
- Phụ thuộc: T11, T13, T12, T14 (project-classifier).
- Chặn: T21, T23, M4.
- Chạy song song với: T17, T18, T19.

## Output kỳ vọng
- Discovery đúng 13 + 4 step, ghi Spine; FE không còn suy state từ JSON tin nhắn.

## Tiêu chí hoàn thành (DoD)
- [x] **Trên mock**: project rỗng đi trọn 13 step Brief + 4 step S-1 (`brief.e2e.test.ts`); đủ 5 trường `project{}`, `addendum[]` = 8, `assumptions[]` = 3, và Brief không chạm `actors/use_cases/screens/functions/sections`. Lượt chạy thật trên dev chưa làm.
- [x] Coaching: 4 step đầu ⇒ 4 lượt elicit, mỗi step một gate. Fast: phase B-0 ≤ 2 lượt (`brief.e2e.test.ts`).
- [x] `grep -rn "evaluation" flintflow_fe/app/projects` **rỗng**. ChatBubble bỏ `DiscoveryEvaluation`, thanh completeness và `parseAiMessage` 4 tầng (còn một tầng: parse JSON lấy `reply`).
- [x] Accept B-2.3 ⇒ `nextStep` trỏ `S-1.1`; chạy S-1.1 đặt `progress.current_phase = S-1`; S-1.4 accepted ⇒ `nextStep` trỏ `S-2.1`.

## Ghi chú / rủi ro
- Câu trả lời Elicit giữ trong transcript session pipeline và tái dùng khi Regenerate (Phases §4.4).

## Kết quả (2026-09-16)

- Nhánh `feat/FLF-160-discovery-brief` ở **cả hai repo**.
  BE: 2 commit, 31 file (+1266 / −80) — typecheck sạch, **667 test xanh / 14 skip**.
  FE: 1 commit, 7 file (+539 / −122) — typecheck sạch, lint 0 lỗi, **192 test xanh**.
- BE: skill `product-brief` viết thật (+ 4 references + brief-template), skill mới `brief-analysis`;
  bỏ hẳn `CHAT_DISCOVERY` và `buildCompletedStepsSummary`; tin nhắn ở session pipeline khi step đang chờ
  trả lời đi thẳng vào `submitAnswer`; gỡ route `advance-to-generation`; 17 fixture op-case + e2e.
- FE: `BriefSummaryCard`, `AssumptionSweepPanel`, `AddendumTriagePanel` (chỉ hiện ở pha B-*/S-1, ghi qua
  `POST /changes`), ChatBubble dọn sạch.
- Quyết định đáng chú ý: **"để dành" ở B-2.2 là đổi `target_section` sang `fixed:5.4`**, không phải chuyển
  sang `other_requirements[]` — step registry chỉ cho B-2.2 ghi `addendum`/`assumptions`, và S-7.4 (T18) đã
  đọc addendum nhắm `fixed:5.4`. Thông tin không mất, quay lại được.
- **Cần trước khi merge**: XREQ T20→T11 cho 2 dòng `STEP_SKILLS` (S-1.1/1.3/1.4 → `brief-analysis`).
- **Việc còn lại**: tạo project mới trên dev, đi trọn B-0.1 → S-1.4 rồi S-2.1 với provider thật (bước 5
  của task) — chưa chạy vì tốn credit và cần một project mới.
