# PHÂN TÍCH KHOẢNG CÁCH NĂNG LỰC AI (AI GAP ANALYSIS)
## TÍNH NĂNG: PHÂN HỆ MOBILE (FR-20) — THANH TOÁN & HỦY ĐƠN (POOL D)

---

## 1. SO SÁNH KẾT QUẢ AI VÀ TIÊU CHUẨN KIỂM THỬ THỰC TẾ

| Tiêu chí | Kết quả AI sinh ban đầu | Yêu cầu Chuyên gia / Thực tế SUT | Khoảng cách & Đánh giá |
| :--- | :--- | :--- | :--- |
| **Bất đồng bộ giữa Mobile UI và Backend API** | AI chỉ kiểm tra giao diện Mobile thấy nút Hủy bị ẩn khi đơn hàng `shipping` thì kết luận PASS. | Chuyên gia kiểm thử dùng Postman gửi trực tiếp `PUT /orders/:id/cancel` khi đơn ở trạng thái `shipping`. | **Cực kỳ nguy hiểm:** Mobile UI ẩn nút nhưng Backend lại cho phép hủy! Đây là lỗi bảo mật "Tin tưởng giao diện phía client" kinh điển mà AI bỏ lọt. |
| **Kiểm tra biên đóng của Coupon** | AI chỉ thử các giá trị cách xa biên ($T=200k$ và $T=500k$). | Bắt buộc kiểm tra chính xác tại biên On-point $T = 300,000$ ₫. | **Cao:** Backend dùng `if (total_amount > coupon.min_order_amount)` nên khách mua đúng 300k trên Mobile bị từ chối mã. |
| **Làm sạch trạng thái State Giỏ hàng sau Checkout** | AI chỉ nhìn thấy màn hình "Đặt hàng thành công". | Kiểm tra biến state `cart` trong React Native AsyncStorage / RAM. | **Trung bình:** Khi user quay lại tab Giỏ hàng, các món hàng cũ vẫn hiển thị nếu component không unmount. |
