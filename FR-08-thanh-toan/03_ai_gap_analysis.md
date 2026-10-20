# PHÂN TÍCH KHOẢNG CÁCH NĂNG LỰC AI (AI GAP ANALYSIS)
## TÍNH NĂNG: FR-08 — THANH TOÁN (CHECKOUT) TRÊN ESHOP SUT

---

## 1. MỤC TIÊU VÀ PHƯƠNG PHÁP ĐỐI SÁNH

Tài liệu này thực hiện đối sánh, phân tích chuyên sâu khoảng cách (**Gap Analysis**) giữa:
1. **Dữ liệu do Mô hình Trí tuệ Nhân tạo (AI) tạo ra ban đầu** khi nhận prompt kiểm thử miền thuần túy.
2. **Tiêu chuẩn Kiểm thử Miền Chuyên gia (Human Expert Domain Testing Standards)** dựa trên kiến trúc thực tế của mã nguồn SUT và lý thuyết kiểm thử hàn lâm.

Mục tiêu cốt lõi là vạch rõ các **điểm mù (Blindspots)**, **ảo giác logic (Hallucinations)** và **thiếu sót nghiệm thu (Acceptance Criteria Gaps)** của AI để đưa ra cơ chế kiểm định an toàn trong công tác kiểm thử phần mềm.

---

## 2. MA TRẬN ĐỐI SÁNH KẾT QUẢ: AI BAN ĐẦU VS. CHUYÊN GIA KIỂM THỬ

| Tiêu chí kiểm định | Kết quả AI sinh ban đầu (Initial AI Output) | Kết quả Chuyên gia hiệu đính (Audited / Expert Output) | Khoảng cách & Mức độ rủi ro (Risk Gap) |
| :--- | :--- | :--- | :--- |
| **Nguyên tắc Single Fault Assumption** | **Bị vi phạm một phần.** AI có xu hướng tạo test case gộp: vừa gửi địa chỉ rỗng vừa không truyền token, hoặc vừa sai mã coupon vừa giá dưới ngưỡng. | **Tuân thủ nghiêm ngặt 100%.** Mỗi test case chỉ kích hoạt duy nhất một biến ở lớp không hợp lệ, giữ nguyên toàn bộ các biến khác ở trạng thái hợp lệ. | **Trung bình:** Khi test thất bại, test case của AI không thể cô lập nguyên nhân do biến nào gây ra. |
| **Tính toàn vẹn giá tiền (Parameter Tampering)** | **Bị bỏ qua hoàn toàn.** AI chỉ kiểm tra các giá trị tiền bình thường trên giao diện (ví dụ nhập số dương, nhập chữ). AI không nghĩ ra kịch bản client can thiệp gói tin POST gửi giá 1,000 ₫ thay vì 500,000 ₫. | **Được xác định là kịch bản bảo mật số 1 (TC_FR08_10).** Đặt yêu cầu kiểm thử backend có tự tính lại giá từ DB hay không. | **Cực kỳ nguy hiểm (Critical):** Bỏ sót lỗ hổng bảo mật gian lận thanh toán kinh điển trong ứng dụng thương mại điện tử. |
| **Hậu điều kiện xóa giỏ hàng (Post-condition: Cart Cleanup)** | **Chỉ kiểm tra HTTP Response.** AI chỉ assert `response.status === 200` và `message === "Checkout successful"`, bỏ qua trạng thái CSDL. | **Bắt buộc kiểm tra CSDL giỏ hàng.** Assert `SELECT count(*) FROM cart WHERE user_id = ?` phải bằng 0 sau khi thanh toán. | **Cao (High):** Hệ thống thực tế (SUT) quên xóa giỏ hàng nhưng AI vẫn đánh dấu PASS vì chỉ nhìn HTTP 200. |
| **Xác định điểm biên mở (Open Boundary BVA)** | **Nhầm lẫn định nghĩa.** Đối với điều kiện $M > 0$, AI nhầm lẫn xem $M = 1$ là On-point và $M = 0$ là Off-point, hoặc xem $M = 0$ là giá trị hợp lệ. | **Chuẩn hóa toán học.** Ranh giới tại $M = 0$. On-point là $M = 0$ (thuộc miền không hợp lệ); Off-point là $M = 1$ (thuộc miền hợp lệ). | **Thấp - Trung bình:** Sai lệch về mặt thuật ngữ hàn lâm kiểm thử phần mềm. |
| **Bẫy lỗi toán tử so sánh Coupon ($>$ vs $\ge$)** | **Bỏ qua điểm biên $T = 300,000$ ₫.** AI chỉ thử các giá trị cách xa biên như $T = 200,000$ ₫ (FAIL) và $T = 500,000$ ₫ (PASS). | **Nhắm trúng điểm biên nhạy cảm $T = 300,000$ ₫.** Phát hiện ngay lập tức lỗi backend dùng `total_amount > min_order_amount` thay vì `>=`. | **Cao (High):** Bỏ lọt lỗi nghiệp vụ khiến khách hàng mua đúng 300k không được giảm giá. |
| **Máy trạng thái hủy đơn (State Machine FR-10)** | **Chỉ kiểm tra luồng đơn giản.** AI kiểm tra hủy đơn khi `pending` (thành công) và `delivered` (thất bại). | **Bổ sung trạng thái ranh giới `shipping`.** Kiểm tra nút Hủy bị ẩn trên Mobile và API chặn trả về 400. Phát hiện SUT cho phép hủy khi đang giao hàng. | **Cao (High):** Bỏ qua trạng thái chuyển tiếp then chốt trong chuỗi cung ứng thương mại điện tử. |

