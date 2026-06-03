# Evidence Pack — Nhóm MoMo AI Teardown

> Bản gom bằng chứng cuối Day 05, nộp kèm thin SPEC. Tổng hợp từ 3 bài mổ app cá nhân.

## 1. Nhóm và track

**Tên nhóm:** NHÓM 86

**Thành viên:**
- Trần Bá Đạt — `2A202600778` (mổ Moni — phân tích chi tiêu)
- Nguyễn Thành Đạt — `2A202600771` (mổ Moni — gợi ý/nudge tài chính)
- Nguyễn Thị Bảo Trân — `2A202600917` (mổ MoMo Chatbot — hỗ trợ CSKH)

**Track:** Fintech / Quản lý tài chính cá nhân

**Product/app đã chọn:** **MoMo** — Trợ thủ tài chính AI **Moni** + **MoMo Chatbot** (cùng một app, hai bề mặt AI)

**Build slice đang nghĩ:** Phân loại giao dịch chi tiêu **có kèm độ tin cậy** + vòng **correction học lại** — bản thu nhỏ nhất của insight "AI phải lộ độ tin cậy và có đường recovery".

## 2. Self-use evidence

Cả nhóm tự dùng app và ghi lại điểm gãy.

| Observation | Screenshot/link | Path liên quan | Điều học được |
|---|---|---|---|
| Hỏi Moni tổng chi tiêu tháng → trả về một con số/biểu đồ nhưng **không nói rõ đây chỉ là chi tiêu qua MoMo** (chưa gồm tiền mặt/ngân hàng khác) | [chi_tieu_momo.jpg](../Day06-2A202600778-TranBaDat/01-invidual-workshop/chi_tieu_momo.jpg) | Failure (sai âm thầm) | AI trình bày dữ liệu một phần như thể đầy đủ → cần lộ **phạm vi/độ phủ dữ liệu** |
| Chat hỏi-đáp với Moni: câu trả lời/con số **không chỉ rõ lấy từ giao dịch nào** | [chatbot_momo.jpg](../Day06-2A202600778-TranBaDat/01-invidual-workshop/chatbot_momo.jpg) | Low-confidence / Failure | AI cần **gắn nguồn** ("dựa trên N giao dịch của bạn") thay vì nói chung chung |
| Giao dịch chuyển tiền/chi hộ bị AI tự gán nhóm (vd "mua sắm") mà **không hiện độ tin cậy**, sửa rồi không rõ có học | (Moni — auto-categorize) | Failure / Correction | Cần **badge độ tin cậy** + correction **lưu thành rule** |
| Moni gợi ý chuyển "tiền nhàn rỗi" sang đầu tư chứng chỉ quỹ **mà không hỏi khoản đó có cần dùng gấp / user có đang nợ không** | (Moni — nudge đầu tư) | Failure (Safety) | Lời khuyên tài chính cần **suitability-check + disclosure rủi ro** |
| Yêu cầu kiểm tra SĐT liên kết → chatbot báo "đã gửi mã xác thực" nhưng **user không nhận được SMS**, bot không kiểm tra trạng thái gửi, không có phương án thay thế → user kẹt | [case1.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case1.jpg), [case1_b.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case1_b.jpg) | Failure | AI **tuyên bố đã hành động mà không có trạng thái thật** + **không recovery** |
| Yêu cầu hủy vé → mã không tìm thấy → xin CSKH → bot báo "đang điều hướng CSKH" nhưng **không có cửa sổ/hotline/nút nào xuất hiện** | [case2.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case2.jpg) | Correction / Failure | **Human handoff không minh bạch** — chỉ hiển thị "đang chuyển" khi backend xác nhận thật |
| Yêu cầu gửi báo cáo chi tiêu qua email → bot xác nhận sẽ gửi → **không hiển thị trạng thái gửi rõ ràng (sent/delivered/failed)**, user không biết email có đến hay không; nếu email chưa xác thực không có phương án thay thế | [case3_1.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_1.jpg), [case3_2.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_2.jpg), [case3_3.jpg](../Day06-2A202600917-NguyenThiBaoTran/01-invidual-workshop/case3_3.jpg) | Failure / Low-confidence | AI **tuyên bố hành động mà không trả trạng thái thật** + **không gợi phương án thay thế khi lỗi** |

## 3. User / review / social evidence

