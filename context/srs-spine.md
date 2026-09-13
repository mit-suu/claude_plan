# SRS Spine — mô hình dữ liệu

> Cấu trúc dữ liệu nằm giữa **step** và **tài liệu SRS**. Step ghi field vào Spine; section trong SRS chỉ là kết quả render từ field.
> File này là hợp đồng cho `srs-spine.schema.json`, `draft-to-ops.md`, `apply-change-op.md`, `deterministic-check.md`.
> Quy trình phase/step ở `Product-Brief-to-SRS-Phases.md`. Use case ở `usecase.md`. Template đích: `SRS_template_FPT.md`.

**Tra nhanh:** [lược đồ](#2-lược-đồ) · [ánh xạ field → section](#4-ánh-xạ-field--section) · [vòng đời trạng thái](#5-vòng-đời-sectionsstatus) · [bất biến](#6-bất-biến) · [cờ đỏ](#7-cờ-đỏ-tất-định) · [cờ vàng](#8-cờ-vàng) · [transaction](#3-transaction--đơn-vị-ghi)

---

## 1. Nguyên tắc

**Lưu quan hệ bằng khoá, không lặp tên trong văn bản.** Mọi quan hệ lưu **một chiều duy nhất**; chiều ngược render bằng truy vấn.

Đồ thị khoá này là backing store của **UC 6.3 Requirement Traceability Map** — render map chỉ là truy vấn read-only.

**Path phải phân giải qua khoá, không qua chỉ số mảng.** `actors[1]` trỏ nhầm nếu có phần tử bị chèn/xoá giữa lúc preview và lúc áp. Mảng không có `id` dùng khoá tự nhiên tường minh:

```text
actors[id=A03].name
permissions[screen_id=S3,role_id=R1,action=create]
glossary[id=G07].definition
```

**Cấm op đổi chính khoá.** Đổi `glossary[].term` là `set` trên field `term`, không phải đổi khoá — vì vậy mọi mảng dùng khoá tự nhiên phải có `id` riêng. Không mảng nào trong lược đồ dùng khoá tự nhiên làm định danh bền.

---

## 2. Lược đồ

```text
project          { name, vision, goals[], type, domain, complexity, form_factor,
                   stakes, working_mode: fast|coaching,
                   release_scope { in[], out[] } }
sessions[]       { id, is_pipeline }
progress         { current_phase, current_step, screen_cursor, screen_queue[],
                   elicit_turns_this_phase }
steps[]          { id, status: pending|in_progress|accepted|revision_requested,
                   first_seq, last_seq, accepted_at }

features[]         { id, name, order }
actors[]           { id, name, kind: human|system|time, description }
roles[]            { id, name, actor_id | null }
use_cases[]        { id, name, actor_ids[], function_ids[], description,
                     includes[], extends[] }
screens[]          { id, feature_id, name, description, flow_to[], is_popup, tabs[],
                     primary_function_id, queue_order,
                     detail_status: pending|in_progress|signed_off|placeholder }
permissions[]      { id, screen_id, role_id, action }
entities[]         { id, name, description, relations[] }
functions[]        { id, screen_id | null, feature_id, order, name, trigger,
                     description, normal[], abnormal[],
                     validations[] { id, kind: business|format|required, statement },
                     business_rule_ids[], priority }
nfrs[]             { id, category: interface|usability|reliability|performance|other,
                     statement, kind: quantitative|descriptive,
                     metric?, threshold?, priority }
business_rules[]   { id, tier: high|detail, statement, source_validation_ids[] }
common_requirements[] { id, category, statement }
messages[]         { id, code, text, function_ids[] }
other_requirements[]  { id, kind: risk|assumption|open_question|technical_risk,
                        statement }
glossary[]         { id, term, term_native?, definition }
addendum[]         { id, topic, content, content_en, target_section, captured_at }

diagrams[]       { id, kind, puml, section, owner_kind, owner_id,
                   render_status: ok|error, error?, source_hash, rendered_at }
assumptions[]    { id, path, statement, rationale, origin_step_id,
                   status: unconfirmed|confirmed|rejected, confirmed_at }
flags[]          { id, level: red|yellow, rule_id, section_id, target_id?, message,
                   remediation_step, opened_at_version, resolved_at,
                   waived_by_user, waive_reason, waived_at_version }
sections[]       { id, asset_version }        ← status là hàm tính, không lưu (mục 5)
baselines[]      { id, version, at, snapshot_ref, checked_at_version, waived_count }
changes[]        { seq, txn, op, path, before, value, reason, at, by }
usage[]          { id, step_id, call_kind, attempt, tokens_in, tokens_out, cost,
                   state: reserved|deducted|refunded, expires_at }
spine_version    số tăng đơn điệu, tăng **một lần mỗi transaction**
```

### 2.1 Ghi chú thiết kế

- **`actors[]` gộp cả terminator.** Hệ thống ngoài là actor `kind = system`, xuất hiện ở cả §1 lẫn §2.1. `roles[].actor_id` nối §2.1 với §3.1.3 để hai chỗ không trôi khỏi nhau.
- **`business_rules[].tier`** — `high` render vào §1 (UC 3.1 yêu cầu), `detail` render vào §5.1. `high` có `source_validation_ids[]` rỗng là hợp lệ.
- **`validations[]` có `id`** để `source_validation_ids[]` trỏ tới từng validation, không phải cả function.
- **`nfrs[].metric/threshold` tuỳ chọn.** §4.1 và phần "chuẩn giao diện" của §4.2.1 là mô tả định tính. Quy tắc "phải có số" chỉ áp `category ∈ {reliability, performance}`.
- **`sections[]` không lưu `status`** — status là hàm tính từ `steps[]` và `changes[]` (mục 5). Lưu nó là tự tạo ra hai nguồn sự thật.
- **`baselines[].snapshot_ref`** — bản sạch export **render từ snapshot**, không render từ Spine hiện tại. Nếu không, file đóng dấu v1.0 sẽ chứa nội dung không thuộc v1.0.
- **`usage[].expires_at`** — dòng `reserved` quá hạn bị dọn tự động; nếu không, đóng tab giữa chừng để lại credit treo vĩnh viễn.

### 2.2 Clone project (UC 1.15)

| Giữ | Reset |
| --- | --- |
| `project`, `features[]` … `glossary[]`, `addendum[]`, `diagrams[]` | `changes[]`, `baselines[]`, `flags[]`, `steps[]`, `progress`, `usage[]`, `sessions[]` |
| | `assumptions[].status → unconfirmed`, `confirmed_at → null` |
| | `screens[].detail_status → pending`, `spine_version = 1` |

`sections[]` giữ (nó chỉ chứa `asset_version`). Ghi một dòng `changes[]` `{op:"clone"}`.

`steps[]` reset về rỗng nghĩa là mọi section tính ra `draft` — đúng, vì người clone chưa duyệt gì. `progress` reset về B-0; Intake sẽ thấy Spine đã đầy và Elicit không hỏi lại.

---

## 3. Transaction — đơn vị ghi

**Một step phát một lô op. Bất biến kiểm ở CUỐI lô, không phải từng op.** `spine_version` tăng **một lần** cho cả lô. Mọi op trong lô mang cùng `txn`.

Không có transaction thì các thao tác sau **bất khả thi**, vì chúng đi qua trạng thái trung gian vi phạm bất biến:

| Thao tác | Vì sao cần lô |
| --- | --- |
| Xoá feature ở giữa | Bất biến 5 đòi `order` liên tục — phải renumber các feature sau trong cùng lô |
| Hoán vị hai feature | Op đầu tạo `order` trùng |
| Xoá màn hình | Phải cascade: `functions[screen_id=X]`, `permissions[screen_id=X]`, gỡ khỏi `flow_to[]`, `use_cases[].function_ids[]`, `screen_queue[]`, `steps[id ~ @X]`, `sections[function:*]` |
| Xoá actor | Phải gỡ khỏi `use_cases[].actor_ids[]`, `roles[].actor_id` |

**Op xoá một phần tử đang được tham chiếu:** engine sinh sẵn các op cascade vào cùng lô. Nếu cascade sẽ vi phạm một bất biến khác (ví dụ xoá màn cuối cùng), **từ chối cả lô** và trả về danh sách người tham chiếu để user quyết.

**Rollback.** `steps[].first_seq`/`last_seq` khoanh dải `changes[]` của step. Ngắt giữa step (`status = in_progress`) → resume revert dải đó bằng `changes[].before`, rồi chạy lại step. Không có dải này thì làm lại step sẽ áp op lần hai — `add` sinh phần tử trùng, `set` đè lên bản chưa ai duyệt.

---

## 4. Ánh xạ field → section

Ba quan hệ khác nhau. `stale` lan theo **cả ba cột**.

| Field | Sở hữu (render chính) | Đọc (in ra, không sở hữu) | Suy dẫn (phải chạy lại bước suy) |
| --- | --- | --- | --- |
| `project.vision`, `.goals[]`, `.name` | `fixed:1` | — | — |
| `project.release_scope` | `fixed:1` | — | — |
| `project.type`, `.domain`, `.complexity`, `.stakes` | — | — | `fixed:4.2.1` … `fixed:4.2.4` (ngưỡng NFR) |
| `business_rules[tier=high]` | `fixed:1` | — | — |
| `actors[kind≠human]` | `fixed:1` | `fixed:2.1` | `diagrams[context]` |
| `actors[]` | `fixed:2.1` | `fixed:2.2.2`, `fixed:3.1.3` | `diagrams[usecase]`, `fixed:5.5` |
| `roles[]`, `permissions[]` | `fixed:3.1.3` | — | — |
| `use_cases[]` | `fixed:2.2.2` | — | `diagrams[usecase]` |
| `features[]` | `feature:<id>` | `fixed:3.1.2` | số mục §3.x (assemble) |
| `screens[].name/.description/.feature_id` | `fixed:3.1.2` | `fixed:3.1.1`, `fixed:3.1.3`, `function:<id>` | `diagrams[screen_flow]`, `fixed:5.5` |
| `screens[].flow_to[]/.is_popup/.tabs[]` | `fixed:3.1.1` | — | `diagrams[screen_flow]` |
| `screens[].primary_function_id` | — | — | `diagrams[screen_layout]` — đổi thì hình chuyển chủ |
| `functions[screen_id≠null]` | `function:<id>` | `fixed:3.1.2`, `fixed:2.2.2` | — |
| `functions[screen_id=null]` | `function:<id>` | `fixed:3.1.4`, `fixed:2.2.2` | — |
| `functions[].order` | — | — | số mục §3.x.y (assemble) |
| `functions[].validations[]` | `function:<id>` | — | **`fixed:5.1`** (S-7.1) |
| `functions[].abnormal[]` | `function:<id>` | — | **`fixed:5.3`** (S-7.3) |
| `entities[]` | `fixed:3.1.5` | — | `diagrams[erd]`, `fixed:5.5` |
| `nfrs[category=interface]` | `fixed:4.1` | — | — |
| `nfrs[category=usability]` | `fixed:4.2.1` | — | — |
| `nfrs[category=reliability]` | `fixed:4.2.2` | — | — |
| `nfrs[category=performance]` | `fixed:4.2.3` | — | — |
| `nfrs[category=other]` | `fixed:4.2.4` | — | — |
| `business_rules[tier=detail]` | `fixed:5.1` | `function:<id>` | — |
| `common_requirements[]` | `fixed:5.2` | — | — |
| `messages[]` | `fixed:5.3` | — | — |
| `other_requirements[]` | `fixed:5.4` | — | — |
| `glossary[]` | `fixed:5.5` | — | — |
| `changes[]` | `fixed:I` | — | — |
| `diagrams[kind=context]` | `fixed:1` | — | — |
| `diagrams[kind=usecase]` | `fixed:2.2.1` | — | — |
| `diagrams[kind=screen_flow]` | `fixed:3.1.1` | — | — |
| `diagrams[kind=erd]` | `fixed:3.1.5` | — | — |
| `diagrams[kind=screen_layout]` | `function:<primary_function_id>` | — | — |

**Không ánh xạ ra section** (state nội bộ): `sessions[]`, `progress`, `steps[]`, `assumptions[]`, `flags[]`, `sections[]`, `baselines[]`, `usage[]`, `spine_version`, `addendum[]`, `project.form_factor/.working_mode`.

### 4.1 `reference_fields[]` — danh sách quyền uy

Dùng chung cho **ba việc**: impact query · bất biến 3 (id chết) · cờ đỏ `dead_reference`. Ba nơi này phải đọc cùng một danh sách, không phải ba bản chép tay.

```text
use_cases[].actor_ids[]        use_cases[].function_ids[]     use_cases[].includes[]
use_cases[].extends[]          screens[].feature_id           screens[].flow_to[]
screens[].primary_function_id  permissions[].screen_id        permissions[].role_id
roles[].actor_id               functions[].screen_id          functions[].feature_id
functions[].business_rule_ids[]  entities[].relations[]
business_rules[].source_validation_ids[]                      messages[].function_ids[]
diagrams[].owner_id            addendum[].target_section      flags[].section_id
assumptions[].path             sections[].id
progress.screen_cursor         progress.screen_queue[]        steps[].id (phần @screen_id)
```

**Giới hạn đã biết:** impact query bắt phụ thuộc **tham chiếu** và **suy dẫn**. Không bắt **ngữ nghĩa** — đổi `project.vision` ảnh hưởng đoạn văn §1 nhưng không khoá nào nối. Ngữ nghĩa chỉ cảnh báo heuristic.

### 4.2 Section dẫn xuất

`fixed:I` (Record of Changes) và `fixed:5.5` (Glossary) là **dẫn xuất thuần**: nội dung tính lại từ Spine, không do user soạn.

- **Không tham gia vòng đời `stale`.** Mọi op ghi `changes[]`, nên nếu `fixed:I` theo luật chung thì nó `stale` vĩnh viễn và điểm sẵn sàng không bao giờ chạm 100%.
- **Render lại vô điều kiện** lúc assemble và lúc export.
- **Không tính vào điểm sẵn sàng**, không sinh cờ đỏ `section_empty`.
- `fixed:5.5` được **chạy lại bắt buộc ở S-9.1** nếu `spine_version` hiện tại lớn hơn `spine_version` lúc S-8.1 chạy.

---

## 5. Vòng đời `sections[].status`

Không phải máy trạng thái — là **hàm tính**. Máy trạng thái sinh ra các cạnh chưa khai và trạng thái kẹt.

```text
status(s) =
  derived                nếu s ∈ {fixed:I, fixed:5.5}
  stale                  nếu ∃ field nuôi s (cột Sở hữu/Đọc/Suy dẫn) có changes[].seq
                         lớn hơn accepted_at của step muộn nhất nuôi s
  accepted               nếu steps_of(s) ≠ ∅ và mọi step ∈ steps_of(s) có status=accepted
  draft                  ngược lại
```

`steps_of(s)` = tập step có field ghi ra thuộc cột **Sở hữu** của `s`. Nghịch đảo của bảng mục 4.

**Ba hệ quả cố ý:**

- **Tập rỗng ⇒ `draft`, không phải `accepted`.** Section chưa step nào chạm không được coi là xong.
- **Không có trạng thái `regenerated`.** Sau hoà giải, section mang **cờ phụ** `awaiting_reaccept = true` cho tới khi user Accept ở cổng chốt của step sở hữu. Cờ phụ hiển thị riêng, **không** tính là sẵn sàng.
- **Không cần luật ưu tiên.** `stale` được kiểm trước `accepted` trong chính định nghĩa, nên không có ca hai luật cùng đúng.

**Điểm sẵn sàng:**

```text
% = count(status = accepted) / count(section bắt buộc, trừ derived)
```

`awaiting_reaccept` hiển thị riêng: `72% accepted · 4 section chờ duyệt lại`.

---

## 6. Bất biến

Vi phạm ⇒ **từ chối cả transaction**. Kiểm ở cuối lô, áp cho cả `Draft` lẫn luồng sửa.

| # | Bất biến |
| --- | --- |
| **1** | Không xoá được section bắt buộc. Danh sách **theo khoá logic**: `fixed:1` · `fixed:2.1` · `fixed:2.2.1` · `fixed:2.2.2` · `fixed:3.1.1`…`fixed:3.1.5` · `feature:*` (≥1) · `function:*` (≥1) · `fixed:4.1` · `fixed:4.2.1` · `fixed:4.2.2` · `fixed:4.2.3` · `fixed:5.1`…`fixed:5.5`. `fixed:4.2.4` **tuỳ chọn**. `fixed:I` và `fixed:5.5` là dẫn xuất |
| **2** | Không xoá phần tử **cuối cùng** của: `actors[]`, `actors[kind=human]`, `screens[]`, `entities[]`, `use_cases[]`, `features[]`, `functions[]`, `roles[]`, `nfrs[category=reliability]`, `nfrs[category=performance]`, `common_requirements[]` |
| **3** | Mọi khoá trong `reference_fields[]` (mục 4.1) trỏ tới phần tử tồn tại |
| **4** | `functions[].screen_id` thuộc `screens[]` hoặc `null`. `screens[].feature_id` và `functions[].feature_id` **không null** |
| **5** | `features[].order` duy nhất và liên tục từ 0. `functions[].order` duy nhất trong feature, tính **riêng** cho `screen_id ≠ null` và `screen_id = null` |
| **6** | `functions[].feature_id == screens[functions[].screen_id].feature_id` khi `screen_id ≠ null`. *(Template §3.1.2 có cột `Feature | Screen` — mỗi màn thuộc đúng một feature, function kế thừa feature của màn)* |
| **7** | Mỗi project có **đúng một** `sessions[is_pipeline = true]`. Session giữ cờ bị xoá ⇒ tự promote session hoạt động gần nhất |
| **8** | Không xoá được `screens[id = progress.screen_cursor]`. Op thêm screen khi `current_phase = S-5` phải append vào `screen_queue[]` với `detail_status = pending` |

> Bất biến 5 và 6 là lý do transaction tồn tại — xem mục 3.

---

## 7. Cờ đỏ tất định

Hàm kiểm chạy trên đồ thị khoá, **không gọi model**. Chặn ghi `baselines[]`.

**Mỗi cờ đỏ bắt buộc có `remediation_step` thi hành được.** Không được định nghĩa cờ đỏ nào thiếu nó — đó là cách user kẹt cứng ở gate cuối.

| `rule_id` | Điều kiện | `remediation_step` | Waive được |
| --- | --- | --- | :---: |
| `section_empty` | Section bắt buộc (bất biến 1, trừ derived) không field nào có dữ liệu | step sở hữu section đó (nghịch đảo bảng mục 4) | ✔ |
| `array_empty` | Mảng ở bất biến 2 rỗng | step sinh mảng đó | ✘ |
| `dead_reference` | Khoá trong `reference_fields[]` không tồn tại | step sở hữu field chứa khoá | ✘ |
| `render_error` | `diagrams[].render_status = error` | step render hình đó | ✘ |
| `diagram_stale` | `diagrams[].source_hash` ≠ hash hiện tại của `source_fields` (mục 7.1) | step render hình đó | ✔ |
| `nfr_missing_number` | `count(nfrs[category=reliability]) = 0` **hoặc** `count(nfrs[category=performance]) = 0` **hoặc** phần tử của hai loại đó thiếu `metric`/`threshold` | S-6.3 / S-6.4 | ✔ |
| `unconfirmed_assumption` | `count(assumptions[status=unconfirmed]) > 0` tại S-9 | `assumptions[].origin_step_id`, hoặc S-9.1 sweep | ✔ |
| `section_stale_at_baseline` | `status(s) = stale` với `s` bắt buộc | hoà giải, hoặc Accept lại ở step sở hữu | ✔ |
| `section_awaiting_reaccept` | `awaiting_reaccept = true` với `s` bắt buộc | cổng chốt của step sở hữu | ✔ |
| `screen_pending_at_baseline` | `screens[].detail_status = pending` | S-5 cho màn đó | ✔ |

**Ba luật `✘`** là vi phạm bất biến hoặc lỗi kỹ thuật, không phải quyết định nghiệp vụ — waive chúng nghĩa là ký baseline trên Spine gãy.

**Khoá `flags[]`:** `(level, rule_id, section_id, target_id, resolved_at IS NULL)`. Có `resolved_at` trong khoá thì cờ tái phát sinh dòng mới, không upsert đè lên dòng đã đóng. `target_id` nullable — luật cấp section thì `section_id` giữ vai trò định danh.

**Waiver hết hạn.** `waived_at_version`; nếu `spine_version` hiện tại khác và điều kiện lỗi vẫn đúng, waiver mất hiệu lực và cờ mở lại. Không có luật này thì waive một lần là waive vĩnh viễn kể cả khi nội dung đổi hoàn toàn.

**`baselines[].waived_count > 0` ⇒ version là `v1.0-conditional`**, không phải `v1.0`. Mọi export kể cả bản sạch in danh sách waive vào §I.

**`waive_reason`** bắt buộc, ≥ 20 ký tự, do user gõ — không sinh bởi model.

### 7.1 `source_fields` cho `diagram_stale`

Hash trên projection đã sắp xếp theo `id`, chỉ gồm field thật sự xuất hiện trên hình. Hash rộng hơn thì mỗi lần sửa mô tả là một lần vẽ lại hình y hệt.

| `kind` | `source_fields` |
| --- | --- |
| `context` | `project.name` · `actors[kind≠human].name` |
| `usecase` | `actors[].name/.kind` · `use_cases[].name/.actor_ids/.includes/.extends` |
| `screen_flow` | `screens[].name/.flow_to/.is_popup/.tabs` |
| `erd` | `entities[].name/.relations` |
| `screen_layout` | `screens[<owner>].name` · `functions[screen_id=<owner>].name/.description` |

---

## 8. Cờ vàng

Không chặn baseline. Hai nguồn:

**8.1 Cardinality — tất định, có `rule_id`**

| `rule_id` | Điều kiện | `remediation_step` |
| --- | --- | --- |
| `orphan_actor` | `actors[kind=human]` không thuộc use case nào | S-3.2 |
| `usecase_no_function` | `use_cases[].function_ids[]` rỗng | S-3.2 |
| `screen_no_function` | Màn không có function nào | S-4.1 |
| `empty_feature` | Feature không có screen lẫn function | S-4.1 |
| `role_no_actor` | `roles[].actor_id` null | S-3.1 |
| `non_english_content` | Field thuộc cột **Sở hữu** chứa ký tự có dấu tiếng Việt | step sở hữu field |

> Cardinality là **cờ vàng**, không phải cờ đỏ. Nếu là cờ đỏ thì mọi màn `placeholder` ở vòng một dính `screen_no_function` và phải waive hàng loạt.

**8.2 Lens LLM** — S-9.2 (UC 6.9), xử lý qua UC 6.10. Mâu thuẫn FR↔NFR, rủi ro bảo mật, câu chữ mơ hồ.

---

## 9. Ba nhánh xử lý một yêu cầu thay đổi

Chạy **impact query** trên `reference_fields[]` + ba cột của bảng mục 4:

| Impact | Hành vi | UC |
| --- | --- | --- |
| Không field nào trỏ tới field bị đổi | Áp lô, ghi `changes[]` | 6.2 |
| Có phụ thuộc | Báo phạm vi → preview diff (UC 6.1) → xác nhận → áp lô → ghi `flags[]` nếu mâu thuẫn | 6.2 · 6.11 nếu lệnh mơ hồ |
| **Sau khi đã có baseline** | Bắt buộc «include» Impact Analysis. Nếu sau khi áp còn cờ đỏ chưa xử lý thì **không ghi baseline mới** — tài liệu thành "v1.0 + thay đổi chưa baseline" | **6.8** |
| Phá bất biến | **Từ chối cả lô**, trả danh sách người tham chiếu | — |

> UC 6.8 có `Precondition: baseline exists`. Trước baseline đầu tiên mọi thay đổi đi theo **UC 6.2**, kể cả sửa Brief muộn.

**Không tự lan tỏa.** Đổi §2.1 không kích hoạt viết lại §2.2 — chỉ làm `status` tính ra `stale`. User sửa xong nhiều thứ → bấm hoà giải **một lượt** (UC 6.10). Hoà giải đọc **chỉ field phụ thuộc**, qua preview diff, và **render lại mọi `diagrams[]` có `source_hash` lệch**.

User từ chối một diff ⇒ section đó vẫn `stale`. Đó là kết quả hợp lệ; ở gate nó thành cờ `section_stale_at_baseline` (waive được).

---

## 10. Chưa chốt

- **Đo token** cho một SRS end-to-end, kịch bản 19 màn. Chi phí tăng **bậc hai** theo số step nếu mỗi Intake nạp toàn Spine — luật context ở mục 3 của `Product-Brief-to-SRS-Phases.md` giảm nó, nhưng chưa ai đo.
- **`assumptions[]` bị reject**: giá trị đã ghi vào Spine có bị hoàn nguyên không, hay chỉ đánh dấu. Hiện `unconfirmed_assumption` chỉ đếm `unconfirmed`, nên giả định bị reject không chặn baseline dù nội dung vẫn nằm trong tài liệu.
- **Thời gian lưu `addendum[].content`** và tài liệu user upload (UC 2.2).
