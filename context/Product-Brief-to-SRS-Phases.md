# Plan — User Idea → Product Brief → SRS

> Các phase người dùng trải qua khi chat với AI để hoàn thành **Product Brief**, và từ Brief sinh ra **SRS** theo template FPT.
> Brief là input chính của SRS — không bắt đầu SRS từ đầu.
> **Mô hình dữ liệu ở `srs-spine.md`** — đọc file đó trước. File này chỉ nói quy trình.

```text
User Idea
    ↓
┌─ PRODUCT BRIEF ────────────────────────────────────────────────────────┐
│  [B-0 Intake]         4 step  (mềm)                                    │
│  [B-1 Product Brief]  6 step                                           │
│  [B-2 Brief Finalize] 3 step   →  ★ Brief approved ★  (UC 2.5)         │
└─────────┬──────────────────────────────────────────────────────────────┘
          ↓
┌─ SRS GENERATION ── mỗi step: Elicit→Draft→Render→Review→Gate→Meter ────┐
│  [S-1 Analyze & Validate Brief]     4 step  (mềm)                      │
│  [S-2 Product Overview]             5 step  §1                         │
│  [S-3 User Requirements]            6 step  §2                         │
│  [S-4 System Functional Overview]   5 step  §3.1   ← chốt N ở S-4.1    │
│                                                                        │
│  ┌──►[S-5 Feature & Function Details] 5 step  §3.2  (lặp theo MÀN HÌNH)│
│  │      ↓                                                              │
│  └──────┤ còn màn `pending`? ──► quay lại S-5.1, giữ con trỏ           │
│         ↓ hết queue → một vòng cuối cho non-screen function            │
│  [S-6 Non-Functional Requirements]  5 step  §4                         │
│  [S-7 Requirement Appendix]         4 step  §5.1–5.4  (suy ra)         │
│  [S-8 Glossary & Assemble]          4 step  §5.5 → Working Draft       │
│  [S-9 Verify, Validate & Baseline]  5 step  (pha gate cuối)            │
│         │                                                              │
│         ├─ còn cờ đỏ chưa resolve và chưa waive ──► về step sở hữu     │
│         ↓ = 0                                                          │
│  ★ Approved SRS Baseline v1.0 ★  (UC 6.14)                             │
└──┬─────────────────────────────────────────────────────┬───────────────┘
   │ từ Working Draft                                    │ từ snapshot Baseline
   ↓ (watermark DRAFT + danh sách cờ)                    ↓ (bản sạch)
   └──────────► Export — Word (UC 8.1) · PDF (UC 8.2) · Handoff (UC 8.3) ◄
```

## Mục lục

