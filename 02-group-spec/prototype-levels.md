# Prototype — 3 Levels Design (Nhóm 086)

> Thiết kế prototype theo 3 mức trong `hackathon_req.md`, áp cho đúng build slice trong [`thin-spec.md`](./thin-spec.md).
> **AI decision cần chứng minh:** chatbot **quyết định khẳng định "đã gửi" hay thừa nhận chưa chắc**, tùy nó có grounding (tool-result xác nhận delivery) hay không. Cùng một yêu cầu, hai mức grounding → hai câu trả lời khác nhau.

---

## 0. Lõi AI dùng chung cho cả 3 level

Mọi level đều xoay quanh **một contract hành vi** của AI. Đây là thứ phân biệt bài này với "thanh trạng thái thường".

### AI behavior contract (system prompt rút gọn)

```text
Bạn là trợ lý MoMo. Khi user yêu cầu một hành động có hệ quả (gửi email, gửi OTP, chuyển CSKH):
- CHỈ được khẳng định hành động THÀNH CÔNG nếu nhận được tool-result với status = "delivered".
- Nếu status = "queued" / "unknown" / "pending" → KHÔNG nói "đã gửi".
  Nói rõ "đã yêu cầu nhưng CHƯA xác nhận được", rồi đưa recovery options.
- Nếu status = "failed" → nói rõ thất bại + nguyên nhân + recovery options.
- TUYỆT ĐỐI không suy đoán kết quả khi thiếu tool-result.
Recovery options chuẩn: [Tải trực tiếp PDF/CSV] · [Nhận link qua SMS] · [Xem preview] · [Gửi lại].
```

### Tool-result giả lập (grounding mà AI đọc vào)

```json
{ "action": "send_email_report", "email": "user@example.com", "status": "queued" }
// status nhận một trong: delivered | queued | unknown | failed
```

### Bộ test case (dùng lại cho cả 3 level — đây là "prompt/test minh họa AI behavior")

| ID | Input user | Grounding (tool-result) | Hành vi AI ĐÚNG (kỳ vọng) | Path |
|---|---|---|---|---|
| T1 | "Gửi báo cáo chi tiêu qua email" | `status: delivered` | Khẳng định: "✅ Đã gửi tới [email] lúc [giờ]" | Happy |
| T2 | (như trên) | `status: queued` / `unknown` | KHÔNG nói đã gửi → "đã yêu cầu, chưa xác nhận" + recovery | Low-confidence |
| T3 (baseline lỗi) | (như trên) | *không có tool-result* (AI cũ, không có contract) | AI over-claim "✅ Đã gửi thành công" (bug As-Is) | Failure |
| T4 | "Cho tôi tải trực tiếp" | `download_ready: true` | Trả link PDF/CSV thật, xác nhận bằng trạng thái thật | Correction |

> Điểm mấu chốt chấm điểm: **T2 vs T3 cùng yêu cầu nhưng khác grounding → AI trả lời khác nhau.** Đó là bằng chứng "một AI decision + một path khi AI không chắc/sai".

---

## Level 1 — Sketch (bắt buộc có, là nền cho L2/L3)

**Mục tiêu rubric:** *Flow rõ tất cả bước · có prompt/test minh họa AI behavior · tất cả thành viên giải thích được.*

**Cần làm:**
1. **Flow diagram** đầy đủ (vẽ tay/Excalidraw/ASCII) cho cả 4 paths — dùng lại As-Is/To-Be trong thin-spec §5/§6.
2. **Dán AI behavior contract** (mục 0) + **bộ 4 test case** T1–T4 với câu trả lời kỳ vọng viết sẵn.
3. **2 mẫu hội thoại đối chiếu** (As-Is vs To-Be):

```text
AS-IS (over-claim):                      TO-BE (calibrated):
User: Gửi báo cáo qua email              User: Gửi báo cáo qua email
Bot : ✅ Đã gửi thành công!              Bot : Mình đã yêu cầu gửi tới a@x.com,
       (thực ra chưa có xác nhận)               nhưng CHƯA xác nhận được đã tới.
                                                Bạn muốn: [Tải PDF] [SMS link] [Gửi lại]?
```

4. **Mỗi thành viên chốt 1 câu giải thích** "AI decision ở đây là gì và vì sao T2≠T3".

**Output nộp:** ảnh flow + file test case (chính là mục 0 của tài liệu này).
**Owner:** cả nhóm; tổng hợp: Trần Bá Đạt (SPEC).

---

## Level 2 — Mock prototype

