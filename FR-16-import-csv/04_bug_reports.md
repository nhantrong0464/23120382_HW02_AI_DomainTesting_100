# BÁO CÁO PHÁT HIỆN LỖI PHẦN MỀM (BUG REPORTS)
## TÍNH NĂNG: FR-16 — IMPORT SẢN PHẨM TỪ CSV (POOL C)

---

## 1. DANH MỤC LỖI PHÁT HIỆN

| Bug ID | Tóm tắt lỗi | Thành phần | Severity | Priority | Trạng thái |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **BUG-FR16-01** | Thiếu kiểm tra quyền Admin: User thông thường vẫn có thể gọi API Import sản phẩm | Backend (`/api/admin/import-products`) | **Critical** | **P1** | **Open** |
| **BUG-FR16-02** | Vi phạm tính nguyên tử: Hệ thống chèn từng dòng đơn lẻ, không Rollback khi gặp dòng lỗi | Backend (`server.js: line 208`) | **High** | **P1** | **Open** |
| **BUG-FR16-03** | Cho phép Import sản phẩm với giá tiền âm hoặc bằng 0đ | Backend / Database | **High** | **P2** | **Open** |

---

## 2. CHI TIẾT CÁC BÁO CÁO LỖI

### BUG-FR16-01: Thiếu phân quyền Role Admin trong API Import
* **Mô tả:** Trong `backend/server.js`:
  ```javascript
  app.post("/api/admin/import-products", authenticateToken, (req, res) => { ... });
  ```
  API chỉ sử dụng middleware `authenticateToken` để xác thực đăng nhập, nhưng hoàn toàn không kiểm tra `if (req.user.role !== 'admin') return res.status(403)`. Do đó, bất kỳ tài khoản khách hàng thông thường nào có Token đều có thể upload file CSV để chèn hàng nghìn sản phẩm giả mạo vào hệ thống.
* **Kỳ vọng:** Trả về HTTP 403 Forbidden nếu `role !== 'admin'`.

### BUG-FR16-02: Không có Database Transaction (Atomic Rollback)
* **Mô tả:** Code SUT duyệt vòng lặp `rows.forEach((row, index) => { stmt.run(...) })` mà không bao bọc trong `db.run("BEGIN TRANSACTION")` và `ROLLBACK`. Nếu một file có 10 dòng mà dòng thứ 5 bị lỗi, 4 dòng đầu vẫn được chèn thành công vào CSDL SQLite, gây phân mảnh và sai lệch dữ liệu kho hàng.
