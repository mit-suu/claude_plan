# FlintFlow — Business Flow

> Tài liệu giải thích FlintFlow chạy thế nào, kèm một ví dụ xuyên suốt. Trạng thái: **đề xuất** (13/09/2026) — chờ nhóm chốt [mục 9](#9-cần-chốt).
> Theo quy ước Phase — Step — Section của [`Product-Brief-to-SRS-Phases.md`](../Product-Brief-to-SRS-Phases.md) §1; đổi gì so với quy ước ghi ở [mục 3](#3-thay-đổi-quy-ước). Bối cảnh scope, rủi ro chi tiết, công nghệ: [`scope-revision-options.md`](scope-revision-options.md).

Từ **"mục"** chỉ dùng cho mục của tài liệu này; phần của SRS gọi là **section**.

## 1. Phương án: một engine, ba chế độ

| Chế độ | Người dùng có gì | Phase chạy | Kết quả |
| --- | --- | --- | --- |
| **1. Sửa SRS có sẵn** (gồm cả "chỉ kiểm tra đạt chưa") | File SRS .docx + yêu cầu thay đổi | **I-\*** Import → **C-\*** Change Request | Chính file đó, chỗ sửa hiện Track Changes |
| **2. Tạo mới theo mẫu FPT** (đề xuất khi khách không có mẫu) | Ý tưởng / dữ liệu thô | **B-\*** → **S-\*** đủ step | File SRS mới điền vào mẫu FPT |
| **3. Tạo mới theo mẫu user cung cấp** | Ý tưởng / dữ liệu thô + file mẫu | **B-\*** → **S-\*** đã lọc theo profile | File SRS mới điền vào chính file mẫu |

Sau khi chốt baseline, **cả 3 chế độ đều quy về C-\***.

```text
                 ┌──────────────── ENGINE (dùng chung) ────────────────┐
                 │ Spine: field + liên kết + neo trỏ vào block trong file │
                 │ Step: skill + khai field ĐỌC / field GHI               │
                 │ Kiểm tra: tầng 2–3 trên Spine, tầng 1 trên profile     │
                 │ Writer: một bộ thao tác cho cả sửa tại chỗ và điền mẫu │
                 └───────────────────────────────────────────────────────┘
                                        ▲
     TEMPLATE PROFILE (mỗi mẫu một cái): field → section · cột bảng → field
                                         · section bắt buộc · ngôn ngữ
     FPT = profile đầu tiên, giữ nguyên ID fixed:3.1.3…
```

### 1.1 Các phần liên hệ thế nào

Hình dung như nhà hàng: **Spine** là nguyên liệu, **Step** là đầu bếp, **Profile** là cách bày món theo từng nhà hàng, **Writer** là người bưng món ra.

```text
Step ──ghi──► Spine ──(Profile: field nằm ở section nào)──► Writer ──► File .docx
```

| Phần | Làm gì | Ví dụ |
| --- | --- | --- |
| **Step** | Soạn nội dung, ghi vào Spine. Không đụng tới file | S-3.5 ghi điều kiện của UC-03 |
| **Spine** | Giữ nội dung, không phụ thuộc mẫu | `UC-03.preconditions = "có văn bản PCCC"` |
| **Profile** | Nói nội dung đó nằm ở đâu trong mẫu | Mẫu Sở: cột "Điều kiện", Chương 4 |
| **Writer** | Ghi vào đúng chỗ trong file | Chèn Track Changes vào ô đó |

**Đổi mẫu thì chỉ đổi Profile**, mọi phần khác giữ nguyên.

**Không có tầng "loại section".** Tầng trung lập giữa các mẫu chính là **Spine field** — đúng quy ước sẵn có `Step → Spine field → Section`. Hỗ trợ mẫu mới = thêm một profile; bảng ánh xạ ở `srs-spine.md` §4 chính là profile FPT.

**Nguyên tắc:**
1. Dự án nào cũng có baseline → sau baseline mọi thay đổi đi qua C-\*.
2. **AI chỉ đề xuất → Code kiểm → Người duyệt → Code ghi.**
3. **Chỉ duyệt trong app.** Track Changes trong Word là để trình bày cho người ngoài.
4. Block không có field Spine nào khớp → **chỉ đọc, không ghi**.
5. **Không tích hợp SharePoint / Google Drive** — người dùng tự tải file lên, tải file về (mục 6).

---

## 2. Khái niệm

| Khái niệm | Là gì | Ví dụ |
| --- | --- | --- |
| **Phase** | Milestone; đơn vị đào sâu `[A]/[P]/[C]` | `B-0`…`B-2`, `S-1`…`S-9`, **`I-1`…`I-4`**, **`C-1`…`C-7`** |
| **Step** | Đơn vị chốt; có skill, khai field đọc / field ghi | `S-3.5` Use Case Descriptions ghi `use_cases[]` |
| **Section** | Phần của SRS **theo mẫu đang dùng**; ID do profile định nghĩa | FPT: `fixed:2.2.2` · Sở XD: `soxd:ch4` |
| **Template profile** | Cấu hình của một mẫu: field → section, cột bảng → field, section bắt buộc, ngôn ngữ | Profile FPT, profile Sở XD |
| **Spine** | Dữ liệu có cấu trúc của SRS + liên kết + neo | `functions[F-S04-01].validations[]` → `use_cases[UC-03]` |
| **Block / neo** | Một đoạn / bảng / hàng / ô trong file Word và mã định danh | Hàng 3 bảng `soxd:ch4`, `paraId=3F2A` |
| **Change Request (CR)** | Đơn vị công việc sau baseline: có nguồn, người đề xuất, người duyệt | CR-012 từ email 10/09 |
| **Baseline** | Bản SRS đã chốt, có version. SRS import vào = **v0 (imported)** | v0, v1, v2.4 |

**Vai trò:**

| Vai trò | Ví dụ | Tạo CR / đưa dữ liệu | Ra lệnh AI | Gate Accept khi soạn mới | Duyệt CR, chốt baseline |
| --- | --- | :-: | :-: | :-: | :-: |
| **Lead** | BA Lead | ✔ | ✔ | ✔ | ✔ |
| **Analyst** | BA, BA junior | ✔ | ✔ | ✔ | — (gửi Lead) |
| **Viewer** | PO, Dev, QA, khách | comment | — | — | — |

---

## 3. Thay đổi quy ước

| Thứ | Quy ước hiện tại (Phases §1) | Đề xuất |
| --- | --- | --- |
| Section | "Mục trong SRS theo template FPT", ID `fixed:3.1.3` | "Section của **mẫu đang dùng**", ID do profile định nghĩa. **Profile FPT giữ nguyên `fixed:3.1.3`** — không đổi tên gì |
| Step | Ghi field | Khai thêm **field đọc** → suy ra phụ thuộc giữa step |
| Phase | `B-0`…`B-2`, `S-1`…`S-9` | Thêm **`I-1`…`I-4`** (Import), **`C-1`…`C-7`** (Change Request) |
| Ngôn ngữ (Phases §1.3) | Nội dung render vào SRS: tiếng Anh | **Theo ngôn ngữ của profile**. FPT = tiếng Anh; mẫu Sở XD = tiếng Việt |
| Thanh tiến độ | Đếm step | Chế độ 2, 3 đếm step. Chế độ 1 hiện **trạng thái CR** (nháp → đang duyệt → đã ghi) |
| Gate | Accept/Revise mỗi step | Soạn mới: mỗi step. C-\*: **Lead duyệt theo nhóm thay đổi** ở C-6 |
| S-8.2 Document Assembly | Ghép file theo thứ tự FPT | **Writer điền vào file mẫu** của profile — FPT cũng là một file mẫu |
| Số step | 51 cố định + 5 × N | Chế độ 2 giữ nguyên; chế độ 3 ít hơn, tuỳ profile |

**Step không viết section** — vẫn giữ. Step ghi field vào Spine; writer đưa field vào section theo profile:

```text
Step (S-x.y) ──ghi op──► Spine field ──profile──► Section trong file
```

---

## 4. Phase mới

### 4.1 I-\* Import

| Step | Làm gì | Dùng lại |
| --- | --- | --- |
| **I-1** Preflight | Định dạng .docx · còn Track Changes / comment cũ → không nhận · mật khẩu · **dấu version** FlintFlow trong file (nếu có) có phải bản mới nhất | — |
| **I-2** Parse & Anchor | Tách block, gắn neo `w14:paraId` hoặc vị trí + hash | — |
| **I-3** Profile Match | Chọn profile có sẵn, hoặc khớp mới: heading → section, cột bảng → field. Người xác nhận chỗ độ tin thấp | — |
| **I-4** Spine Extraction | AI trích field theo profile (có độ tin) → **baseline v0** → chạy kiểm tra ra gap | S-9.1, S-9.2 |

### 4.2 C-\* Change Request

| Step | Làm gì | Dùng lại |
| --- | --- | --- |
| **C-1** Intake | Nguồn bắt buộc (email / biên bản / gap / "yêu cầu miệng từ <ai>"), người yêu cầu | — |
| **C-2** Clarify | AI hỏi lại nếu mơ hồ | UC 6.11 |
| **C-3** Impact | Đồ thị Spine + tìm từ khoá trong block → danh sách field và block bị ảnh hưởng | UC 6.8 |
| **C-4** Propose | Với mỗi field bị ảnh hưởng, gọi **skill của step sở hữu field đó**; khoá block | Skill S-3.5, S-5.4, S-6.x… |
| **C-5** Verify | Code kiểm chữ cũ khớp đúng block · luật Spine · Consistency Pass trên phạm vi bị đổi | S-8.4, S-9.2 |
| **C-6** Approve | Analyst gửi → Lead duyệt / từ chối theo nhóm thay đổi, lưu lý do | Gate, UC 6.14 |
| **C-7** Write | Writer ghi Track Changes (tác giả = mã CR) → version mới → mở khoá block | S-8.3 |

Step **sở hữu** field = step ghi field đó trong bảng Phases §6.4 (vd. `use_cases[]` → S-3.5; `functions[].validations[]` → S-5.4; `business_rules[tier=detail]` → S-7.1).

### 4.3 Phase cũ dùng thế nào ở mỗi chế độ

| Phase | 1. Sửa có sẵn | 2. Tạo mới FPT | 3. Tạo mới mẫu họ |
| --- | --- | --- | --- |
| B-0 → B-2 | ✗ | ✔ | ✔ |
| S-1 | ✗ (S-1.3 dùng lại khi CR mâu thuẫn) | ✔ | ✔ |
| S-2 → S-7 | Không chạy tuần tự — C-4 gọi skill của step sở hữu field | ✔ đủ | ✔ đã lọc (mục 5) |
| S-8.1 Glossary | Chỉ khi CR thêm thuật ngữ | ✔ | Nếu profile có section glossary |
| S-8.2 Assembly | ✗ (sửa tại chỗ) | Writer điền mẫu FPT | Writer điền mẫu họ |
| S-8.3 Record of Changes | Sinh từ CR | ✔ | Nếu profile có section tương ứng |
| S-8.4 Consistency Pass | Trong C-5 | ✔ | ✔ |
| S-9 | Sau I-4 và trước baseline mới | ✔ | ✔ |

---

## 5. Lọc step cho chế độ 3

1. Chọn step có field được **render vào một section có trong profile**.
2. Thêm step có field mà **step đã chọn đọc tới** — lặp đến khi không thêm được nữa. Step thêm vì lý do này chạy nhưng **chỉ lưu vào Spine**, trừ khi field của nó cũng được render.
3. Step **không ra section nào** (B-\*, S-1.x, S-5.1, S-5.5, S-8.4, S-9.x) **luôn chạy**.
4. Thứ tự giữ theo ID step (Phases §6.4), **không** theo thứ tự section của mẫu.
5. Section **bắt buộc** trong profile mà không có field nào render vào → để trống, **cờ đỏ "cần viết tay"**.

**Kiểm tra 3 tầng** — "tài liệu đạt chưa?":

| Tầng | Kiểm gì | So với | Không đạt |
| --- | --- | --- | --- |
| 1. Hình thức | Đủ section bắt buộc, đúng bảng | Profile đang dùng | Cờ đỏ |
| 2. Nội dung | Spine có đủ loại field SRS cần có (phạm vi, actor, yêu cầu chức năng, NFR…) | ISO/IEC/IEEE 29148 | Cờ vàng |
| 3. Chất lượng yêu cầu | Mơ hồ, không kiểm thử được, mâu thuẫn, thiếu nguồn | 29148 + luật Spine | Đỏ / vàng như S-9 |

---

## 6. Vòng đời file — không tích hợp SharePoint

**Quyết định:** không kết nối SharePoint / Google Drive (cần đăng nhập Microsoft, Microsoft Graph API, xin quyền công ty). Workspace = bộ tài liệu dự án **nằm trong FlintFlow**; người dùng tự mang file ra vào.

| Bước | Cách làm |
| --- | --- |
| Sinh ra | Chế độ 2, 3: writer điền vào file mẫu → version đầu |
| Up lên | Chế độ 1: import .docx (I-\*). Bản người ngoài sửa: upload lại |
| Lưu trữ | Mỗi lần ghi = một version của cùng file trong kho file |
| Nhận về | Tải version có Track Changes; watermark DRAFT nếu chưa baseline |
| **Dấu version** | Mỗi file tải về có mã dự án + version **trong file** (document property) và **trong tên file** — vì người dùng tự mang file đi lại giữa SharePoint và FlintFlow, dễ upload nhầm bản cũ |
| Upload lại | I-1 đọc dấu version → import như **version mới** → **báo cáo khác biệt theo block** → người tạo CR đồng bộ. So 3 chiều tự động để ở P5 |

Nói với thầy: *"Workspace là bộ tài liệu dự án trong FlintFlow; kết nối SharePoint/Drive là bước mở rộng sau capstone."*

---

## 7. Ví dụ xuyên suốt — "Cổng cấp phép xây dựng trực tuyến"

> Mẫu SRS của Sở Xây dựng là **giả định** để minh hoạ. Field Spine và step ID lấy theo `srs-spine.md` §4 và Phases §6.4.

### 7.1 Hai profile

| Field Spine | Step ghi | Profile FPT | Profile Sở XD (tiếng Việt) |
| --- | --- | --- | --- |
| `project.vision`, `.release_scope` | S-2.1, S-2.2 | `fixed:1` | `soxd:ch1` Chương 1. Giới thiệu |
| `actors[kind=system]` | S-2.3 | `fixed:1` | — |
| `business_rules[tier=high]` | S-2.4 | `fixed:1` | — |
| `actors[]` | S-3.1 | `fixed:2.1` | cột "Người thực hiện" trong `soxd:ch4` |
| `use_cases[]` | S-3.2 → S-3.5 | `fixed:2.2.2` | `soxd:ch3` Danh sách chức năng + `soxd:ch4` |
| `diagrams[usecase]` | S-3.6 | `fixed:2.2.1` | — |
| `screens[]`, `features[]` | S-4.1 | `fixed:3.1.2` | — |
| `screens[].flow_to[]` | S-4.2 | `fixed:3.1.1` | — |
| `roles[]`, `permissions[]` | S-4.3 | `fixed:3.1.3` | — |
| `functions[screen_id=null]` | S-4.4 | `fixed:3.1.4` | `soxd:ch4` |
| `entities[]` | S-4.5 | `fixed:3.1.5` | — |
| `functions[].*` | S-5.2, S-5.4 | `function:<id>` | `soxd:ch4` |
| `diagrams[screen_layout]` | S-5.3 | `function:<id>` | — |
| `nfrs[]` | S-6.1 → S-6.5 | `fixed:4.1`, `fixed:4.2.x` | `soxd:ch5` Yêu cầu phi chức năng |
| `business_rules[tier=detail]` | S-7.1 | `fixed:5.1` | — |
| `common_requirements[]`, `messages[]`, `other_requirements[]` | S-7.2 → S-7.4 | `fixed:5.2`–`5.4` | — |
| `glossary[]` | S-8.1 | `fixed:5.5` | — |
| *(không field nào)* | — | — | `soxd:ch2` Quy trình nghiệp vụ · `soxd:appA` Dự toán — **bắt buộc** |

Cột bảng `soxd:ch4`:

| Mã CN | Tên | Người thực hiện | Mô tả | Điều kiện | Kết quả |
| --- | --- | --- | --- | --- | --- |
| `use_cases[].id` | `.name` | `.actor_ids` → tên actor | `.description` | `.preconditions` | `.postconditions` |

### 7.2 Chế độ 1 — Sửa SRS có sẵn

**Tình huống:** công ty có `SRS_CapPhepXD_v2.3.docx` (180 trang, mẫu Sở, tải từ SharePoint của công ty). Sếp chuyển email:
> "Bổ sung: công trình trên 7 tầng phải có văn bản thẩm duyệt PCCC khi nộp hồ sơ."

**I-\* (một lần):**
```text
I-1  .docx ✔ · không còn Track Changes cũ ✔ · chưa có dấu version FlintFlow → hỏi xác nhận bản mới nhất ✔
I-2  2.140 block
I-3  chọn profile Sở XD (đã có)
I-4  Spine: 14 UC, 22 function, 31 validation (3 UC độ tin thấp → BA xác nhận)
     → BASELINE v0 → S-9.1/9.2: 6 gap
```

**CR-012:**

| Step | Kết quả |
| --- | --- |
| C-1 | Nguồn: email anh Nam 10/09 · người tạo: BA Linh (Analyst) |
| C-2 | Rõ, không cần hỏi lại |
| C-3 | Spine: `functions[F-S04-01]` (upload hồ sơ, màn S-04) → `use_cases[UC-03]` Nộp hồ sơ, `use_cases[UC-07]` Thẩm định. Tìm từ khoá "PCCC", "số tầng" → thêm 1 block trong `soxd:ch2` |

**C-4 — gọi skill của step sở hữu field:**

| Field bị ảnh hưởng | Step sở hữu | Section / block | Đề xuất |
| --- | --- | --- | --- |
| `functions[F-S04-01].validations[]` | **S-5.4**@S-04 | `soxd:ch4`, hàng F-S04-01 | Thêm validation: "số tầng > 7 → bắt buộc file văn bản PCCC" |
| `use_cases[UC-03].preconditions` | **S-3.5** | `soxd:ch4`, hàng UC-03, cột Điều kiện | Thêm "đã có văn bản PCCC nếu công trình > 7 tầng" |
| `use_cases[UC-07].description` | **S-3.5** | `soxd:ch4`, hàng UC-07, cột Mô tả | Thêm bước kiểm tra văn bản PCCC |
| *(không field)* | — | block trong `soxd:ch2` | Chỉ **comment**: "cần cập nhật quy trình theo CR-012" |

**Không gọi:** S-7.1 — profile Sở không render `business_rules[tier=detail]`, không step nào đang chạy đọc tới. Nếu là SRS mẫu FPT, S-7.1 sẽ được gọi để cập nhật `fixed:5.1`.

```text
C-5  chữ cũ khớp đúng block ✔ · luật Spine ✔ · S-8.4 trên 3 block ✔ → khoá 3 block
C-6  BA Linh gửi → Lead duyệt cả nhóm
C-7  SRS_CapPhepXD_v2.4.docx: 3 chỗ Track Changes (tác giả "CR-012") + 1 comment · dấu version v2.4
```

### 7.3 Chế độ 2 — Tạo mới theo mẫu FPT

**Tình huống:** công ty khác làm cổng cấp phép cho một tỉnh; chủ đầu tư **không quy định mẫu**; PO có ý tưởng + vài biên bản họp.

```text
Bước 0   chọn mẫu → [dùng FPT] → profile FPT, tiếng Anh
B-0…B-2  Intake đọc biên bản → AI chỉ hỏi phần thiếu → Brief (13 step)
S-1…S-9  profile FPT render mọi field → chạy ĐỦ 51 step cố định + 5 × N, gate mỗi step
S-8.2    writer điền vào file mẫu FPT
S-9.5    cờ đỏ = 0 → BASELINE v1
```
Sau baseline → thay đổi đi C-\* như chế độ 1.

### 7.4 Chế độ 3 — Tạo mới theo mẫu user cung cấp

**Tình huống:** Sở XD mở gói thầu mới, **bắt buộc** nộp SRS theo `Mau_SRS_SoXD.docx`. Chưa có SRS.

**Bước 0 — tạo profile (một lần mỗi mẫu):**
```text
Upload Mau_SRS_SoXD.docx → khớp heading → section (bảng 7.1) · khớp 6 cột soxd:ch4
→ đánh dấu chữ hướng dẫn cần xoá ("Mục này mô tả các chức năng…")
→ section bắt buộc: soxd:ch1…ch5, soxd:appA · ngôn ngữ: tiếng Việt
```

**Lọc step (mục 5):**

| Step | Chạy? | Ghi file? | Lý do |
| --- | :-: | :-: | --- |
| B-0 → B-2, S-1.1 → S-1.4 | ✔ | — | Luật 3: luôn chạy |
| S-2.1, S-2.2 | ✔ | ✔ | Luật 1: render `soxd:ch1` |
| S-2.3 External Systems | ✔ | ✗ | Luật 2: S-3.1 đọc `actors[kind=system]` |
| S-2.4, S-2.5 | ✗ | ✗ | Không render, không ai đọc |
| S-3.1 Actors | ✔ | ✔ | Luật 1: cột "Người thực hiện" trong `soxd:ch4` |
| S-3.2, S-3.3, S-3.5 | ✔ | ✔ | Luật 1: `soxd:ch3`, `soxd:ch4` |
| S-3.4, S-3.6 | ✗ | ✗ | Không render, không ai đọc |
| S-4.1 Screen Inventory | ✔ | ✗ | Luật 2: S-5 đọc `screens[]`, `screen_queue[]` |
| S-4.4 Non-Screen Functions | ✔ | ✔ | Luật 1: `soxd:ch4` |
| S-4.2, S-4.3, S-4.5 | ✗ | ✗ | Không render, không ai đọc |
| S-5.1, S-5.5 @mỗi màn | ✔ | — | Luật 3 |
| S-5.2, S-5.4 @mỗi màn | ✔ | ✔ | Luật 1: `soxd:ch4` |
| S-5.3 Screen Layout | ✗ | ✗ | Mẫu không có wireframe |
| S-6.1 → S-6.5 | ✔ | ✔ | Luật 1: `soxd:ch5` |
| S-7.1 → S-7.4 | ✗ | ✗ | Không render, không ai đọc |
| S-8.1, S-8.3 | ✗ | ✗ | Mẫu không có glossary / record of changes |
| S-8.2 | ✔ | ✔ | Writer điền mẫu Sở |
| S-8.4, S-9.1 → S-9.5 | ✔ | — | Luật 3 |
| *(soxd:ch2, soxd:appA)* | — | trống | Luật 5: **cờ đỏ "cần viết tay"** |

→ **38 step cố định + 4 × N** (ước tính), so với 51 + 5 × N của FPT.

**Ghi và kiểm tra:**
```text
S-8.2  copy mẫu → SRS_GoiThau05.docx → xoá chữ hướng dẫn → điền dưới từng heading (tiếng Việt)
       → soxd:ch4: mỗi UC / function một hàng bảng 6 cột
S-9    Tầng 1 (profile Sở): soxd:ch2, soxd:appA trống → CỜ ĐỎ
       Tầng 2 (29148):      không có entities[] → cờ vàng
       Tầng 3:              UC-09 "xử lý nhanh" không đo được → cờ vàng
→ BA viết tay soxd:ch2, soxd:appA → cờ đỏ = 0 → BASELINE v1
```
Sau baseline → thay đổi đi C-\* như chế độ 1.

### 7.5 So sánh

| | 1. Sửa có sẵn | 2. Tạo mới FPT | 3. Tạo mới mẫu họ |
| --- | --- | --- | --- |
| Bắt đầu từ | File SRS của khách | Ý tưởng / dữ liệu thô | Ý tưởng / dữ liệu thô + file mẫu |
| Profile | Chọn / tạo ở I-3 | FPT (có sẵn) | Tạo ở bước 0 |
| Phase | I-\* → C-\* | B-\* → S-\* đủ | B-\* → S-\* đã lọc |
| Step chạy | Skill của step sở hữu field bị ảnh hưởng | 51 + 5 × N | 38 + 4 × N (ví dụ) |
| Ghi file | Track Changes trên file cũ | Điền mẫu FPT | Điền mẫu họ |
| Section không có field | Chỉ comment | Không có | Trống + cờ đỏ nếu bắt buộc |
| Ngôn ngữ nội dung | Theo profile | Tiếng Anh | Theo profile |
| Tiến độ hiển thị | Trạng thái CR | Đếm step | Đếm step |
| Sau baseline | C-\* | C-\* | C-\* |

---

## 8. Thứ tự làm

Làm **theo chiều dọc**, cắt từ dưới lên nếu trễ (cùng tinh thần Phases §9.3).

| Ưu tiên | Làm | Chứng minh | Nếu trễ |
| --- | --- | --- | --- |
| **P0** (2 tuần) | Thử nghiệm: (a) chèn `w:ins/w:del` vào đoạn + ô bảng, `paraId` có ổn định không; (b) import 1 SRS thật không theo mẫu FPT — tỉ lệ bảng / văn xuôi, độ chính xác trích xuất; (c) đo token 1 CR | 3 giả định gốc | (a) sai → tô màu + comment · (b) sai → cam kết chỉ FPT + mẫu dạng bảng |
| **P1** | **Chế độ 1 trên file FPT**: I-1…I-4 → C-1…C-7. Fixture = SRS của chính FlintFlow | **Demo chính**, câu "file 200 trang" của thầy | Không được cắt |
| **P2** | Chế độ 2: pipeline cũ, S-8.2 → writer điền mẫu FPT | Luồng cũ chạy, một writer duy nhất | Giữ S-8.2 cũ, xuất file mới |
| **P3** | Profile thứ 2 + chế độ 1 trên mẫu đó | "Kiến trúc hỗ trợ nhiều mẫu" | Chỉ trình bày thiết kế |
| **P4** | Chế độ 3: lọc step + điền mẫu thứ 2 | Tạo mới theo mẫu khách | **Cắt đầu tiên** |
| **P5** | 3 vai trò đầy đủ, so 3 chiều tự động khi upload lại, audit log đầy đủ | Làm nhóm | Rút Lead / Analyst; đồng bộ bằng báo cáo khác biệt + CR tay |

P1 dùng lại skill S-3.5, S-5.4 (qua C-4) → vẫn phải xây các skill đó sớm.

**Phương án không còn đúng khi:** P0 cho thấy SRS thật chủ yếu văn xuôi → chế độ 1 chỉ còn gap + danh sách thay đổi + comment, demo chính chuyển về chế độ 2 · thầy yêu cầu duyệt trong Word hoặc tích hợp SharePoint.

---

## 9. Cần chốt

**Nhóm chốt:**
1. Bỏ "loại section" — dùng Spine field + template profile; profile FPT giữ ID `fixed:*`.
2. Thêm phase `I-1`…`I-4`, `C-1`…`C-7`; step khai field đọc.
3. Ngôn ngữ nội dung theo profile (sửa Phases §1.3).
4. Thứ tự ưu tiên P0 → P5.

**Đã chốt:** không tích hợp SharePoint / Google Drive (mục 6).

**Sửa tài liệu theo thứ tự** (sau khi nhóm chốt):
1. `Product-Brief-to-SRS-Phases.md` §1 (quy ước), §6.4 (thêm I-\*, C-\*, cột field đọc).
2. `srs-spine.md` §4 → profile FPT; thêm neo, `baselines[]` loại `imported`.
3. `docs/scope-revision-options.md` — đồng bộ: bỏ "loại section", tạo mới theo mẫu họ, câu hỏi SharePoint thành quyết định.
4. `usecase.md`, prototype.
