# BẢN PHÊ BÌNH & ĐÁNH GIÁ NĂNG LỰC AI (AI CRITIQUE)
## ĐỒ ÁN: HW02 — DOMAIN TESTING & BOUNDARY VALUE ANALYSIS (ESHOP SUT)

*(Dung lượng: ~270 từ — Tập trung vào 3 luận điểm bắt buộc: Sai sót của AI, Nguyên nhân không tìm được lỗi thật, và Bài học kinh nghiệm)*

---

Trong quá trình thực nghiệm kiểm thử miền trên EShop SUT, việc sử dụng AI bộc lộ 3 vấn đề cốt lõi:

**1. AI sai và thiếu sót ở các lỗ hổng bảo mật và trạng thái ngầm:**  
Cụ thể ở FR-08 (Thanh toán), AI ngây thơ mặc định backend tự tính lại tiền, bỏ qua hoàn toàn kịch bản gian lận tham số (*Parameter Tampering*) khi sửa giá từ 500.000 ₫ thành 1.000 ₫. Ở FR-20, AI thấy nút Hủy bị ẩn trên Mobile khi đơn hàng `shipping` thì vội kết luận Pass, bỏ sót việc gọi API trực tiếp vẫn hủy đơn thành công. Ngoài ra, AI thường gộp nhiều lỗi vào một ca test, vi phạm nguyên tắc Giả định lỗi đơn (*Single Fault Assumption*).

**2. Vì sao AI không phát hiện được lỗi thật:**  
AI không phát hiện được lỗi thật vì nó không chạy trên hệ thống thực tế mà chỉ suy diễn thống kê từ văn bản đặc tả (SRS). Khi đặc tả viết mơ hồ hoặc thiếu chi tiết kỹ thuật, AI bị ảo giác về một "hệ thống lý tưởng", dẫn đến việc không thể biết được lỗi lập trình thực tế như backend dùng toán tử `>` thay vì `>=` cho mã giảm giá, hay tăng bộ đếm 2 đơn vị mỗi lần ở FR-02.

**3. Bài học kinh nghiệm rút ra:**  
Kỹ sư kiểm thử phải prompt theo kỹ thuật chặt chẽ, không prompt chung chung và luôn lấy kết quả của AI làm bản thảo ban đầu. Bắt buộc phải duy trì vai trò kiểm chứng thực nghiệm (*Human-in-the-loop*) trên mã nguồn SUT và cơ sở dữ liệu để tìm ra các bug thực tế của phần mềm.
