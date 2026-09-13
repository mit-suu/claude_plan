Báo cáo audit: codebase FlintFlow so với tài liệu định hướng

Phạm vi đã đọc:
- context/: đọc hết Product-Brief-to-SRS-Phases.md, srs-spine.md, business-flow.md; flintflow-main-flow.bpmn chỉ đọc tên các task và gateway.
- BE: đọc hết các model, service, route, controller, phần AI, diagram và config. Riêng scripts/ chỉ lướt.
- FE: đọc toàn bộ workspace projects/[projectId] (page, ChatPane, DocumentPane, PhaseNavBar, VerificationPane, DiscoveryStepBar, DraftReviewCard) và lib/constants/section-types.ts. Các trang auth, home, admin, draw-test chỉ grep, không đọc kỹ.
- Không chạy code, không chạy test.

Nguồn định hướng mình dùng:
- Mô hình dữ liệu và luồng ghi: srs-spine.md.
- Quy trình phase/step: Product-Brief-to-SRS-Phases.md (gọi tắt "Phases").
- Mô hình nhiều chế độ: business-flow.md, hiện vẫn ở trạng thái đề xuất.

Mức độ ảnh hưởng:
- Breaking: mâu thuẫn với mô hình dữ liệu hoặc mô hình ghi cốt lõi, phải thay module và chuyển đổi dữ liệu.
- Cần refactor: khái niệm đã có nhưng làm khác tài liệu.
- Nhỏ: sửa cục bộ.

---

1–2. Các điểm lệch hướng và mức độ ảnh hưởng

A. Mô hình dữ liệu và mô hình ghi

#: A1
Tài liệu yêu cầu: Có Spine: dữ liệu có cấu trúc, lưu quan hệ bằng khoá (actors[], use_cases[], screens[], functions[]…). Step ghi vào field, section chỉ là
kết quả render. business-flow ghi rõ "Không có tầng loại section".                                                                                        Code hiện tại: Không có Spine. Mỗi project có 25 "section typelà chuỗi markdown AI sinh). Index unique theo (projectId,type).                                                                                                                                                    Không có thực thể nào có id riêng.
File: flintflow_be/src/modules/specification/section.model.ts:112, flintflow_be/src/shared/constants/section-types.ts                                     Mức độ: Breaking
────────────────────────────────────────                                                                                                                  #: A2
Tài liệu yêu cầu: Ghi theo op: AI phát ra thao tác, code thực hiện. Mỗi step là một transaction, có spine_version, kiểm bất biến cuối lô, xoá thì cascade,
sai bất biến thì từ chối cả lô.
Code hiện tại: AI sinh lại cả section rồi ghi đè section.conte mà tài liệu cảnh báo). Không có txn, spine_version, bất biến
hay cascade.
File: specification.service.ts:109-172 (saveSection), :287-310
Mức độ: Breaking
────────────────────────────────────────
#: A3
Tài liệu yêu cầu: Danh sách section theo template FPT: fixed:1ction:<id>, fixed:I. Không có User Stories / AcceptanceCriteria
(Phases §1.3).
Code hiện tại: Có user_story, acceptance_criteria, user_journey, stakeholders, success_metrics, priority_ranking, scope_out_of_scope. Thiếu Context
Diagram
§1, Use Case Diagram §2.2.1, section feature:*/function:* cho từng màn, §I Record of Changes, §5.4 Other Requirements (chỉ có assumptions_risks gần
giống). FE gom section vào 5 chương theo cách riêng.
File: section-types.ts (BE), flintflow_fe/lib/constants/section-types.ts:333-387 (SRS_CHAPTERS)
Mức độ: Breaking
────────────────────────────────────────
#: A4
Tài liệu yêu cầu: Lịch sử là changes[] theo op (seq, txn, path, before, value, reason, by). Dùng cho undo, diff, §I.
Code hiện tại: SectionVersion lưu snapshot toàn bộ content của chuỗi cố định ("Cập nhật tài liệu"). Không có
before/path/reason.
File: section-version.model.ts, specification.service.ts:151-1
Mức độ: Cần refactor
────────────────────────────────────────
#: A5
Tài liệu yêu cầu: baselines[] có snapshot_ref, checked_at_vers sạch render từ snapshot.
Code hiện tại: Chỉ có project.baselineVersion là một chuỗi, gán cứng "v1.0", không bao giờ tăng, không có snapshot.
File: project.model.ts:13, specification.service.ts:622
Mức độ: Breaking
────────────────────────────────────────
#: A6
Tài liệu yêu cầu: Các mảng/thực thể steps[], progress{current_cursor}, assumptions[], flags[], addendum[], diagrams[],
usage[], sessions[].is_pipeline.
Code hiện tại: Không có cái nào. VerificationContext có hidden luôn là mảng rỗng, không code nào ghi vào.
File: verification-context.model.ts, verification-context.service.ts:68-74
Mức độ: Breaking
────────────────────────────────────────
#: A7
Tài liệu yêu cầu: sections[].status là hàm tính, không lưu (tránh hai nguồn sự thật). Trạng thái: draft / accepted / stale / derived, cờ phụ
awaiting_reaccept, không có regenerated.
Code hiện tại: status được lưu trong Section với enum draft/accepted/edited_manually/regenerated. progressPercent cũng lưu trong Project. Không có stale.
File: section.model.ts:33,93-97, project.model.ts:14
Mức độ: Cần refactor
────────────────────────────────────────
#: A8
Tài liệu yêu cầu: project có type, domain, complexity, form_faelease_scope, vision, goals[].
Code hiện tại: Project chỉ có name, domain, status, currentStep, currentPhase(2/3/4), workspacePhase, baselineVersion, progressPercent. currentStep mặc
định không nhất quán: model để "phase_2", service tạo project
File: project.model.ts, project.service.ts:18
Mức độ: Cần refactor

