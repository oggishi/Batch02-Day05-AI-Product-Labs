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
| Moni phân loại giao dịch nhưng không hiện độ tin cậy; sửa không rõ có học | Self-use Đạt (778) | User không biết nhãn nào đáng tin, sửa mệt | Thêm **badge độ tin cậy** + correction **lưu thành rule** |
| Con số chi tiêu/đáp chat không dẫn nguồn dữ liệu | [chatbot_momo.jpg](../Day06-2A202600778-TranBaDat/01-invidual-workshop/chatbot_momo.jpg) | User không kiểm chứng được | Gắn nguồn "dựa trên N giao dịch của bạn" |
| Chatbot báo "đã gửi SMS"/"đã chuyển CSKH" nhưng không có trạng thái/recovery thật | [case1.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case1.jpg), [case2.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case2.jpg) | AI over-claim → user kẹt | Nguyên lý chung: **chỉ khẳng định khi có trạng thái thật + luôn có recovery** |
| Gợi ý đầu tư không hỏi suitability | Self-use Thành Đạt (771) | Lời khuyên có thể hại user | (Backlog) suitability-check trước khi nudge |
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
prototype sẽ mô phỏng flow gửi email từ chatbot với các cải tiến:
1. Hiển thị trạng thái gửi rõ ràng: "Đang gửi..." → "Đã gửi" / "Gửi thất bại" + nguyên nhân cụ thể
2. Khi gửi thất bại hoặc email chưa xác thực, đưa ra 3 phương án thay thế: 
   - Tải báo cáo trực tiếp trong app (PDF/CSV)
   - Gửi link tải qua SMS
   - Hiển thị preview báo cáo ngay trong chat
3. Cho phép gửi lại dễ dàng + hướng dẫn rõ ("Nếu không nhận trong 5 phút, bấm gửi lại hoặc chọn phương án khác")
4. Xử lý failure mode "user chờ email mà không có trạng thái, bị kẹt" bằng cách hiển thị recovery options ngay.
```

## 5. Auto/Aug decision

- [x] **Augmentation:** Chatbot hỏi & xác nhận trước khi gửi + hiển thị trạng thái + recovery options.
- [ ] Conditional automation.
- [ ] Automation.

**Lý do chọn:** Gửi email liên quan tới dữ liệu tài chính nhạy cảm & kỳ vọng user. Cần xác nhận trước gửi + trạng thái rõ ràng để user không bị kẹt hoặc tin sai. Không nên tự động gửi mà không xác nhận + trạng thái.
**Human role:** **decider** (chốt email/phương án gửi) + **observer** (thấy trạng thái gửi thật).

## 6. Four paths

| Path | Prototype phải thể hiện gì? |
|---|---|
| **Happy** | User: "Gửi báo cáo email cho tôi" → Chatbot xác nhận email → Gửi → Hiển thị "✅ Đã gửi thành công tới [email]" + thời gian. |
| **Low-confidence** | User cung cấp email lạ → Chatbot hỏi lại: *"Gửi tới email này: [email]? (có/không)"* hoặc *"Email này có được xác thực không? Gửi thay vào email trong MoMo?"* |
| **Failure** | Email gửi thất bại → Hiển thị "❌ Gửi thất bại (email chưa xác thực)" + 3 phương án: Tải trực tiếp / Gửi qua SMS / Gửi lại. |
| **Correction** | User chọn phương án khác (vd "Tải trực tiếp") → Hiển thị link tải PDF/CSV + thời gian sẵn sàng + hướng dẫn rõ. |

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
| Nguyễn Thị Bảo Trân (917) | **Frontend UI mockup** (HTML/CSS/React component): hiển thị chatbot dialog cho gửi email + trạng thái gửi (pending/success/failed) + 3 phương án recovery (tải app, SMS, gửi lại) | Evidence research + Test failure case "email chưa xác thực / gửi thất bại" | Mockup UI có thể tương tác cho cả 4 paths; test log case email lỗi |
| Trần Bá Đạt (778) | **Backend logic** (Python/Node): email delivery status simulation + recovery option API contract + mock email gateway | SPEC definition + API contract cho status check & recovery trigger | Code `check_email_status()` + `trigger_recovery_option()` + test case logic |
| Nguyễn Thành Đạt (771) | **Integration & Demo** (kết nối FE/BE mockup): chatbot client + demo runner + kịch bản chạy 4 paths (happy/low-confidence/failure/correction) | README + repo structure + Demo script | Working demo (user input → BE check status → FE hiển thị recovery); README hướng dẫn chạy |
