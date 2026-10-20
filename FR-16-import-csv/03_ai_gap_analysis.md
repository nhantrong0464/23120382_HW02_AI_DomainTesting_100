# PHÂN TÍCH KHOẢNG CÁCH NĂNG LỰC AI (AI GAP ANALYSIS)
## TÍNH NĂNG: FR-16 — IMPORT SẢN PHẨM TỪ CSV (POOL C)

---

## 1. SO SÁNH KẾT QUẢ AI VÀ TIÊU CHUẨN KIỂM THỬ THỰC TẾ

| Tiêu chí | Kết quả AI sinh ban đầu | Yêu cầu Chuyên gia / Thực tế SUT | Khoảng cách & Đánh giá |
| :--- | :--- | :--- | :--- |
| **Kiểm tra Phân quyền (Access Control)** | AI chỉ tập trung vào cấu trúc file CSV và lỗi định dạng. | Bắt buộc kiểm tra token của `role: "user"` khi gọi API `/api/admin/import-products`. | **Nghiêm trọng:** SUT thực tế quên kiểm tra role admin trong middleware, cho phép user thường import sản phẩm! |
| **Tính nguyên tử giao dịch (Atomic Rollback)** | AI chỉ kiểm tra tỷ lệ đếm `inserted/total` (ví dụ thông báo "Import 3/5 sản phẩm"). | Kiểm tra nguyên tắc All-or-Nothing của cơ sở dữ liệu: Nếu 1 dòng lỗi, 4 dòng còn lại phải bị ROLLBACK hoàn toàn. | **Cao:** Code SUT thực tế insert từng dòng đơn lẻ, để lại dữ liệu dở dang trong DB. |
| **Kiểm soát kiểu dữ liệu giá tiền** | AI kiểm tra chuỗi chữ, nhưng bỏ qua số âm `-50000` trong file CSV. | Bắt buộc kiểm tra $P \le 0$ để ngăn chặn đưa sản phẩm giá âm vào hệ thống. | **Cao:** SUT chấp nhận insert sản phẩm giá âm vào bảng `products`. |