B. Quy trình phase, step và gate

#: B1
Tài liệu yêu cầu: 12 phase: B-0…B-2 (13 step) và S-1…S-9 (38 step cố định + 5×N). S-5 lặp theo từng màn hình. business-flow thêm I-1…I-4 và C-1…C-7.
Code hiện tại: Có 2 hệ phase song song: currentPhase 2/3/4 và roduct_overview / functional_spec / nfr_appendix / export).
Không có khái niệm step trong phần generation, không có vòng lặp theo màn, không có N.
File: phase-gate.service.ts, section-types.ts:246-308, project
Mức độ: Breaking
────────────────────────────────────────
#: B2
Tài liệu yêu cầu: Discovery = B-0 (Brain Dump, Form-Factor, Ststep) → B-2 (Assumption Sweep, Addendum Triage, Three-Lens
Review) → UC 2.5 Approve.
Code hiện tại: Chỉ có 6 step tương ứng B-1.1…B-1.6. Thiếu toànnh hay chưa là do LLM tự đánh giá (evaluation.isStepComplete).
Kết quả Brief chỉ nằm dưới dạng JSON trong tin nhắn chat (stepSummary), không lưu thành field có cấu trúc. Approve Summary chỉ đổi workspacePhase.
File: assets/prompts/chat_discovery.md, chat-session.service.t.ts:35-39, specification.service.ts:644-656
Mức độ: Cần refactor
────────────────────────────────────────
#: B3
Tài liệu yêu cầu: Không có S-1 phase mềm (Brief Extraction, Clmption Review, Gap List).
Code hiện tại: Không có.
File: —
Mức độ: Cần refactor (chức năng thiếu)
────────────────────────────────────────
#: B4
Tài liệu yêu cầu: Gate đặt theo step: Accept · Request revisio. Tối đa 8 lượt gọi model/step, 3 lần Regenerate/step. Có menu
[A]/[P]/[C]. Fast path gate một lần cho cả phase.
Code hiện tại: Gate đặt theo section. Nút "Yêu cầu sửa qua chat, không gọi action sửa nào. Regenerate không giới hạn. Khôngcó
Accept as-is, không có [A]/[P]/[C], không có Fast/Coaching.
File: flintflow_fe/.../DraftReviewCard.tsx, flintflow_fe/app/projects/[projectId]/page.tsx:583-587, :590-621
Mức độ: Cần refactor
────────────────────────────────────────
#: B5
Tài liệu yêu cầu: Context gửi cho model là projection theo cột Sở hữu/Đọc/Suy dẫn của step, cộng addendum[] liên quan. Không nạp toàn Spine để tránh chi
phí tăng bậc hai.
Code hiện tại: generateSection nạp toàn bộ transcript của session (không giới hạn số tin nhắn), tài liệu upload (theo map true/false từng section) và mọi
section của phase trước, kể cả bản draft, cắt ở 8000 token. Mỗpt generic generate_section.
File: specification.service.ts:243-255, document-context.service.ts:25-56,306-312, assets/prompts/generate_section.md
Mức độ: Cần refactor
────────────────────────────────────────
#: B6
Tài liệu yêu cầu: Sinh tuần tự trong khung step (Elicit → Draft → Render → Review → Gate → Meter).
Code hiện tại: FE lặp gọi /generate lần lượt cho mọi section chase"). BE còn có thêm endpoint generate-phase làm việc tương
tự, FE không dùng.
File: page.tsx:448-539, specification.service.ts:547-579
Mức độ: Cần refactor
────────────────────────────────────────
#: B7
Tài liệu yêu cầu: Section đã chốt ở phase trước vẫn sửa được t làm section liên quan thành stale). Sau baseline thì đi C-*
hoặc UC 6.8.
Code hiện tại: canModifySection khoá cứng section đã accepted  trình Change Request", nhưng không hề có Change Request.
File: phase-gate.service.ts:115-141
Mức độ: Cần refactor
────────────────────────────────────────
#: B8
Tài liệu yêu cầu: Trình tự: S-2.2 chốt release_scope từ Brief. Đến S-9.4 mới gán ưu tiên Must/Should/Could/Won't vào field priority.
Code hiện tại: UC34/UC35: sau khi có FR ở phase 3 mới chạy MoSn FR và set lại draft. Scope/out-of-scope được suy ra từ
priority. generatePriorityRanking còn bỏ qua canModifySection.
File: specification.service.ts:348-534
Mức độ: Cần refactor