[0. Tóm tắt](#0-tóm-tắt) · [1. Quy ước](#1-quy-ước) · [2. Mô hình ghi](#2-mô-hình-ghi) · [3. Khung hành động](#3-khung-hành-động) · [4. Vận hành](#4-vận-hành) · [5. Product Brief](#5-product-brief--b-0--b-2) · [6. SRS Generation](#6-srs-generation--s-1--s-9) · [7. Diagram](#7-diagram) · [8. Bản đồ file md](#8-bản-đồ-file-md) · [9. Phạm vi vòng một](#9-phạm-vi-vòng-một) · [10. Câu chưa chốt](#10-câu-chưa-chốt)

**Tra nhanh:** một step → [6.4](#64-toàn-bộ-step--nguồn-sự-thật) · một field → `srs-spine.md` §2 · field nào nuôi section nào → `srs-spine.md` §4 · cờ đỏ → `srs-spine.md` §7 · file `.md` → [8.2](#82-nhóm-tự-viết) · ai làm gì → [9.2](#92-ngoài-pipeline--3371-uc)

---

## 0. Tóm tắt

**Chia việc:** 3 người pipeline (Spine engine · 29 file skill · renderer · assemble/export) · 2 người nền tảng (auth · project CRUD · chat session · credit ledger · admin · notification). Chi tiết [9.2](#92-ngoài-pipeline--3371-uc).

1. **Ba tầng: phase — step — section** ([1.1](#11-phase--step--section)). Step ghi field vào Spine, section render từ field. 12 phase, **51 step cố định + 5 × N**.
2. **Spine lưu quan hệ bằng khoá.** Impact analysis, `stale`, undo, truy vết (UC 6.3) đều là hệ quả. Quyết định kỹ thuật quan trọng nhất của sản phẩm.
3. **Op-based write.** AI phát ra thao tác, code thực hiện. Một step = một transaction.
4. **Mọi phase chạy cùng khung hành động** — Elicit → Draft → Render → Review → Gate → Meter.
5. **Kiểm tất định tách khỏi phán đoán model.** Chỉ cờ đỏ tất định chặn baseline; mọi cờ đỏ đều có đường sửa.
6. **Hai phase mềm: B-0 và S-1.** S-5 là loop theo màn hình, S-9 là pha gate cuối. Còn lại cố định vì template FPT là hợp đồng.
7. **AI chỉ hỏi phần thiếu** — field nào của Spine còn trống, cộng `addendum[]` đã có.
8. **Export không bị chặn bởi chất lượng** — sau S-8.2. Bản Working Draft có watermark DRAFT và in danh sách cờ.
9. **Vòng một không làm hết** — phạm vi ở [mục 9](#9-phạm-vi-vòng-một).
10. **Kế thừa BMAD**: Discovery của `bmad-product-brief`; khung Activation → Discovery → Reviewer Gate → Finalize; `readiness-gate` cho soft gate.

---

## 1. Quy ước

### 1.1 Phase — Step — Section

| Tầng | Là gì | Ai thấy | ID |
| --- | --- | --- | --- |
| **Phase** | Milestone; đơn vị đào sâu `[A]/[P]/[C]` | Chat header | `B-0`…`B-2`, `S-1`…`S-9` |
| **Step** | Đơn vị chốt; đơn vị hiển thị tiến độ | Thanh tiến độ | `<phase>.<số>`; S-5 là `<phase>.<số>@<screen_id>` |
| **Section** | Mục trong SRS theo template FPT | Người đọc SRS | `fixed:3.1.3`, `feature:<id>`, `function:<id>` |

**Step không viết section. Step ghi field vào Spine; section render từ field.**

```text
Step  ──ghi op──►  Spine field  ──render──►  Section
```

Không quan hệ nào là 1–1: nhiều step → một section; một step → nhiều section; step không ra section nào (B-0, B-1, B-2, S-1, S-9).

Thanh tiến độ đếm **step**; ô trạng thái tài liệu đếm **section**. Nhãn UI hiển thị chữ "Bước", tương ứng với step.

**N = số màn hình + 1** nếu có non-screen function. N chốt ở **S-4.1**; trước đó thanh tiến độ hiển thị theo phase, không hiển thị phần trăm.

### 1.2 Ký hiệu

| Ký hiệu | Trỏ tới |
| --- | --- |
| `§2.1`, `§3.1.3` | section của **SRS** theo template FPT |
| `mục 2.1`, `mục 5.3` | mục của **tài liệu này** |
| `UC 2.1`, `UC 6.14` | use case trong `usecase.md` |
| `(chưa có UC)` | chức năng đã chốt, chưa có use case — [mục 10](#10-câu-chưa-chốt) |

**Bảng [6.4](#64-toàn-bộ-step--nguồn-sự-thật) là nguồn sự thật duy nhất về số step.** Sơ đồ đầu file và bảng 6.1 là bản chiếu, phải khớp với nó.

Từ **"mục"** chỉ dùng cho mục của tài liệu này. Mục của SRS gọi là **section**. Một phần tử `addendum[]` gọi là **entry**.

### 1.3 Ngôn ngữ

| Thứ | Ngôn ngữ |
| --- | --- |
| Nội dung sẽ render vào SRS: tên actor, use case, màn hình, mô tả, NFR, business rule, message, glossary | **Tiếng Anh** |
| Nhãn trong `.puml` | **Tiếng Anh** |
| `addendum[].content` | **Nguyên văn user**; `content_en` là bản dịch dùng để elicit |
| `changes[].reason`, `flags[].message`, `waive_reason` | Theo ngôn ngữ user; dịch lúc export §I |
| `glossary[].term_native` | Tuỳ chọn |
| Hội thoại chat, nhãn UI | **Theo user** |

Cờ vàng `non_english_content` (`srs-spine.md` §8.1) bắt field tiếng Anh còn ký tự có dấu.

**Không có User Stories và Acceptance Criteria.** Template FPT không có hai section này; §3.2.1 Function Details đã chứa actor, mục đích, giao diện, xử lý dữ liệu, luồng thường/bất thường.

---

## 2. Mô hình ghi

> Lược đồ, bảng ánh xạ, bất biến, cờ đỏ: **`srs-spine.md`**. Mục này chỉ nói cách step tương tác với nó.

### 2.1 Op-based write

```json
{ "txn": "t42", "op": "set", "path": "actors[id=A03].name",
  "value": "Administrator", "reason": "typo" }
```

Model chỉ quyết định *sửa cái gì*; việc sửa là code thuần. Nếu model sinh lại cả section thì các phần không liên quan đổi giọng, mất vế → **tài liệu trôi**. Thao tác thì tất định, rẻ, undo được (UC 6.13), diff được (UC 6.1).

**Một step phát một transaction.** Bất biến kiểm ở cuối lô, `spine_version` tăng một lần. Xoá phần tử sinh op cascade trong cùng lô — chi tiết `srs-spine.md` §3.

Trường `reason` là nhật ký, dùng cho §I Record of Changes và UC 6.12.

### 2.2 Ba nhánh xử lý thay đổi

Xem `srs-spine.md` §9. Tóm tắt: không đụng ai → áp im lặng · có phụ thuộc → preview diff rồi áp, section liên quan tính ra `stale` · sau baseline → UC 6.8 · phá bất biến → từ chối cả lô.

**Hoà giải** là thao tác user bấm để quét mọi section `stale` và đề xuất cập nhật theo field phụ thuộc, xử lý **một lượt** thay vì từng cái. Hoà giải chỉ đọc field phụ thuộc, đi qua preview diff, và render lại mọi `diagrams[]` có `source_hash` lệch.

### 2.3 Sửa qua hội thoại — Document pane read-only

Mọi thay đổi đi qua chat (UC 6.2 trước baseline, UC 6.8 sau baseline), preview diff (UC 6.1), highlight vùng đã đổi.

Bù cho việc không có editor:

- **Panel Tên riêng / Glossary** — khai tên chuẩn của actor, entity, hệ thống một lần qua form. Đây là field trong Spine nên sửa một chỗ lan ra mọi section.
- **Preview diff** — user thấy đúng chỗ sẽ đổi trước khi xác nhận.

> **Giới hạn:** Panel Tên riêng chỉ sửa được field có khoá. Tên được nhắc trong các field văn xuôi không tự đổi theo — S-8.4 Consistency Pass bắt phần này.

**Read-only view (UC 1.14)** dùng một projection của Spine: ẩn `flags[]`, `assumptions[]`, `changes[].by/reason`, điểm sẵn sàng, nhãn `stale`. Section bắt buộc chưa `accepted` hiển thị tiêu đề kèm dòng "chưa hoàn thiện", **không ẩn im lặng** — ẩn thì người xem tưởng SRS thiếu section.

---

## 3. Khung hành động

```text
PHASE  ├─ Intake        đọc projection Spine của phase; đọc addendum[] liên quan
       │                                              ← 1 lần đầu phase
       ├─ Step 1 ─┐
       ├─ Step 2 ─┤ mỗi step:  Elicit → Draft → Render → Review → Gate → Meter
       ├─ …      ─┘
       └─ Menu đào sâu [A] / [P] / [C]                ← 1 lần cuối phase
```

| Hành động | Làm gì |
| --- | --- |
| **Elicit** | Chỉ hỏi phần còn thiếu. Fast path → gom ≤ 2 lượt mỗi phase, ghi `assumptions[]`. Coaching path → hỏi theo nhóm, vặn lại khi câu trả lời mỏng |
| **Draft** | Sinh transaction op, validate schema, kiểm bất biến ở cuối lô |
| **Render** | Xuất `.puml` vào `diagrams[]`, compile-check, ghi `source_hash` |
| **Review** | Chạy hàm kiểm tất định + lens LLM, ghi `flags[]` |
| **Gate** | Cổng chốt: **Accept · Request revision · Regenerate** |
| **Meter** | Deduct credit, ghi `usage[]` (UC 9.6) |

**Context = projection, không phải toàn Spine.** Mỗi step chỉ nạp field thuộc cột Sở hữu / Đọc / Suy dẫn của nó (`srs-spine.md` §4) cộng `addendum[target_section]` liên quan. Nạp toàn Spine làm chi phí input tăng theo tiến độ, tổng chi phí thành **bậc hai** theo số step.

**Trần gọi model: 8 lượt / step**, gồm mọi loại — Elicit, Draft, retry schema, compile-check, Review, Regenerate, Request revision. Counter reset khi step chuyển `accepted`; quay lại một step đã accepted thì counter bắt đầu lại.

**Trần Regenerate: 3 lần / step.** Từ lần thứ tư, nút Regenerate tắt, chỉ còn Request revision. Ở S-5.4, Regenerate áp ở **mức function**, không phải mức step — sinh lại 20 function để sửa 1 cái là lãng phí.

Sáu hành động viết **một lần**, dùng cho cả 12 phase — nhóm A ở [8.2](#82-nhóm-tự-viết).

---

## 4. Vận hành

### 4.1 Xử lý lỗi

| Lỗi | Xử lý | UC |
| --- | --- | --- |
| Op sai schema / path không phân giải | Validate trước khi áp; sai thì gửi lại cho model kèm thông báo lỗi, tối đa 2 lần, rồi hỏi user | 6.11 |
| `.puml` không compile | Compile-check → sửa → thử lại (tối đa 2 lần); vẫn lỗi thì lưu text, `render_status = error` → cờ đỏ `render_error` | «extend» Handle AI Generation Failure |
| Gọi model lỗi / timeout | Retry; giữ nguyên bản ổn định gần nhất | «extend» Handle AI Generation Failure |
| Hai tab cùng ghi | Mọi transaction mang `base_version`; nếu `base_version` khác `spine_version` hiện tại thì từ chối, **refund credit**, giữ câu trả lời Elicit để sinh lại |
| Trần Regenerate cạn mà Request revision cũng không giải được | Cổng chốt mở thêm **Accept as-is** — ghi `flags[]` cờ vàng kèm lý do, để user không kẹt |

### 4.2 Credit

**Reserve theo lượt gọi model, không theo step.** Một step nhiều lượt; reserve cả step thì hết credit giữa chừng để lại op đã áp mà step chưa xong.

- Reserve trước mỗi lượt → deduct khi thành công → refund khi thất bại hoặc xung đột (UC 9.6)
- `usage[].expires_at` — dòng `reserved` quá hạn bị dọn; đóng tab giữa chừng không để lại credit treo
- Step **tất định** (S-8.2, S-8.3, S-9.1, S-9.5) **không gọi model ⇒ không Meter**. Hết credit không được chặn việc ghi `baselines[]`
- Refund do xung đột **không tiêu trần** Regenerate — user không nên trả giá cho hành vi của session khác
- `usage[state=refunded]` **không** tính vào chi phí báo cáo cho UC 10.3, nhưng **có** tính vào chi phí thật đã trả provider

`[P]` Party Mode là thao tác đắt nhất — cảnh báo credit trước khi chạy.

### 4.3 Ranh giới dữ liệu

| Đi ra ngoài | Không đi ra ngoài |
| --- | --- |
| Projection Spine và hội thoại → API model | Tài liệu user upload — chỉ gửi phần trích xuất |
| — | `.puml` → server PlantUML tự host |

FlintFlow bật cờ opt-out huấn luyện trên mọi API model. Thời gian lưu tài liệu upload: chưa quyết ([mục 10](#10-câu-chưa-chốt)).

### 4.4 Session và resume

`progress` thuộc **project**, không thuộc session. UC 1.10/1.11 cho phép nhiều chat session trên một project, nhưng SRS là một tài liệu, tiến trình một đường:

- Session `is_pipeline = true` (đúng một — bất biến 7): chạy Elicit/Draft/Gate, đẩy `progress`
- Session khác: chỉ phát op sửa (UC 6.2), không đẩy con trỏ
- Session giữ cờ bị xoá → tự promote session hoạt động gần nhất

**Resume ở mức step.** `steps[].first_seq`/`last_seq` khoanh dải `changes[]` của step. Mở lại mà gặp step `in_progress`: revert dải đó bằng `changes[].before`, rồi chạy lại step. Không revert thì làm lại step sẽ áp op lần hai — `add` sinh phần tử trùng.

Câu trả lời Elicit của user được giữ trong transcript và tái dùng khi sinh lại; không hỏi lại từ đầu.

**Sửa Brief muộn.** `project` và mọi field gốc từ Brief nằm trong phạm vi impact query như mọi field khác — lan xuống S-2…S-7 dưới dạng `stale`.

**Đổi `working_mode`** ở ranh giới phase (menu `[C]`), ghi vào `changes[]`.

---

## 5. Product Brief — B-0 → B-2

### 5.1 B-0 Intake — 4 step

| # | Step | Làm gì | UC |
| --- | --- | --- | --- |
| B-0.1 | **Brain Dump** | "Kể hết cho tôi" (UC 2.1) + hỏi tài liệu có sẵn (UC 2.2). Bóc tách trước, tóm tắt cho user xác nhận (UC 2.3) | 2.1, 2.2, 2.3 |
| B-0.2 | **Form-Factor** | Web / mobile / desktop / đa nền / API | *(chưa có UC)* |
| B-0.3 | **Stakes** | Đồ án · nội bộ · gọi vốn · ra mắt thật | *(chưa có UC)* |
| B-0.4 | **Working Mode** | Fast path hay Coaching path | *(chưa có UC)* |

> Hỏi lại thứ user đã viết là lý do người ta bỏ ngang — đó là toàn bộ lý do B-0.1 tồn tại.

**`stakes` + `complexity` → ngưỡng §4.2.2 và §4.2.3.** Không bao giờ bỏ section — template FPT là hợp đồng.

| Nhánh | Cách chạy | Hợp với |
| --- | --- | --- |
| **Fast path** | Gom phần thiếu thành ≤ 2 lượt hỏi mỗi phase, sinh nháp đầy đủ, ghi `assumptions[]`. Cổng chốt gộp **một lần cho cả phase** | "Tuần sau nộp rồi" |
| **Coaching path** | Đi từng step, vặn lại khi câu trả lời mỏng (UC 2.6). Mỗi step ít nhất một lượt hỏi, cổng chốt từng step | "Muốn brief tử tế, không gấp" |

`progress.elicit_turns_this_phase` đếm lượt để thi hành ràng buộc; nó sống qua resume.

### 5.2 B-1 Product Brief — 6 step

| # | Step | AI hỏi | Output |
| --- | --- | --- | --- |
| B-1.1 | **Product Vision, Problem & Opportunity** | Vấn đề gì? Cho ai? Tại sao bây giờ? Vision? Mục tiêu kinh doanh? | Problem statement, vision, business goals |
| B-1.2 | **Target Users & Jobs-to-be-Done** | Ai là user chính? Vai trò nào khác? Mỗi user cần làm gì? | Users, personas, stakeholders, roles, JTBD |
| B-1.3 | **Value Proposition & Differentiation** | Giá trị cốt lõi? Khác biệt? Tại sao user chọn mình? | Value prop, differentiators |
| B-1.4 | **MVP Scope & Feature Hypotheses** | Feature nào MUST HAVE? Ràng buộc? Feature nào KHÔNG làm? | MVP scope, feature inventory, constraints |
| B-1.5 | **Success Metrics & Learning Goals** | Thành công nghĩa là gì? Metrics gì? | Success metrics |
| B-1.6 | **Risks, Assumptions & Open Questions** | Rủi ro lớn nhất? Giả định cần validate? Câu hỏi mở? | Risks, assumptions, open questions |

Cả 6 step chạy trong **UC 2.1**, đào sâu bằng **UC 2.6**.

**Step Brief nuôi section nào** *(gián tiếp qua Spine)*: B-1.1 → §1 + validate ở S-9 · B-1.2 → §2.1, §3.1.3 · B-1.3 → §1 · B-1.4 → §1 release scope, §2.2, §3, §4 · B-1.5 → §4 · B-1.6 → §5.4.

### 5.3 B-2 Brief Finalize — 3 step

| # | Step | Làm gì | UC |
| --- | --- | --- | --- |
| B-2.1 | **Assumption Sweep** | Rà `assumptions[status=unconfirmed]`; Coaching duyệt lẻ, Fast duyệt theo lô | 2.9 |
| B-2.2 | **Addendum Triage** | Entry nào vào Brief, entry nào để dành cho SRS, entry nào bỏ | *(chưa có UC)* |
| B-2.3 | **Three-Lens Review** | Ba lens do FlintFlow tự định nghĩa: Skeptic / Opportunity / Contextual → **UC 2.5 Approve** → mở S-1 | 2.5 |

Về B-2.1:

- **Fast** nhóm giả định theo `target_section`, một nút "Accept cả nhóm"
- Vẫn bắt duyệt lẻ những giả định chạm bất biến hoặc nuôi §4.2.2/§4.2.3
- Fast path cố ý sinh nhiều giả định để né hỏi; duyệt lẻ từng cái sẽ đưa lại đúng số câu hỏi vừa né

### 5.4 Addendum

User thường nói ra nhiều thứ **không thuộc Brief** nhưng SRS cần: persona chi tiết, ràng buộc kỹ thuật, phương án đã loại, số liệu quy mô, quy định ngành. **AI ghi ngay** trong lúc hội thoại. Mỗi entry có `target_section`. Ở S-5 / S-6 / S-7, Intake đọc addendum trước khi hỏi.

---

## 6. SRS Generation — S-1 → S-9

### 6.1 Chín phase

| Phase | Hình thái | Input từ Brief | Spine ghi ra |
| --- | --- | --- | --- |
| **S-1** Analyze & Validate Brief | **Mềm** | Toàn bộ B-1 + addendum | `project` |
| **S-2** Product Overview | Cố định | B-1.1, B-1.3, B-1.4 | `release_scope`, `actors[kind≠human]`, `business_rules[tier=high]` |
| **S-3** User Requirements | Cố định | B-1.2, B-1.4 | `actors[]`, `roles[]`, `use_cases[]` |
| **S-4** System Functional Overview | Cố định | B-1.4, B-1.2, B-1.5 | `features[]`, `screens[]`, `permissions[]`, `entities[]`, `functions[]` (khung) |
| **S-5** Feature & Function Details | **Loop theo màn hình** | B-1.2, B-1.4, B-1.5, S-4, addendum | `functions[].*` |
| **S-6** Non-Functional Requirements | Cố định | B-1.4, B-1.5, B-1.6, addendum | `nfrs[]` |
| **S-7** Requirement Appendix | Cố định — **suy ra là chính** | B-1.6, addendum | `business_rules[tier=detail]`, `common_requirements[]`, `messages[]`, `other_requirements[]` |
| **S-8** Glossary & Assemble | Cố định | Section S-2 → S-7 đã `accepted` | `glossary[]` |
| **S-9** Verify, Validate & Baseline | **Pha gate cuối** | B-1.1, B-1.5 | `flags[]`, `baselines[]`, `priority` |

### 6.2 S-5 lặp theo màn hình, không theo function

Mỗi màn CRUD có 4–6 function, cộng non-screen function. Lặp theo function thì dự án 19 màn ra 60–100 vòng — không ai đi hết, và hoá đơn model nổ.

**Một vòng S-5 = một màn hình.** `progress.screen_queue[]` chốt thứ tự ở S-4.1; `screens[].detail_status` cho điều kiện thoát:

```text
còn màn có detail_status = pending  →  lấy màn đầu queue, tiếp tục
hết  →  một vòng S-5 cuối cùng gộp toàn bộ non-screen function
```

Màn `placeholder` **không** tính là `pending` — nó đã được quyết định để lại (mục 9.1).

**Chia lô ≤ 6 function mỗi lượt gọi model.** Màn lớn (Project Workspace: chat pane + document pane + verification panel) có thể 15–20 function — sinh một lượt thì vượt giới hạn output và tụt chất lượng về cuối. Chia lô không đổi công thức `5 × N` vì đây là chia lượt gọi, không chia step.

Vòng non-screen function dùng id step `S-5.<n>@nonscreen`.

### 6.3 Đánh số §3.2, §3.3, §3.4 …

```text
§3.(2 + features[].order)                    feature thứ order (từ 0)
§3.(2 + order).(functions[].order + 1)       function trong feature đó
```

`functions[].order` đánh riêng cho `screen_id ≠ null` và `screen_id = null`, nên non-screen function không làm thủng dãy số của §3.x.y.

Bất biến 5 giữ `order` duy nhất và liên tục — xoá hoặc hoán vị feature phải đi qua transaction có op renumber (`srs-spine.md` §3).

Spine **không lưu số hiệu section** — `addendum[].target_section` và `flags[].section_id` lưu khoá logic (`feature:F2`). Mọi tham chiếu chéo trong văn xuôi render từ khoá logic lúc assemble; **cấm** field prose ghi cứng `"3.4"`.

Wireframe của màn gắn vào `screens[].primary_function_id` — hình nằm trong §3.x.y của function đó, function khác cùng màn tham chiếu tới.

### 6.4 Toàn bộ step — nguồn sự thật

**Brief — 13 step.** Chi tiết ở [mục 5](#5-product-brief--b-0--b-2).

| # | Step | UC |
| --- | --- | --- |
| B-0.1 | Brain Dump | 2.1, 2.2, 2.3 |
| B-0.2 | Form-Factor | *(chưa có UC)* |
| B-0.3 | Stakes | *(chưa có UC)* |
| B-0.4 | Working Mode | *(chưa có UC)* |
| B-1.1 | Product Vision, Problem & Opportunity | 2.1 |
| B-1.2 | Target Users & Jobs-to-be-Done | 2.1, 2.6 |
| B-1.3 | Value Proposition & Differentiation | 2.1 |
| B-1.4 | MVP Scope & Feature Hypotheses | 2.1, 2.6 |
| B-1.5 | Success Metrics & Learning Goals | 2.1 |
| B-1.6 | Risks, Assumptions & Open Questions | 2.1 |
| B-2.1 | Assumption Sweep | 2.9 |
| B-2.2 | Addendum Triage | *(chưa có UC)* |
| B-2.3 | Three-Lens Review | 2.5 |

**SRS — 38 step cố định + 5 × N.**

| # | Step | Làm gì | Ra SRS | UC |
| --- | --- | --- | --- | --- |
| S-1.1 | Brief Extraction | B-1 + addendum → dữ liệu cấu trúc | — | 2.3 |
| S-1.2 | Project Classification | type / domain / complexity | — | *(chưa có UC)* |
| S-1.3 | Conflict & Assumption Review | trình `assumptions[]` và xung đột cho user chốt | — | 2.4, 2.9 |
| S-1.4 | Gap List | cái gì SRS cần mà Brief chưa có | — | 2.6 |
| S-2.1 | Product Overview | từ B-1.1 + B-1.3: vấn đề, giải pháp, giá trị | §1 | 3.1 |
| S-2.2 | Release 1.0 Scope | in / out → `release_scope` | §1 | 3.1 |
| S-2.3 | External Systems | hệ thống ngoài → `actors[kind=system]` | §1 | 3.1 |
| S-2.4 | High-Level Business Rules | nguyên tắc nghiệp vụ chung → `tier=high` | §1 | 3.1 |
| S-2.5 | System Context Diagram | render `diagrams[context]` | §1 | 3.1 |
| S-3.1 | Actors | người dùng + hệ thống ngoài + actor thời gian; `roles[].actor_id` | §2.1 | 3.2 |
| S-3.2 | Actor–Goal List | mỗi actor cần đạt gì → UC ứng viên, tên động từ + tân ngữ | §2.2 | 3.2 |
| S-3.3 | Missing Use Case Sweep | admin, support, thông báo, quên mật khẩu, audit log, xử lý lỗi | §2.2 | 3.2 |
| S-3.4 | Use Case Relationships | `«include»` / `«extend»` | §2.2 | 3.2 |
| S-3.5 | Use Case Descriptions | bảng §2.2.2: ID, tên, actor, mô tả | §2.2.2 | 3.2 |
| S-3.6 | Use Case Diagram | render `diagrams[usecase]`, tách hình nếu quá lớn | §2.2.1 | 3.2 |
| S-4.1 | Screen Inventory | `features[]` + `screens[]` theo feature — **chốt N và `screen_queue[]`** | §3.1.2 | 4.1 |
| S-4.2 | Screens Flow | điều hướng; ký hiệu riêng cho pop-up và màn nhiều tab | §3.1.1 | 4.1 |
| S-4.3 | Screen Authorization | ma trận màn × vai trò, dòng con cho từng hành động | §3.1.3 | 4.1 |
| S-4.4 | Non-Screen Functions | cron, webhook, background engine | §3.1.4 | 4.2 |
| S-4.5 | Entity Relationship Diagram | thực thể, mô tả, quan hệ → `diagrams[erd]` | §3.1.5 | 4.2 |
| S-5.1 | Screen Queue | lấy màn `pending` đầu queue | — | *(chưa có UC)* |
| S-5.2 | Trigger & Description | cách kích hoạt, actor, mục đích, giao diện, xử lý dữ liệu | §3.x.y | 4.3 |
| S-5.3 | Screen Layout | render `diagrams[screen_layout]` bằng salt — chỉ màn cốt lõi ([7.2](#72-wireframe-chia-tầng)) | §3.x.y | 4.3 |
| S-5.4 | Function Details | luồng thường / bất thường, validation (có `id`) | §3.x.y | 4.3 |
| S-5.5 | Screen Sign-off | Accept → `detail_status = signed_off` → quay lại S-5.1 | — | 4.3 |
| S-6.1 | External Interfaces | giao tiếp người dùng, phần cứng, hệ thống ngoài | §4.1 | 5.1 |
| S-6.2 | Usability | thời gian học, thời gian thao tác, chuẩn giao diện | §4.2.1 | 5.1 |
| S-6.3 | Reliability | availability %, MTBF, MTTR — **phải có số** | §4.2.2 | 5.1 |
| S-6.4 | Performance | response time, throughput, capacity — **phải có số** | §4.2.3 | 5.1 |
| S-6.5 | Domain-Specific Attributes | bảo mật, quyền riêng tư, tuân thủ | §4.2.4 | 5.1 |
| S-7.1 | Business Rules | suy từ `validations[kind=business]` → `tier=detail` | §5.1 | 5.2 |
| S-7.2 | Common Requirements | phân trang, định dạng ngày, thông báo lỗi chung | §5.2 | 5.2 |
| S-7.3 | Application Messages | suy từ `abnormal[]` → danh sách mã | §5.3 | 5.2 |
| S-7.4 | Other Requirements | từ B-1.6 + Technical Risk Analysis | §5.4 | 5.2 |
| S-8.1 | Glossary | quét Spine lấy thuật ngữ + viết tắt → user chốt | §5.5 | 5.3 |
| S-8.2 | Document Assembly | ghép theo thứ tự FPT, sinh số hiệu section ([6.3](#63-đánh-số-32-33-34-)), chèn ảnh | toàn văn | 5.4 |
| S-8.3 | Record of Changes | dựng §I từ `changes[]` | §I | *(chưa có UC)* |
| S-8.4 | Consistency Pass | **tất định**: toàn vẹn tham chiếu. **LLM**: trùng lặp ngữ nghĩa, thuật ngữ lệch | toàn văn | 5.4 |
| S-9.1 | Completeness & Assumption Sweep | section nào thiếu/mỏng; rà `assumptions[]` sinh trong SRS; chạy lại `glossary[]` nếu lệch version | — | 6.4, 2.9 |
| S-9.2 | Quality Lens Run | FR↔NFR mâu thuẫn, rủi ro bảo mật, câu chữ mơ hồ → **cờ vàng** | — | 6.9, 6.10 |
| S-9.3 | Business Goal Validation | đối chiếu SRS với business goals ở B-1.1 | — | 6.5 |
| S-9.4 | Requirement Prioritization | Must / Should / Could / Won't | — | 6.6, 6.7 |
| S-9.5 | Baseline Sign-off | **quét lại toàn bộ** `deterministic-check` trên `spine_version` hiện tại → cờ đỏ = 0 → ghi `baselines[]` | — | 6.14 |

**Đếm:** Brief 13 · SRS 38 cố định + 5 × N = **51 step cố định + 5 × N**.
N = 1 → 56 · N = 20 (19 màn + 1 vòng non-screen) → 151.

### 6.5 Pha gate cuối và Export

| | Quy tắc |
| --- | --- |
| **Điều kiện baseline** | Số cờ đỏ có `resolved_at = null` **và** `waived_by_user = false` phải bằng **0** |
| **Quét lại trước khi ký** | S-9.5 chạy lại toàn bộ `deterministic-check` trên `spine_version` hiện tại, lưu `checked_at_version`. Nếu `spine_version` đổi giữa lúc quét và lúc ký → từ chối, quét lại |
| **Van xả** | Ba luật `dead_reference`, `array_empty`, `render_error` **không waive được** — chúng là vi phạm bất biến, không phải giới hạn phạm vi (`srs-spine.md` §7) |
| **Cờ vàng** | Không chặn; xử lý qua UC 6.10 |
| **Không đặt ngưỡng phần trăm** | User sẽ bịa nội dung cho đủ điểm. Gate bằng danh sách cờ đỏ có tên |
| **`waived_count > 0`** | Version là `v1.0-conditional`, không phải `v1.0` |

**Export** (UC 8.1–8.3) — **không bị chặn bởi chất lượng tài liệu, sau khi đã có Working Draft**:

| Nguồn | File ra |
| --- | --- |
| Working Draft | Watermark **DRAFT** mọi trang · §I in **danh sách cờ đỏ đang mở, số section `stale`, mọi waive** · tên file `-draft` |
| Baseline | Render từ `baselines[].snapshot_ref`, **không** từ Spine hiện tại · bản sạch đóng dấu version |

Trước S-8.2 chưa có bản ghép — nút export hiển thị lý do và đường tắt tới S-8.2. Section `stale` hoặc `awaiting_reaccept` được đánh dấu **tại chỗ** trong file, không chỉ có watermark toàn trang. Giới hạn gói (Pro/Free) là ràng buộc **thương mại**, độc lập với chất lượng.

**Điểm sẵn sàng** hiển thị như phản hồi, không phải điều kiện chốt: `72% accepted · 4 section chờ duyệt lại · 2 cờ đỏ`. Công thức ở `srs-spine.md` §5.

---

## 7. Diagram

### 7.1 Một đường duy nhất

**agent xuất `.puml` → ghi `diagrams[]` + `source_hash` → compile-check → server render → nhúng khi export.**

| SRS section | `kind` | `owner_kind` | PlantUML | File renderer |
| --- | --- | --- | --- | --- |
| §1 Context Diagram | `context` | `null` | component / rectangle | `render-context-diagram.md` |
| §2.2.1 Use Case Diagram | `usecase` | `null` | `usecase` (actor, `«include»`, `«extend»`) | `render-usecase-diagram.md` |
| §3.1.1 Screens Flow | `screen_flow` | `null` | `state` (composite state cho màn có tab, note cho pop-up) | `render-screen-flow.md` |
| §3.1.5 ERD | `erd` | `null` | `entity` + crow's foot | `render-erd.md` |
| §3.x.y Screen Layout | `screen_layout` | `screen` | `salt` | `render-screen-layout.md` |

`source_fields` cho từng `kind`: `srs-spine.md` §7.1.

**Không có Sequence Diagram** — template FPT §3.2.1 không yêu cầu, và là loại sơ đồ model sinh sai nhiều nhất. Đã hoãn Phase 2 trong `usecase.md` UC 4.3.

**Render chạy phía server** vì hai lý do. Một: web preview và export dùng chung một pipeline, nên hình trong file xuất giống hệt hình trên màn hình. Hai: dữ liệu khách hàng không đi qua server công cộng. Self-host `plantuml/plantuml-server` + biến `PLANTUML_BASE_URL`.

### 7.2 Wireframe chia tầng

- **Màn cốt lõi** (Project Workspace, Verification & Change Panel, Export, Project Dashboard, Plan & Pricing, Notification Center): Salt đầy đủ.
- **Màn CRUD lặp lại** (~10 màn Admin, giống nhau ~90%): một Salt mẫu + bảng biến thể.

Salt là **low-fi có chủ đích** — SRS nói *màn hình có gì và làm gì*, không nói *nó trông ra sao*.

---

## 8. Bản đồ file .md

File `.md` là **prompt asset agent đọc lúc chạy**. Tổ chức theo BMAD: `SKILL.md` (luôn nạp, ≤ ~150 dòng) + `references/*.md` (nạp có điều kiện) + `assets/*-template.md` (nạp lúc render).

**Nguồn sự thật:** repo. Vòng một không có DB override — `usecase.md` nhóm 10 không có UC quản lý prompt/document template. Vẫn ghi `sections[].asset_version` để sau này thêm được; khi bật DB override thì so version lúc mở project, không phải lúc admin lưu.

> Bỏ **cơ chế** đọc `customize.toml` của BMAD (cần `uv`). Ý tưởng override phân tầng để dành khi có UC.

### 8.1 Tái dùng từ BMAD (MIT — giữ copyright notice)

| File | Lấy gì |
| --- | --- |
| `bmm-skills/plan/bmad-prd/SKILL.md` | State machine: Conventions, Discovery, Concern scan, Reviewer Gate, Finalize |
| `bmad-product-brief/SKILL.md` | Discovery: brain dump → form-factor → stakes → Fast/Coaching. Nguồn của B-0 |
| `bmad-prd/assets/headless-schemas.md` + tương tự ở `bmad-spec`, `bmad-ux` | Schema chạy không cần người — nền cho `srs-spine.schema.json` |
| `bmad-prd/references/headless.md` | Chạy workflow headless + override |
| `bmad-prd/assets/prd-template.md` | Pattern "Essential Spine vs Adapt-In Menu" |
| `bmad-product-brief/assets/brief-template.md` | Cấu trúc Brief |
| `bmad-ux/SKILL.md`, `bmad-ux/assets/key-screens.md` | IA / screens / states; nguyên tắc key screens |
| `bmad-architecture/assets/spine-template.md` | Pattern spine document |
| `core-skills/bmad-advanced-elicitation/SKILL.md` | `[A]` |
| `core-skills/bmad-party-mode/SKILL.md` + `references/mode-subagent.md` | `[P]` |
| `core-skills/bmad-review/references/lens-*.md` (5 file) | Pattern lens; nguồn cờ vàng cho S-9.2 |
| `bmad-prd/assets/prd-validation-checklist.md`, `references/validate.md` | Rubric + báo cáo HTML cho S-9 |
| `bmad-sprint-planning/references/readiness-gate.md` | Soft gate — khớp [6.5](#65-pha-gate-cuối-và-export) |

> Ba lens của B-2.3 (Skeptic / Opportunity / Contextual) là **do FlintFlow tự định nghĩa**, không có trong BMAD.

### 8.2 Nhóm tự viết

**A. Skill hành động — dùng chung mọi phase (10 file)**

| File | Việc |
| --- | --- |
| `srs-orchestrator.md` | State machine toàn cục; giữ `progress`, `working_mode`; điều phối B-0 → S-9 |
| `phase-intake.md` | Đọc projection Spine, liệt kê field trống, đọc `addendum[]` liên quan |
| `elicit-loop.md` | Hỏi phần thiếu; hai nhánh Fast / Coaching; đếm `elicit_turns_this_phase` |
| `draft-to-ops.md` | Câu trả lời → transaction op; validate schema; kiểm bất biến cuối lô |
| `apply-change-op.md` | Impact query → 3 nhánh → `stale`; hoà giải; cascade xoá |
| `deterministic-check.md` | 10 luật cờ đỏ + 6 luật cardinality (`srs-spine.md` §7, §8.1) |
| `review-section.md` | Lens LLM → cờ vàng |
| `gate-check.md` | Accept / Request revision / Regenerate / Accept as-is; trần 3 và 8; menu `[A]/[P]/[C]` |
| `meter.md` | Reserve / deduct / refund theo lượt gọi, ghi `usage[]` |
| `plantuml-conventions.md` | Quy ước chung 5 renderer |

**B. Skill nội dung (12 file)**

`project-classifier` (S-1) · `product-overview` (S-2) · `high-level-rules` (S-2) · `actors-and-usecases` (S-3 — **gap lớn nhất**, BMAD không có UML use case) · `screens-and-flow` (S-4) · `authorization-matrix` (S-4) · `entities-erd` (S-4) · `non-screen-functions` (S-4) · `function-detail` (S-5) · `nfr-quality-attributes` (S-6) · `appendix-content` (S-7) · `glossary` (S-8)

**C. Renderer (5 file)** — xem bảng [7.1](#71-một-đường-duy-nhất)

**D. Đầu ra cuối (2 file)** — `assemble-srs.md` (S-8) · `srs-completeness-score.md` (S-9)

Tổng **29 file**.

---

## 9. Phạm vi vòng một

### 9.1 Cắt theo độ sâu, không cắt section

**Không cắt renderer nào.** Cả 5 loại hình đều thuộc section bắt buộc — cắt bất kỳ cái nào làm section rỗng → cờ đỏ → không baseline. Renderer lại là phần rẻ nhất.

| Giữ | Cắt | UC bị ảnh hưởng |
| --- | --- | --- |
| Cả 9 phase, đủ 5 phần template FPT | S-5 chỉ 3–5 màn cốt lõi, còn lại `detail_status = placeholder` | 4.3 (một phần) |
| Cả 5 renderer | — | — |
| **Hoà giải thủ công một lượt** | Hoà giải **tự động** (lan tỏa không cần bấm) | — |
| **Undo một op cuối** (UC 6.13) | — | — |
| **Traceability map tối giản** (UC 6.3) — đọc thẳng bảng `srs-spine.md` §4 | — | — |
| Cổng chốt + Accept as-is | `[A]`, `[P]` | — |
| Cờ đỏ tất định + cardinality + van xả | Cờ vàng LLM | **6.9, 6.10** |
| Export Word + DRAFT watermark | Export PDF, Handoff Summary | **8.2, 8.3** |
| — | Share / clone / revoke | **1.13, 1.14, 1.15, 1.19** |

Màn `placeholder` vẫn sinh đủ khung `functions[]` để §3.x.y không rỗng. Cờ `screen_pending_at_baseline` không bắn cho màn `placeholder`; phần chưa chi tiết dùng van xả.

> **Undo và traceability map được giữ** vì mục 2.1 tự nói đồ thị khoá đã trả chi phí cho chúng: `changes[].before` làm undo là một câu lệnh, và traceability map là truy vấn read-only trên bảng đã có. Cắt chúng là bỏ đúng thứ tài liệu tuyên bố là lý do kiến trúc này tồn tại.

### 9.2 Ngoài pipeline — 33/71 UC

| | Người | Làm |
| --- | :---: | --- |
| **Pipeline** | 3 | Spine engine · 29 file skill · 5 renderer · assemble/export |
| **Nền tảng** | 2 | Auth · project CRUD · chat session · credit ledger · admin đọc · notification |

| Nhóm | Vòng một | Hoãn |
| --- | --- | --- |
| **1** Workspace (19 UC) | 1.1–1.9 auth + project CRUD · 1.10–1.11 chat session · 1.12 onboarding | 1.13–1.15, 1.19 share/clone · 1.16–1.18 |
| **9** Billing (6 UC) | 9.1, 9.2 · **9.6 credit ledger** · 9.3/9.4 dùng **mock / sandbox gateway** | 9.5 |
| **10** Admin (7 UC) | 10.1, 10.2, 10.3 — chỉ đọc | 10.4–10.7 |
| **11** Notification (2 UC) | 11.1 in-app | 11.2 · email |

≈ **15/33 UC** ở vòng một. §3.1.4 của chính SRS FlintFlow liệt kê 6 non-screen function (cron reset quota, subscription expiry, payment webhook, proactive completeness evaluation, cost aggregator, health monitor) — toàn backend thật, bằng chứng 2 người là mức sàn.

### 9.3 Thứ tự làm — dọc, không ngang

```text
Bước 0.  Dựng FIXTURE: một Spine seed đầy đủ cho project mẫu 19 màn (viết tay,
         không qua AI). Dùng cho cả hai phép đo dưới đây và cho bước 2, 3.
         a) Đo token end-to-end trên fixture 19 màn — không chỉ SRS nhỏ
         b) 10 ca thử "model có sinh op đúng path/schema không"
         ↑ HAI RỦI RO GỐC. Sai một trong hai = làm lại kiến trúc giữa kỳ.

1.  srs-spine.schema.json + op engine (transaction, cascade, rollback)
    + deterministic-check
2.  S-3 end-to-end, chạy trên fixture seed (không cần B-0…B-2 chạy trước)
    ↑ gap BMAD không phủ; chứng minh phần AI khó; ra hình demo được
3.  S-8.2 assemble --partial (bỏ qua section_empty, chỉ môi trường dev,
    không ghi baselines[]) + export Word
    ↑ có MỘT luồng đi trọn: fixture → §2 → file Word. Demo được từ đây.
4.  B-0 → B-2 (13 step chat) — làm sau, vì không ra hình và không demo được sớm
5.  Mở rộng S-2, S-4, S-5, S-6, S-7 — rẻ, vì 6 skill hành động đã xong
```

Tệ nhất là tuần cuối có 12 phase mỗi cái chạy 40% và không luồng nào đi trọn.

---

## 10. Câu chưa chốt

### 10.1 Step chưa có use case

| Step | Cần UC cho |
| --- | --- |
| B-0.2 – B-0.4 | Form-factor · stakes · chọn Fast/Coaching |
| B-2.2 | Ghi và tra `addendum[]` |
| S-1.2 | Phân loại project type / domain / complexity |
| S-5.1 | Hàng đợi màn hình + con trỏ resume |
| S-8.3 | Dựng §I Record of Changes |

*(Mục này chỉ gom lại các đánh dấu `(chưa có UC)` rải trong tài liệu — không có thông tin mới.)*

### 10.2 Cơ chế chưa có UC

| Cơ chế | Ghi chú |
| --- | --- |
| Resume dự án dở dang | Nhóm 1 có 1.11 Switch Chat Session nhưng không có UC khôi phục vị trí phase/step |
| Chọn working mode | Không UC nào mô tả Fast vs Coaching |
| Quản lý prompt / document template | Nhóm 10 không còn UC này → đã bỏ khỏi vòng một |

### 10.3 Khác

- **Ngưỡng mở inline edit** cho ô văn xuôi lá: đo tỉ lệ lượt chat chỉ để sửa câu chữ qua `usage[]`; vượt ~30% thì xem xét lại.
- Ba câu còn treo về mô hình dữ liệu: `srs-spine.md` §10.