---

## 3. PHÂN TÍCH NGUYÊN NHÂN GỐC RỄ (ROOT CAUSE OF AI GAPS)

### 3.1. Căn bệnh "Niềm tin ngây thơ vào Backend" (Backend Trust Hallucination)
Mô hình AI huấn luyện trên hàng triệu đoạn văn bản lý thuyết và tài liệu thiết kế lý tưởng. Do đó, AI có xu hướng mặc định rằng *"Backend hiện đại luôn có middleware tự động tính toán lại giá tiền, tự động rollback transaction và tự động làm sạch giỏ hàng"*. AI không thể hình dung rằng trong mã nguồn thực tế (như file `backend/server.js` của EShop SUT):
```javascript
// Lỗi thực tế trong SUT: Nhận thẳng total_amount từ req.body mà không hề tính lại!
const { total_amount, shipping_address } = req.body;
db.run("INSERT INTO orders (user_id, total_amount, status, shipping_address) VALUES (?, ?, ?, ?)",
  [userId, total_amount, "pending", shipping_address], ...);
```
Chính ảo giác này khiến AI không tự giác sinh ra test case kiểm tra gian lận giá tiền nếu kỹ sư kiểm thử không ép buộc trong prompt.

### 3.2. Tư duy kiểm thử "Hộp đen bề mặt" (Surface-level Black-box Bias)
AI chủ yếu suy luận theo hướng người dùng thông thường thao tác trên giao diện đồ họa (UI Testing). Tuy nhiên, các lỗi bảo mật và lỗi toàn vẹn nghiệp vụ lại xảy ra ở tầng giao tiếp API và cơ sở dữ liệu. Khi không được yêu cầu kiểm thử tích hợp (Integration / API / DB Checks), AI sẽ bỏ qua các hậu điều kiện (Post-conditions) như xóa bảng `cart` hay cập nhật tồn kho.

### 3.3. Hạn chế trong tư duy phân tích toán học biên mở (Open Boundary Topology)
Khi xử lý các bất đẳng thức nghiêm ngặt ($x > a$), AI thường gặp khó khăn trong việc phân biệt giữa khái niệm "điểm nằm trên đường biên hình học" (Boundary Surface) và "điểm thuộc tập nghiệm hợp lệ" (Feasible Region). Điều này dẫn đến sự không nhất quán giữa các câu trả lời nếu không có hướng dẫn định chuẩn.

---

## 4. CHIẾN LƯỢC ĐÓNG KHOẢNG CÁCH (PROMPT ENGINEERING & HARNESS STRATEGY)

Để đóng hoàn toàn các khoảng cách (Gaps) đã chỉ ra ở trên, bộ quy chuẩn tác nghiệp cho AI Agent (Agent Skill) cần được bổ sung 4 điều kiện ràng buộc bắt buộc (Hard Constraints):

1. **Ràng buộc Single Fault tuyệt đối:** Mọi bảng test case sinh ra phải có cột kiểm tra ma trận lớp tương đương; từ chối mọi trường hợp có từ 2 giá trị không hợp lệ trở lên trong cùng một ca kiểm thử.
2. **Ràng buộc kiểm thử gian lận tham số (Parameter Tampering Rule):** Luôn tự động bổ sung ca kiểm thử can thiệp payload HTTP gửi giá trị không đồng nhất giữa Client và Server.
3. **Ràng buộc kiểm tra hậu điều kiện (State & DB Verification):** Mọi ca kiểm thử thành công phải định nghĩa rõ ràng trạng thái CSDL trước và sau khi thực thi (Pre-conditions & Post-conditions).
4. **Quy tắc biên 3 điểm (3-Value Boundary Rule):** Đối với mọi biến số có điều kiện so sánh, bắt buộc sinh đủ bộ 3 điểm: On-point, Off-point và In-point.