C. Chỉnh sửa, undo, impact, verification, baseline

#: C1
Tài liệu yêu cầu: Document pane chỉ đọc. Mọi thay đổi đi qua c diff (UC 6.1). Chỉ có panel Tên riêng/Glossary dạng form.
Code hiện tại: DocumentPane có nút "Sửa" mở textarea, gọi PUT /specifications/projects/:id/:type để ghi đè content. Chat thường (CHAT) không sửa được
section nào. Body của PUT nhận status tuỳ ý, nên client gửi "akhông qua endpoint accept.
File: DocumentPane.tsx:235-275, specification.controller.ts:58-81
Mức độ: Breaking so với §2.3
────────────────────────────────────────
#: C2
Tài liệu yêu cầu: Undo (UC 6.13) là revert một op cuối bằng changes[].before. Resume thì revert dải first_seq..last_seq của step.
Code hiện tại: "Hoàn tác" là cắt tin nhắn chat tại một index, ctionVersion: rollback về discovery/product_overview xoá toànbộ
section, về functional_spec thì xoá phase 3 và 4. Phase xác địtin nhắn cuối còn lại.
File: chat-session.service.ts:360-506, ConfirmRollbackModal.tsx
Mức độ: Breaking (mất dữ liệu, trái mô hình op)
────────────────────────────────────────
#: C3
Tài liệu yêu cầu: Có impact query, stale, hoà giải một lượt, preview diff (UC 6.1, 6.8, 6.10).
Code hiện tại: Không có.
File: —
Mức độ: Cần refactor (thiếu)
────────────────────────────────────────
#: C4
Tài liệu yêu cầu: Traceability (UC 6.3) là truy vấn read-only trên đồ thị khoá giữa các thực thể Spine.
Code hiện tại: RequirementSourceLink nối section với đoạn tríca substring). Đây là khái niệm khác, không phải map
FR↔UC↔screen.
File: requirement-source-link.model.ts, traceability.service.t
Mức độ: Cần refactor
────────────────────────────────────────
#: C5
Tài liệu yêu cầu: S-9 gồm cờ đỏ tất định (10 luật, chặn baselie được hoặc không) và cờ vàng (cardinality + lens LLM).
Code hiện tại: Không có luật kiểm nào. "Readiness" chỉ là progressPercent (0 → BLOCKED, 100 → READY).
File: verification-context.service.ts:31-56
Mức độ: Breaking
────────────────────────────────────────
#: C6
Tài liệu yêu cầu: Không có (tài liệu yêu cầu dữ liệu thật).
Code hiện tại: VerificationPane FE hiển thị dữ liệu demo hardcode: 78%, facts "Sinh viên & Tài xế", "84% aligned" viết cứng trong JSX. Nút Xác nhận/Sửa
không gắn handler. Dữ liệu FE mong đợi (readinessLevel, complei dữ liệu BE trả về.
File: flintflow_fe/.../VerificationPane.tsx:38-80,254-259,230-241
Mức độ: Cần refactor
────────────────────────────────────────
#: C7
Tài liệu yêu cầu: Điều kiện baseline: cờ đỏ chưa resolve và chưa waive bằng 0, quét lại trên spine_version, có v1.0-conditional. "Không đặt ngưỡng phần
trăm."
Code hiện tại: Baseline khi đủ 22 section accepted. Không quét lại, không waive, không conditional.
File: specification.service.ts:594-636
Mức độ: Cần refactor
────────────────────────────────────────
#: C8
Tài liệu yêu cầu: Export không bị chặn bởi chất lượng: có sau  watermark DRAFT.
Code hiện tại: FE khoá Export cho đến khi progress đạt 100%.
File: PhaseNavBar.tsx:52-54,84
Mức độ: Cần refactor
────────────────────────────────────────
#: C9
Tài liệu yêu cầu: Thanh tiến độ đếm step. Điểm sẵn sàng = accesection derived).
Code hiện tại: Progress = số section accepted / 22 section mappedToTemplate.
File: phase-gate.service.ts:147-159
Mức độ: Nhỏ (đi theo A7)