| Quote / review / observation | Nguồn | User là ai? | Pain/failure mode |
|---|---|---|---|
| MoMo định vị lại từ "ví điện tử" thành "Trợ thủ tài chính với AI" (29/10/2024) → lời hứa AI là lõi sản phẩm, càng phải đúng | [VnEconomy](https://vneconomy.vn/momo-dinh-vi-thuong-hieu-moi-tro-thanh-tro-thu-tai-chinh-voi-ai.htm) | Toàn bộ user MoMo | Khoảng cách promise–reality khi AI sai |
| Moni tự ghi chép & phân loại giao dịch (ăn uống/di chuyển/mua sắm), cảnh báo ngân sách, gợi ý tiết kiệm/đầu tư | [Điện Máy Chợ Lớn](https://dienmaycholon.com/kien-thuc/tro-thu-chi-tieu-momo-ai-moni) | User quản lý chi tiêu | Phân loại sai làm sai toàn bộ phân tích |
| Rủi ro chung của chatbot AI: ảo giác, không hiểu câu hỏi, cần fallback sang người | [Tuổi Trẻ](https://tuoitre.vn/chatbot-ai-tra-loi-ngay-cang-nhanh-muot-nhung-nen-tin-bao-nhieu-20251203115156601.htm) | User hỏi chatbot | Trả lời tự tin nhưng sai/không dẫn nguồn |


## 4. Competitor / analog evidence

| App / mô hình tham khảo | Họ xử lý task này thế nào? | Pattern học được | Có áp dụng trong 1 ngày không? |
|---|---|---|---|
| Money Lover / Sổ Thu Chi MISA | Cho user **sửa nhóm giao dịch ngay tại dòng**, nhớ lựa chọn cho lần sau | Correction tại chỗ + ghi nhớ rule | Có — mock được trong prototype |
| Gmail / email phân loại | Gợi ý nhãn + cho "Move to" và **học dần** theo hành vi user | Augmentation + learning loop | Có — mô phỏng rule đơn giản |
| Chatbot có "confidence/disclaimer" | Khi không chắc thì **hỏi lại / nói rõ độ chắc** thay vì khẳng định | Low-confidence path tường minh | Có — là chính build slice |

## 5. Evidence -> Insight

```text
Evidence nổi bật nhất:
3 bề mặt AI khác nhau của MoMo (phân tích chi tiêu, gợi ý đầu tư, chatbot hỗ trợ)
đều lặp lại CÙNG MỘT lỗi: AI kết luận/hành động một cách tự tin nhưng KHÔNG cho user thấy
độ tin cậy / nguồn dữ liệu / trạng thái thật, và KHÔNG có đường recovery khi sai.

Insight:
User trẻ dùng MoMo không chỉ cần AI trả lời nhanh hay tự động gán nhãn.
Họ thật ra cần biết KHI NÀO nên tin AI và có đường sửa khi AI sai,
vì khi AI giấu độ tin cậy/nguồn/trạng thái thì user tin sai (về chi tiêu, về lời khuyên)
hoặc bị kẹt (xác thực, handoff) mà không biết bước tiếp theo.

Opportunity:
AI có thể giúp bằng cách gợi ý CÓ KÈM độ tin cậy + đường recovery rõ ràng
(hỏi lại khi không chắc, cho sửa và học lại), giữ human là người quyết cuối.
```

## 6. Evidence đổi SPEC như thế nào?

- [x] Đổi user chính → chốt **user trẻ quản lý chi tiêu trong MoMo** (giao điểm chung của 3 bài).
- [x] Đổi pain statement → từ "AI sai" sang "**AI không lộ độ tin cậy & không có recovery**".
- [x] Đổi build slice → cắt xuống **phân loại giao dịch có confidence + correction học lại**.
- [x] Đổi Auto/Aug decision → chốt **Augmentation** (AI gợi ý, user quyết).
- [x] Đổi 4 paths → Low-confidence và Correction là trọng tâm demo.
- [x] Đổi failure mode → "gán nhầm khoản nhạy cảm (chuyển tiền) thành Mua sắm".
- [ ] Đổi owner/test plan → xem thin SPEC.

```text
Trước evidence, nhóm định: làm một "AI assistant tài chính" rộng (phân tích + khuyên + chatbot).
Sau evidence, nhóm đổi thành: chỉ build MỘT flow — phân loại giao dịch có độ tin cậy + correction học lại —
vì đây là bản thu nhỏ demo được trong 1 ngày của insight chung "lộ confidence + có recovery".
Lý do: cả 3 bài đều chỉ về cùng một gốc lỗi; build flow này chứng minh được nguyên lý mà không phải dựng backend OTP/handoff.
```
