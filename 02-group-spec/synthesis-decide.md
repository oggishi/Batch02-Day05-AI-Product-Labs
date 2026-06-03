# Synthesis & Decide — Từ Evidence Đến Build Slice

> Dùng sau khi nhóm đã có evidence (xem `evidence-pack.md`). Mục tiêu: chốt một build slice đủ nhỏ cho Day 06.

## 1. Gom evidence thành cụm

Gom theo **workflow/pain**, không theo tên feature.

| Cụm pain | Evidence thuộc cụm | Thành viên |
|---|---|---|
| **"AI kết luận mà không cho biết chắc tới đâu"** | Phân loại giao dịch không lộ confidence; con số chi tiêu không nói chỉ-trong-MoMo; lời khuyên/đáp chat không dẫn nguồn | Trân |
| **"AI khuyên/hành động mà không kiểm điều kiện"** | Gợi ý đầu tư không hỏi suitability; gửi OTP mà không kiểm trạng thái gửi | Đạt |
| **"AI nói đã làm xong nhưng không có đường sửa / không có trạng thái thật"** | Báo "đã chuyển CSKH" nhưng không handoff; sửa nhãn rồi không học; user không nhận SMS mà không có phương án thay thế; gửi báo cáo email mà không hiển thị trạng thái gửi (sent/delivered/failed) hay phương án backup |  Đạt , Trân |

→ Cả 3 cụm quy về **một gốc**: *thiếu tín hiệu tin cậy (confidence/nguồn/trạng thái) + thiếu đường recovery.*

## 2. Viết insight

```text
User trẻ dùng MoMo (segment) không chỉ cần AI trả lời nhanh / tự gán nhãn (surface need).
Họ thật ra cần biết KHI NÀO nên tin AI và có đường sửa khi AI sai (deeper need),
vì 3 bề mặt AI khác nhau (phân tích chi tiêu, gợi ý đầu tư, chatbot hỗ trợ) đều lặp lại
cùng một lỗi: AI tự tin nhưng giấu độ tin cậy/nguồn/trạng thái và không có recovery (evidence pattern).
```

## 3. Viết opportunity

```text
Cơ hội là dùng AI để gợi ý CÓ KÈM độ tin cậy + đường recovery rõ ràng
(hỏi lại khi không chắc, cho sửa và học lại),
giúp user kiểm soát và tin đúng chỗ thay vì tin mù hoặc bị kẹt,
trong khi vẫn chặn được lỗi gán nhầm/over-claim làm sai lệch dữ liệu tài chính.
```

## 4. Chọn build slice

**Build slice được chọn:** *Flow gửi báo cáo chi tiêu qua email từ chatbot, tạo PDF report, hiển thị trạng thái gửi thật và recovery options.*

| Câu hỏi | Đạt khi | Trạng thái |
|---|---|---|
| User cụ thể chưa? | User trẻ muốn biết "Tháng này tôi chi bao nhiêu" và nhận báo cáo qua email. | ✅ |
| Task đủ hẹp chưa? | Tạo báo cáo chi tiêu tháng, render chart + insight, xuất PDF, gửi email, hiển thị trạng thái. | ✅ |
| AI decision rõ chưa? | AI xác định intent, tổng hợp giao dịch, tạo insight, quyết định gửi email report. | ✅ |
| Failure path rõ chưa? | Email sai / gửi thất bại / email chưa xác thực → cần recovery + cho sửa lại. | ✅ |
| Có evidence không? | Case email gửi báo cáo, case SMS/CSKH over-claim, analog Gmail status + mock transaction data. | ✅ |

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

**Vì sao chọn slice này thay vì OTP-recovery hay nudge-đầu tư:**
- Demo được với backend status mock / logic giả lập, phù hợp Day 06.
- Là cải tiến trực tiếp trên workflow đã có: báo cáo chi tiêu + gửi email, thêm status thật và recovery.
- Mọi thành viên đóng góp: evidence case bot (Bảo Trân), report/PDF/status logic (Đạt), integration/demo (Thành Đạt).

## 5. Quyết định: giữ, giảm scope, hay đổi hướng?

| Tình huống | Áp dụng cho nhóm? | Quyết định |
|---|---|---|
| Evidence yếu, user mơ hồ | Không — user & pain đã rõ | Giữ |
| Ý tưởng quá rộng | Có — ban đầu định làm "AI tài chính" rộng | **Giữ domain MoMo, cắt xuống 1 flow gửi email** |
| AI không cần thiết | Không — gửi trạng thái email cần AI/hệ thống | Giữ AI ở mức augmentation |
| Rủi ro cao | Một phần (nudge đầu tư + OTP/handoff) | **Đưa nudge đầu tư + OTP/handoff vào backlog**, chọn flow email rủi ro thấp |
| Không demo được trong 1 ngày | OTP/handoff không demo nổi | **Backlog**, chọn flow email có thể mock được |

## 6. Câu chốt cuối

```text
Dựa trên evidence rằng AI của MoMo tuyên bố đã hành động (gửi SMS, gửi email, chuyển CSKH) 
mà không hiển thị trạng thái thật và không có recovery options,
nhóm sẽ build prototype flow gửi báo cáo chi tiêu qua email từ chatbot,
cho user trẻ đang muốn kiểm tra & chia sẻ chi tiêu hàng tháng,
để giải quyết pain "user chờ email mà không biết đã gửi hay không, bị kẹt",
bằng cách: hiển thị trạng thái gửi thật (pending/sent/failed) + luôn cung cấp recovery options (tải app, SMS, gửi lại),
và sẽ test failure path "email chưa xác thực / gửi thất bại" để chứng minh recovery hoạt động.
```

## 7. Backlog

Những thứ **không build trong Day 06** (giữ làm hướng mở rộng):

- Phân loại giao dịch chi tiêu có kèm độ tin cậy + correction học lại (Đạt — F1, insight chung song khác flow).
- Mù dữ liệu ngoài MoMo: thêm giao dịch tiền mặt/ngân hàng khác thủ công + hiển thị độ phủ dữ liệu (Đạt — F1).
- Trừ tiền hoàn/chi hộ khỏi tổng chi tiêu (net vs gross) (Đạt — điểm gãy 4).
- Suitability-check + disclosure rủi ro cho gợi ý đầu tư (Thành Đạt — F1, rủi ro cao).
- Cảnh báo ngân sách tại điểm-ra-quyết-định thay vì sau khi đã tiêu (Thành Đạt — F2).
- Trạng thái OTP thật + recovery khi không nhận SMS (Bảo Trân — F1, insight chung song khác flow).
- Human handoff minh bạch (chỉ báo "đang chuyển CSKH" khi backend xác nhận) (Bảo Trân — F2, insight chung song khác flow).