**Mục tiêu rubric:** *HTML/app/Figma · thể hiện output AI và fallback · flow rõ + có edge case.*

**Cần làm:**
1. **Mock chat UI** (HTML/CSS hoặc Figma) hiển thị hội thoại chatbot MoMo.
2. **Hard-code 2 kịch bản theo grounding** bằng một công tắc `status` (delivered / queued):
   - `delivered` → bong bóng chat "✅ Đã gửi tới [email]".
   - `queued/unknown` → bong bóng "chưa xác nhận" + **3 nút recovery** bấm được.
3. **Fallback hiển thị được:** bấm "Tải trực tiếp" → hiện link/preview PDF giả; bấm "Gửi lại" → quay lại trạng thái đang xử lý.
4. **Edge case:** `status: failed` (email chưa xác thực) → thông báo lỗi + nguyên nhân + recovery.

**Tiêu chí đạt:** người xem bấm thử thấy **AI đổi câu trả lời theo `status`**, và mọi nhánh fallback đều có đích đến (không nút chết).
**Output nộp:** mock chạy trong trình duyệt / link Figma prototype.
**Owner:** Nguyễn Thị Bảo Trân (FE mock + edge case email lỗi).

---

## Level 3 — Working prototype (đích nhắm tới — "base 10 nếu chạy ổn")

**Mục tiêu rubric:** *Input → AI → output · demo live được.*

**Kiến trúc tối thiểu:**

```text
[User gõ yêu cầu]                         (Nguyễn Thành Đạt — integration/demo)
        │
        ▼
[Chat client / FE]  ──────────────►  [AI layer]  ◄────── system prompt (mục 0)
   (Bảo Trân)                          (gọi LLM, vd Claude Haiku, hoặc rule-engine
        ▲                               nếu không có API key)
        │                                   │ đọc grounding
        │                                   ▼
        └──────────  câu trả lời  ◄──  [Mock email gateway / BE]
                                         check_email_status() → {delivered|queued|unknown|failed}
                                         trigger_recovery_option()        (Trần Bá Đạt — BE)
```

**Cần làm:**
1. **BE mock** (Python/Node): `check_email_status()` trả `status` cấu hình được (để demo ép từng case); `trigger_recovery_option()` cho tải/SMS/gửi lại. — *Đạt (778)*
2. **AI layer:** gọi LLM với system prompt mục 0, **truyền tool-result vào context**; LLM tự quyết claim/hedge. (Nếu không dùng API: rule-engine theo đúng contract.) — *Thành Đạt (771)*
3. **FE chat** nối input→AI→output, render recovery options bấm được. — *Bảo Trân (917) dựng UI, Thành Đạt nối*
4. **Demo runner** chạy lần lượt T1→T2→T3→T4 trước giám khảo.

**Tiêu chí đạt:** gõ thật một câu, đổi `status` ở BE, **AI tự đổi giữa "đã gửi" và "chưa xác nhận + recovery"** — không hard-code câu trả lời.

---

## Demo narrative (90 giây, cuối Day 06)

```text
1. Pain (10s):  "Chatbot MoMo nói 'đã gửi email' mà không hề biết email có tới không." (chỉ 3 screenshot)
2. AI decision (15s): "AI phải quyết định: khẳng định hay thừa nhận chưa chắc — tùy nó có bằng chứng."
3. Live (45s):
   - Chạy T1 (status=delivered) → AI khẳng định đã gửi.      ← happy
   - Đổi status=queued, chạy lại T2 → AI KHÔNG nói đã gửi,    ← path AI không chắc
     thừa nhận chưa chắc + đưa recovery → bấm "Tải PDF".      ← correction
   - (Đối chiếu) tắt contract → AI over-claim như bản gốc.     ← failure As-Is
4. Reflection (20s): nguyên lý chung "AI chỉ claim khi có grounding" áp được cho OTP & CSKH handoff (backlog).
```

## Mức nhắm & phân công nhanh

| Level | Trạng thái nhắm tới | Ai lo chính |
|---|---|---|
| L1 Sketch | Bắt buộc xong sáng Day 06 | Cả nhóm (Đạt tổng hợp) |
| L2 Mock | Xong trưa Day 06 (an toàn nếu L3 trục trặc) | Bảo Trân |
| L3 Working | Đích chiều Day 06 — demo live | Thành Đạt (nối) + Đạt (BE) + Bảo Trân (FE) |

> Chiến lược an toàn: **luôn có L2 chạy được làm phao**, rồi mới đẩy lên L3. Đừng để cả nhóm kẹt ở L3 mà không có gì demo.
