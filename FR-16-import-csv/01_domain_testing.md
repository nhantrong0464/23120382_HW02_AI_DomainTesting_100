# BÁO CÁO KIỂM THỬ MIỀN (DOMAIN TESTING REPORT)
## TÍNH NĂNG: FR-16 — IMPORT SẢN PHẨM TỪ CSV (POOL C — WEB ADMIN)

---

## 1. TỔNG QUAN & ĐẶC TẢ NGHIỆP VỤ (SRS FR-16)
* **Thuộc nhóm tính năng:** Pool C — Web Admin.
* **Mô tả nghiệp vụ:**
  1. Chỉ tài khoản Quản trị viên (`role: "admin"`) mới có quyền truy cập và thực thi chức năng Import sản phẩm từ file CSV.
  2. Định dạng file hợp lệ: Đuôi mở rộng `.csv`, mã hóa UTF-8 chuẩn RFC 4180.
  3. Cấu trúc các cột bắt buộc: `name` (Tên SP), `price` (Giá tiền), `category_id` (Mã danh mục), `description` (Mô tả), `imageUrl` (Ảnh).
  4. Ràng buộc trường dữ liệu:
     - `name`: Không được rỗng, độ dài từ 1 đến 100 ký tự.
     - `price`: Bắt buộc là số dương ($P > 0$ ₫).
     - `category_id`: Bắt buộc phải là ID danh mục đã tồn tại trong CSDL.
  5. Tính nguyên tử (Atomic Transaction): Nếu bất kỳ một dòng nào trong file CSV bị lỗi, toàn bộ quá trình Import phải bị hủy bỏ và Rollback 100%, không để lại dữ liệu rác trong hệ thống.

---

## 2. LIỆT KÊ BIẾN ĐẦU VÀO VÀ MIỀN GIÁ TRỊ

| Tên biến | Kiểu dữ liệu | Miền giá trị lý thuyết | Ý nghĩa |
| :--- | :---: | :--- | :--- |
| `user_role` ($R$) | Enum | $\{ \text{'admin'}, \text{'user'}, \text{'guest'} \}$ | Quyền hạn của người thực hiện Import. |
| `file_extension` ($EXT$) | String | Chuỗi định dạng đuôi file | Định dạng tệp tin tải lên (`.csv`, `.xlsx`, `.txt`). |
| `product_name` ($N$) | String | Độ dài chuỗi $L_N \in \mathbb{N}$ | Tên sản phẩm trong từng hàng của file CSV. |
| `product_price` ($P$) | Real | $(-\infty, +\infty)$ | Đơn giá sản phẩm (₫). |
| `category_id` ($CAT$) | Integer | $\mathbb{Z}^+$ | ID danh mục liên kết. |

---

## 3. PHÂN CHIA LỚP TƯƠNG ĐƯƠNG & GIẢI THÍCH LÝ DO

### 3.1. Biến quyền người dùng `user_role` ($R$)
* **$R_1$ (Hợp lệ):** Có token của tài khoản `admin`.
* **$R_2$ (Không hợp lệ):** Tài khoản người dùng thường (`role: "user"`) hoặc chưa đăng nhập.

### 3.2. Biến định dạng file `file_extension` ($EXT$)
* **$EXT_1$ (Hợp lệ):** File định dạng `.csv`.
* **$EXT_2$ (Không hợp lệ):** File định dạng khác (`.xlsx`, `.exe`, `.pdf`).

### 3.3. Biến tên sản phẩm `product_name` ($N$)
* **$N_1$ (Hợp lệ):** $1 \le L_N \le 100$ ký tự.
* **$N_2$ (Không hợp lệ):** $L_N = 0$ (Để trống tên).

### 3.4. Biến giá sản phẩm `product_price` ($P$)
* **$P_1$ (Hợp lệ):** $P > 0$ ₫.
* **$P_2$ (Không hợp lệ):** $P = 0$ ₫.
* **$P_3$ (Không hợp lệ):** $P < 0$ ₫.

---

## 4. BẢNG TEST CASES DOMAIN TESTING (SINGLE FAULT ASSUMPTION)

| Test ID | Lớp tương đương | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected Result) | Kết quả thực tế trên SUT (Actual Result) | Trạng thái (Pass/Fail) |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC_FR16_01** | $R_1, EXT_1, N_1, P_1$ | Role Admin, file CSV 5 dòng hợp lệ | Import thành công 5/5 sản phẩm (HTTP 200 OK). | Import hoàn tất 5/5 sản phẩm. | **PASS** |
| **TC_FR16_02** | **$R_2$ (User thường)** | Role User thường gọi API Import | HTTP 403 Forbidden: "Chỉ Admin mới có quyền". | API trả về **HTTP 200 OK, cho phép User thường import**! | **FAIL (BUG-FR16-01)** |
| **TC_FR16_03** | **$EXT_2$ (Sai file)** | File `products.xlsx` | Từ chối file, báo lỗi định dạng hỗ trợ. | Giao diện chặn định dạng file thành công. | **PASS** |
| **TC_FR16_04** | **$N_2$ (Thiếu tên SP)** | File CSV dòng 2 có `name: ""` | Báo lỗi dòng 2 thiếu tên, từ chối import. | Báo lỗi Hàng 2: Thiếu tên sản phẩm. | **PASS** |
| **TC_FR16_05** | **$P_2$ (Giá = 0)** | File CSV dòng 3 có `price: 0` | Báo lỗi: "Giá sản phẩm phải lớn hơn 0". | Backend chấp nhận insert sản phẩm giá 0đ vào DB. | **FAIL (BUG-FR16-03)** |
| **TC_FR16_06** | **$P_3$ (Giá âm)** | File CSV dòng 3 có `price: -100000` | Báo lỗi: "Giá sản phẩm không được là số âm". | Backend chấp nhận insert sản phẩm giá -100,000 ₫! | **FAIL (BUG-FR16-03)** |
| **TC_FR16_07** | Atomic Rollback | File 5 dòng: dòng 1-3 hợp lệ, dòng 4 lỗi | Hủy toàn bộ, Rollback 100%, không chèn dòng nào. | **Dòng 1-3 vẫn bị chèn vào DB**, chỉ dòng 4-5 lỗi! | **FAIL (BUG-FR16-02)** |
