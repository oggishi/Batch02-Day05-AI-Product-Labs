# Thin SPEC — Cuối Day 05 (Nhóm MoMo AI Teardown)

> Bản cam kết đủ rõ để sáng Day 06 build ngay. Không phải PRD đầy đủ.

## 1. Track, product/app và user

**Track:** Fintech / Quản lý tài chính cá nhân

**Product/app thật:** MoMo — Trợ thủ tài chính AI **Moni** (bề mặt phân loại & phân tích chi tiêu)
**User cụ thể:** Người trẻ (sinh viên/đi làm sớm) chi tiêu chủ yếu qua MoMo, mở mục chi tiêu để xem lại tháng vừa rồi tiêu vào việc gì.

**Nhóm có phải user thật không? Nếu không, khác ở đâu?** Có — cả 3 thành viên đều dùng MoMo hàng ngày. Khác biệt: nhóm rành công nghệ hơn user phổ thông, nên sẽ kiểm thêm với 3-5 user ngoài nhóm ở M1.

## 2. Evidence summary

| Evidence | Nguồn | User/pain nói lên điều gì? | SPEC phải đổi gì? |
|---|---|---|---|
| Moni phân loại giao dịch nhưng không hiện độ tin cậy; sửa rồi không rõ có học | case4.jpg | User không biết nhãn nào đáng tin, sửa mệt | Thêm **badge độ tin cậy** + correction **lưu thành rule** |
| Con số chi tiêu/đáp chat không dẫn nguồn dữ liệu | [chatbot_momo.jpg](../Day06-2A202600778-TranBaDat/01-invidual-workshop/chatbot_momo.jpg) | User không kiểm chứng được | Gắn nguồn "dựa trên N giao dịch của bạn" |
| Chatbot báo "đã gửi SMS"/"đã chuyển CSKH" nhưng không có trạng thái/recovery thật | [case1.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case1.jpg), [case2.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case2.jpg) | AI over-claim → user kẹt | Nguyên lý chung: **chỉ khẳng định khi có trạng thái thật + luôn có recovery** |
| Gợi ý đầu tư không hỏi suitability | case5.jpg | Lời khuyên có thể hại user | (Backlog) suitability-check trước khi nudge |
| Chatbot báo "sẽ gửi báo cáo email" nhưng không hiển thị trạng thái gửi / không có phương án thay thế khi lỗi | [case3_1.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_1.jpg), [case3_2.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_2.jpg), [case3_3.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_3.jpg) | User không biết email có đến, phải chờ mà không biết bước tiếp theo | **Trả về trạng thái gửi thật (queued/sent/delivered/failed)** + **gợi phương án thay thế (tải app, link, SMS)** |

## 3. Pain statement

```text
User trẻ quản lý chi tiêu trong MoMo muốn xem báo cáo chi tiêu hàng tháng qua email
để kiểm tra & chia sẻ, nhưng chatbot báo "sẽ gửi" mà không hiển thị trạng thái thực tế (sent/delivered/failed),
user không biết email có đến hay không, phải chờ & kiểm tra mà không có hướng dẫn tiếp theo.
Khi email chưa được xác thực hoặc gửi thất bại, chatbot không cung cấp phương án thay thế.
Bằng chứng chính là 3 case chatbot tuyên bố đã hành động mà không có trạng thái thật:
Case 1 (OTP không trạng thái), Case 2 (CSKH handoff không minh bạch), Case 3 (email không hiển thị gửi thành công/thất bại).
```

## 4. Build slice

```text
Cho user trẻ đang muốn gửi báo cáo chi tiêu tháng vừa rồi qua email,
prototype sẽ mô phỏng flow từ mock transactions → AI agent → báo cáo PDF → email delivery status.
1. AI hiểu intent "Tháng này tôi chi bao nhiêu?" và "Gửi báo cáo cho email của tôi".
2. Tổng hợp giao dịch tháng này, phân bổ theo danh mục và sinh insight.
3. Tạo báo cáo 1 trang PDF có chart + tổng chi + breakdown + AI insight.
4. Gửi email thật và hiển thị trạng thái gửi (sending/success/failure).
5. Nếu email sai / gửi thất bại, cho user sửa/correction và gửi lại.
```

**Prototype architecture:**

```
Mock Transactions
        ↓
     AI Agent
        ↓
 Generate Report
        ↓
 Chart + Insights
        ↓
      PDF
        ↓
     Real Email
```

**Mock transaction data:**