D. Diagram, prompt, ngôn ngữ

#: D1
Tài liệu yêu cầu: Một đường duy nhất cho diagram: .puml → diage-check → PlantUML server → nhúng khi export. Có 5 loại:
context, usecase, screen_flow, erd, screen_layout.
Code hiện tại: Client PlantUML và compile-check đã có nhưng chối vào luồng sinh. erd và screen_flow chỉ là markdown do
generate_section viết. Song song còn một pipeline Excalidraw (diagram_classify/generate, dùng assets/diagram-skill), mở qua /ai-actions/execute với
generate_diagram và trang FE /draw-test.
File: shared/diagram/*, modules/drawtest/diagram-pipeline.service.ts, ai-action.controller.ts:48-63, flintflow_fe/app/draw-test/page.tsx
Mức độ: Cần refactor (riêng Excalidraw: nhỏ, vì tách biệt)
────────────────────────────────────────
#: D2
Tài liệu yêu cầu: 29 file skill theo cấu trúc BMAD: 10 skill hành động, 12 skill nội dung, 5 renderer, 2 đầu ra.
Code hiện tại: 8 prompt phẳng: chat, chat_discovery, generate_ope_out_of_scope, summarize_document, diagram_classify,
diagram_generate.
File: assets/prompts/
Mức độ: Cần refactor
────────────────────────────────────────
#: D3
Tài liệu yêu cầu: "Vòng một không có DB override", nguồn sự thset_version.
Code hiện tại: Registry đọc DB trước, không có mới đọc file trên đĩa. Có trang admin CRUD/activate version prompt. Không ghi asset_version.
File: prompt-registry.service.ts:31-47, modules/admin/prompt-tdmin/prompt-templates/page.tsx
Mức độ: Cần refactor
────────────────────────────────────────
#: D4
Tài liệu yêu cầu: Nội dung render vào SRS viết tiếng Anh (Phasất đổi thành theo ngôn ngữ của profile. Hội thoại theo ngôn ngữ
user.
Code hiện tại: Prompt generate_section viết tiếng Việt và không yêu cầu ngôn ngữ đầu ra, nên nội dung thực tế ra tiếng Việt. Label section là song ngữ
Việt. Không có cờ non_english_content.
File: assets/prompts/generate_section.md, section-types.ts
Mức độ: Nhỏ (ở mức prompt)
────────────────────────────────────────
#: D5
Tài liệu yêu cầu: Khi op sai schema: gửi lại cho model kèm lỗi, tối đa 2 lần, sau đó hỏi user.
Code hiện tại: Khi output generate_section không parse được JSt làm content và code lưu vào section.
File: response-parser.ts:318-320, specification.service.ts:301
Mức độ: Cần refactor

E. Credit, session, đồng thời, vai trò

#: E1
Tài liệu yêu cầu: usage[] có step_id, call_kind, attempt, tokens, cost, state, expires_at. Bản ghi reserved quá hạn thì bị dọn.
Code hiện tại: Có ví balance/reserved và ledger CreditTransactông có expires_at, nên nếu process chết giữa reserve và deduct
thì credit bị treo vĩnh viễn. Ví mới được tặng cứng 100 credit. Model Subscription có nhưng không nơi nào dùng.
File: credit-reservation.service.ts:43, credit-transaction.mod
Mức độ: Cần refactor
────────────────────────────────────────
#: E2
Tài liệu yêu cầu: Deduct khi thành công, refund khi thất bại.
Code hiện tại: Ở luồng stream, deduct chạy trước parseResponse. Nếu parse ném lỗi thì nhánh catch lại releaseCredit, nên cùng một lượt vừa bị trừ vừa được

giải phóng reserved. Math.max(0) che mất lỗi này.
File: ai-action.service.ts:275,290,327
Mức độ: Nhỏ
────────────────────────────────────────
#: E3
Tài liệu yêu cầu: Mỗi transaction mang base_version, lệch thì
Code hiện tại: Không kiểm tra phiên bản, hai tab ghi đè lẫn nhau.
File: —
Mức độ: Cần refactor
────────────────────────────────────────
#: E4
Tài liệu yêu cầu: progress thuộc project. Đúng một session có y pipeline, session khác chỉ phát op sửa.
Code hiện tại: createChatSession tắt isActive của các session khác. Session nào cũng gọi generate được, và nội dung sinh ra phụ thuộc transcript của
session đang chọn. Nếu chưa có tin nhắn nào mang workspacePhasry.
File: chat-session.service.ts:10-19, specification.service.ts:238-255
Mức độ: Cần refactor
────────────────────────────────────────
#: E5
Tài liệu yêu cầu: business-flow: 3 vai trò Lead / Analyst / Viewer (dời sang P5).
Code hiện tại: Chỉ có user/admin, project một chủ, không cộng ation, chat và verification không kiểm tra project có thuộcuser
 không (chỉ project và document service kiểm).
File: user.model.ts:52-56, specification.controller.ts, chat-session.controller.ts
Mức độ: Nhỏ (P5)
────────────────────────────────────────
#: E6
Tài liệu yêu cầu: Ranh giới dữ liệu: tài liệu upload chỉ gửi phần trích xuất cho model.
Code hiện tại: Nếu còn budget thì gửi toàn bộ extractedText, cet. File gốc lưu trên Cloudinary.
File: document-context.service.ts:187-217, project-document.service.ts:97
Mức độ: Nhỏ

F. Phần hoàn toàn chưa có nhưng thuộc định hướng hoặc phạm vi

┌─────┬───────────────────────────────────────────────────────────────────────────────────────────┬─────────────────────┐
│  #  │                 Chức năng theo tài liệu                 │                          Hiện trạng                          │       Mức độ        │
├─────┼───────────────────────────────────────────────────────────────────────────────────────────┼─────────────────────┤
│     │ Chế độ 1 (business-flow): import .docx (I-1…I-4), neo   │ Không có. Upload hiện tại dùng mammoth.extractRawText, mất   │                     │
│ F1  │ paraId, template profile, Writer ghi Track Changes, dấng/paraId, và chỉ phục vụ làm       │ Breaking (nếu nhóm  │
│     │  version trong file, C-1…C-7. business-flow đặt đây là  │ context.                                                     │ chốt business-flow) │
│     │ P1, demo chính.                                                                           │                     │
├─────┼─────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┼─────────────────────┤
│     │                                                       markdown rồi copy hoặc tải file     │                     │
│ F2  │ Export Word + watermark DRAFT (vòng một), Writer điền   │ .md. ChatPane vẫn ghi "kích hoạt quyền xuất bản              │ Cần refactor        │
│     │ vào mẫu FPT.                                          tsx:543-547). package.json không có │ (thiếu)             │
│     │                                                         │  thư viện docx.                                              │                     │
├─────┼───────────────────────────────────────────────────────────────────────────────────────────┼─────────────────────┤
│ F3  │ Notification in-app 11.1, Admin đọc 10.1–10.3, Billing  │ Không có. Admin hiện chỉ có trang prompt template.           │ Thiếu (mảng nền     │
│     │ 9.1/9.2 + mock gateway, Onboarding 1.12.                                                  │ tảng)               │
├─────┼─────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┼─────────────────────┤
│ F4  │ Share / clone / revoke: hoãn theo §9.1.                                                   │ —                   │
└─────┴─────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────┴─────────────────────┘

---

3. Phần đã đúng hướng, không cần đụng

┌──────────────────────┬──────────────────────────────────────────────────────────────────────┬───────────────────────────────────────────────────────┐
│         Phần         │                          Vì sao đúng    │                         File                          │
├──────────────────────┼──────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│                      │ Coi đĩa là nguồn sự thật (§8), quét đ   │                                                       │
│ Loader prompt asset  │ bỏ _archive, trùng actionType thì fail ngay, kiểm lúc khởi động, có  │ shared/ai/prompt-assets.ts, prompt-assets.test.ts,    │
│ trên đĩa             │ test chặn asset thiếu hoặc mồ côi. (C   │ config/startup-checks.ts, server.ts:8-9               │
│                      │ tiên là lệch, xem D3.)                                               │                                                       │
├──────────────────────┼─────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│                      │ Khớp §7.1: gọi POST rồi fallback GET, phát hiện lỗi dù server trả    │ shared/diagram/plantuml.client.ts, compile-check.ts,  │
│ PlantUML self-host   │ HTTP 200 (header hoặc quét SVG), enco   │ plantuml.test.ts, docker-compose.yml,                 │
│                      │ source_hash, có service trong docker-compose và biến                 │ config/env.ts:94-95                                   │
│                      │ PLANTUML_BASE_URL.                      │                                                       │
├──────────────────────┼──────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│                      │ Một entrypoint duy nhất: reserve theo   │ shared/ai/ai-action.service.ts, retry.service.ts,     │
│ AI Action framework  │ deduct hoặc release → ghi AiActionLog (token, latency, provider).    │ credit-reservation.service.ts,                        │
│                      │ Đây là nền cho Meter và UC 10.3. Retr   │ admin/ai-action-log.model.ts                          │
├──────────────────────┼──────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Giữ nội dung cũ khi  │ Section chỉ được ghi sau khi AI trả vn  │ specification.service.ts:287-310                      │
│ AI lỗi               │ bản ổn định gần nhất".                                               │                                                       │
├──────────────────────┼─────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│                      │ Register, verify email, forgot/reset (email cho các ca               │                                                       │
│ Auth                 │ auth-critical), Google, refresh rotat   │ modules/auth/*, shared/auth/*, shared/email/*         │
│                      │ nền cho UC 1.1–1.9.                                                  │                                                       │
├──────────────────────┼─────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Project CRUD +       │                                                                      │                                                       │
│ archive mềm +        │ Nền cho UC 1.x, 1.10, 1.11.             │ modules/project/project.*, chat-session.*             │
│ rename, nhiều chat   │                                                                      │                                                       │
│ session              │                                         │                                                       │
├──────────────────────┼──────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Upload tài liệu tham │ Parse, tóm tắt, chia token budget: nề   │ project-document.service.ts,                          │
│  khảo                │                                                                      │ document-context.service.ts:131-244                   │
├──────────────────────┼─────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Nội dung 6 step      │ Tên và nội dung khớp đúng B-1.1…B-1.6.                               │ assets/prompts/chat_discovery.md                      │
│ Discovery            │                                         │                                                       │
├──────────────────────┼──────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Kiểm tra env         │ Fail-fast khi thiếu secret ở producti   │ config/env.ts                                         │
└──────────────────────┴──────────────────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────┘

---

4. Chỗ tài liệu chưa rõ hoặc thiếu thông tin để quyết định

1. business-flow.md vẫn là đề xuất (§9 "Cần chốt"). Nếu chốt thì P1 là chế độ 1 (import + CR). Nếu không, Phases §9.3 lại xếp thứ tự: fixture → S-3 →
   export → B-*. Hai thứ tự ưu tiên đang mâu thuẫn, nên chưa qnằm ngoài phạm vi.
2. Thiếu tài liệu được tham chiếu: usecase.md, SRS_template_FPT.md, scope-revision-options.md, srs-spine.schema.json không có trong context/. Vì vậy không
   đối chiếu được số UC và nội dung UC.
3. Số UC và actor không khớp nhau:
   - CLAUDE.md nói 84 UC, 11 nhóm, 4 actor (Guest/User/Admin/P
   - Phases §9.2 nói "33/71 UC".
   - business-flow dùng vai trò Lead/Analyst/Viewer.
4. User Stories, Sequence Diagram, PDF/CSV:
   - Phases §1.3 và §7.1 loại bỏ User Stories/AC và Sequence D
   - Danh sách nhóm UC trong CLAUDE.md vẫn ghi user stories, sequence diagram, user stories CSV.
   - Vì vậy chưa kết luận được user_story và acceptance_criterg.
5. Brief lưu ở đâu: lược đồ Spine chỉ có project{vision, goals[]…}. Output của B-1.2 (personas, JTBD, stakeholders), B-1.3 (value prop, differentiators)
   và B-1.5 (success metrics) không có field tương ứng. S-1.1 ng định nghĩa cấu trúc đó. Nên chưa biết số phận các sectionvalue_proposition, business_goals, success_metrics, stakeholders hiện có.
6. Nơi lưu Spine: tài liệu không nói Spine là một document Monollection; snapshot_ref trỏ đi đâu; spine_version vàbase_version được khoá như thế nào.
7. Ngôn ngữ nội dung: Phases nói tiếng Anh, business-flow đề x
8. Chat ngoài pipeline: nói "session khác chỉ phát op sửa", nhưng không rõ chat hỏi đáp tự do (như CHAT hiện tại) có còn được phép không.
9. Chuyện gate: bảng business-flow ghi "gate mỗi step", còn Phmột lần mỗi phase. Chưa rõ quy tắc áp cho S-* khi đang ở Fastpath.
10. Prompt override qua admin: §8 ghi "vòng một không có DB ovnày". Nhưng code đã có trang admin. Tài liệu không nói nên giữhay bỏ.
11. Hai loại upload: reference docs (UC 2.2, nhận pdf/docx/md/ .docx). Tài liệu không tách rõ pipeline, lưu trữ và thời hạnlưu của hai loại (§10 spine cũng ghi "chưa quyết"). Việc dùng Cloudinary (bên thứ ba) chưa được xét trong §4.3 ranh giới dữ liệu.
12. Credit: có nói reserve theo lượt, nhưng không có bảng giá Pro và monthly_reset cũng không mô tả.
13. Quan hệ MoSCoW và scope: S-2.2 in/out scope và S-9.4 priority không được định nghĩa quan hệ với nhau (Won't-have có đồng nghĩa với out-of-scope
    không).
14. Assumption bị reject: hoàn nguyên giá trị hay chỉ đánh dấu, srs-spine.md §10 vẫn còn treo.
15. File BPMN khác Phases: flintflow-main-flow.bpmn (T1–T31) c & UC → Screens/ERD → NFR & Appendix), cổng Purchase Creditsđặt trước khi generate, và thứ tự Quality Check → Completeness → Goal Validation. Luồng này gần với code hiện tại hơn là với 12 phase của Phases. Tài liệu không nói file nào có thẩm quyền.

---