```json
[
  {
    "date": "2026-06-01",
    "category": "Ăn uống",
    "amount": 45000
  },
  {
    "date": "2026-06-03",
    "category": "Ăn uống",
    "amount": 115000
  },
  {
    "date": "2026-06-05",
    "category": "Di chuyển",
    "amount": 25000
  },
  {
    "date": "2026-06-10",
    "category": "Học tập",
    "amount": 55000
  },
  {
    "date": "2026-06-20",
    "category": "Hóa đơn",
    "amount": 10000
  }
]
```

**AI insight example:**

```text
Tổng chi tiêu tháng này: 250.000đ

Danh mục lớn nhất:
Ăn uống (64%)

Nhận xét:
Chi tiêu ăn uống đang chiếm phần lớn ngân sách.
Nếu giảm 20% nhóm này bạn có thể tiết kiệm khoảng 32.000đ/tháng.
```

**Visualization:**
- Hiển thị biểu đồ tròn giống MoMo: phân bổ chi tiêu theo danh mục.
- Thêm label tổng chi và các số liệu chính.

**PDF:**
- 1 trang gồm header "Moni Monthly Report", tháng/năm, tổng chi, chart, breakdown, AI Insight.
- Có thể tạo bằng `reportlab` (Python) hoặc `jsPDF` (React).

**Email delivery:**
- Chatbot hỏi email nhận báo cáo.
- Hiển thị trạng thái: "Generating report..." → "PDF created" → "Sending email..." → "✅ Email delivered".
- Nếu lỗi, hiển thị lỗi email và cho user sửa/correction.
```

## 5. Auto/Aug decision

- [x] **Augmentation:** Chatbot hỏi & xác nhận trước khi gửi + hiển thị trạng thái + recovery options.
- [ ] Conditional automation.
- [ ] Automation.

**Lý do chọn:** Gửi email liên quan tới dữ liệu tài chính nhạy cảm & kỳ vọng user. Cần xác nhận trước gửi + trạng thái rõ ràng để user không bị kẹt hoặc tin sai. Không nên tự động gửi mà không xác nhận + trạng thái.
**Human role:** **decider** (chốt email/phương án gửi) + **observer** (thấy trạng thái gửi thật).

## 6. Four paths

### Happy

```text
User: Gửi báo cáo
↓
PDF tạo thành công
↓
Email gửi thành công
```

### Low-confidence

```text
User: Gửi báo cáo
↓
AI thấy có 2 email
↓
Hỏi chọn email nào
```

### Failure

```text
Email sai format
↓
Không gửi được
```

### Correction

```text
Email sai
↓
User sửa email
↓
Gửi lại
```

## 7. Failure mode nguy hiểm nhất

```text
User yêu cầu gửi báo cáo email nhưng chatbot chỉ nói "đang gửi" mà không có trạng thái thật,
user chờ mà không biết email có đến hay không, phải tự kiểm tra inbox hoặc spam folder,
hoặc phải quay lại chat hỏi lại mà không có quyền lực gì (vì chatbot không cung cấp phương án backup).
Hậu quả: user mất niềm tin, bỏ cuộc hoặc phải tìm cách khác → trải nghiệm xấu.
Prototype sẽ xử lý bằng: hiển thị trạng thái gửi thật (sent/failed/pending) + luôn có recovery options (tải app, SMS, gửi lại).
Owner kiểm thử path này là Nguyễn Thị Bảo Trân.
```

## 8. Owner plan cho sáng Day 06

| Thành viên | Việc phụ trách (Code) | Trách nhiệm bổ sung | Bằng chứng cần có trong repo |
|---|---|---|---|
| Nguyễn Thị Bảo Trân (917) | **Frontend UI mockup** (HTML/CSS/React component): hiển thị chatbot dialog, chart tròn, summary report và email status flow | Evidence research + Test failure case "email sai / gửi thất bại" | Mockup UI có thể tương tác cho cả 4 paths; test log case email lỗi |
| Trần Bá Đạt (778) | **Report logic** (Python/Node): mock transaction aggregation, AI insight generation, PDF creation + email delivery status simulation | SPEC definition + report format + email status states | Code `generate_report()` + `create_pdf()` + `send_email()` + test case logic |
| Nguyễn Thành Đạt (771) | **Integration & Demo** (kết nối FE/BE mockup): chatbot flow, report generation pipeline, email confirmation + resend | README + repo structure + Demo script | Working demo (user input → mock transactions → report/PDF → email status); README hướng dẫn chạy |
