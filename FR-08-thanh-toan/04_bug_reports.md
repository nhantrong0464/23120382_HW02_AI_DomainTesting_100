# BÁO CÁO PHÁT HIỆN LỖI PHẦN MỀM (BUG REPORTS / DEFECT LOGS)
## TÍNH NĂNG: FR-08 — THANH TOÁN (CHECKOUT) & LUỒNG LIÊN KẾT TRÊN ESHOP SUT

> **Tiêu chuẩn tài liệu:** Báo cáo lỗi được chuẩn hóa theo tiêu chuẩn quốc tế **IEEE 829** và cấu trúc trường quản lý lỗi chuyên nghiệp của Atlassian Jira.
> **Môi trường thử nghiệm:**
> - Backend: Node.js v20.x, Express, SQLite3 (Chạy tại `http://localhost:5000` hoặc port `3000`).
> - Frontend: Web React/Vite (`http://localhost:5173`), Mobile App React Native Expo.
> - Công cụ kiểm thử: Postman Desktop, Newman CLI, Browser DevTools, SQLite Explorer.

---

## 1. DANH MỤC LỖI ĐÃ PHÁT HIỆN (DEFECT SUMMARY TABLE)

| Mã lỗi (Bug ID) | Tóm tắt lỗi (Summary) | Thành phần bị lỗi | Mức độ nghiêm trọng (Severity) | Mức ưu tiên (Priority) | Trạng thái (Status) |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **BUG-FR08-01** | Lỗ hổng Client Parameter Tampering: Backend không tính lại tổng tiền từ giỏ hàng mà nhận trực tiếp giá trị client gửi lên | Backend (`/api/checkout`) | **Critical (Blocker)** | **P1 (Highest)** | **Open** |
| **BUG-FR08-02** | Vi phạm hậu điều kiện: Giỏ hàng không được làm sạch trong CSDL sau khi thanh toán thành công | Backend (`/api/checkout`) | **High** | **P1 (High)** | **Open** |
| **BUG-FR08-03** | Thiếu kiểm soát giá trị: Hệ thống chấp nhận thanh toán đơn hàng có tổng tiền âm hoặc bằng 0đ | Backend / Validation | **High** | **P2 (Medium)** | **Open** |
| **BUG-FR08-04** | Vi phạm State Machine: Cho phép người dùng hủy đơn hàng đang ở trạng thái vận chuyển (`shipping`) | Backend (`/api/orders/:id/cancel`) | **High** | **P1 (High)** | **Open** |
| **BUG-FR08-05** | Lỗi giá trị biên (Off-by-one): Mã giảm giá bị từ chối khi tổng tiền đạt đúng ngưỡng tối thiểu 300,000 ₫ | Backend (`/api/coupons/apply`) | **Medium** | **P2 (Medium)** | **Open** |
| **BUG-FR08-06** | Thiếu validation độ dài địa chỉ: Cho phép thanh toán với địa chỉ rỗng hoặc địa chỉ vượt quá 255 ký tự | Backend & Web Form | **Medium** | **P3 (Low)** | **Open** |

---

## 2. CHI TIẾT CÁC BÁO CÁO LỖI (DETAILED DEFECT REPORTS)

---

### BUG-FR08-01: Lỗ hổng Client Parameter Tampering - Backend không xác thực tổng tiền
* **Bug ID:** `BUG-FR08-01`
* **Tiêu đề:** Backend chấp nhận số tiền `total_amount` tùy ý do client gửi lên trong payload `POST /api/checkout` mà không tự tính toán lại từ cơ sở dữ liệu giỏ hàng.
* **Thành phần:** `Backend / API Service (server.js: line 125)`
* **Mức độ nghiêm trọng:** `Critical (Lỗ hổng bảo mật tài chính)`
* **Mức độ ưu tiên:** `P1 (Cần khắc phục ngay lập tức trước khi release)`
* **Người phát hiện:** Chuyên viên Kiểm thử (Domain Testing Specialist)
* **Tiền điều kiện (Pre-conditions):**
  - Tài khoản User test đã đăng nhập, sở hữu JWT Token hợp lệ.
  - Giỏ hàng có 2 sản phẩm A (100,000 ₫) và B (400,000 ₫), tổng giá trị thực tế trong DB là 500,000 ₫.
* **Các bước tái hiện (Steps to Reproduce):**
  1. Mở Postman hoặc công cụ can thiệp Proxy (Burp Suite/Fiddler).
  2. Tạo request `POST http://localhost:5000/api/checkout`.
  3. Đính kèm Header `Authorization: Bearer <valid_jwt_token>` và `Content-Type: application/json`.
  4. Sửa đổi Body JSON cố tình gian lận số tiền:
     ```json
     {
       "total_amount": 1000,
       "shipping_address": "123 Le Loi, Quan 1, TP.HCM"
     }
     ```
  5. Gửi request đến máy chủ.
* **Kết quả thực tế (Actual Result):**
  - Backend trả về HTTP `200 OK` (hoặc `201 Created`): `{"message": "Checkout successful", "orderId": 14}`.
  - Truy vấn CSDL `SELECT * FROM orders WHERE id = 14`: Cột `total_amount` lưu đúng giá trị gian lận là **1,000 ₫**!
* **Kết quả mong đợi (Expected Result theo SRS FR-08):**
  - Backend bắt buộc phải tự tính lại tổng tiền từ bảng `cart` trong CSDL.
  - Backend phải từ chối request với mã lỗi HTTP `400 Bad Request` do phát hiện sai lệch dữ liệu, hoặc tự động ghi đè số tiền thực tế là **500,000 ₫** vào bản ghi đơn hàng.
* **Phân tích nguyên nhân kỹ thuật (Code Defect Analysis):**
  Trong file `backend/server.js`:
  ```javascript
  app.post("/api/checkout", authenticateToken, (req, res) => {
    const userId = req.user.id;
    const { total_amount, shipping_address } = req.body; // <-- NHẬN TRỰC TIẾP TỪ CLIENT

    db.run(
      "INSERT INTO orders (user_id, total_amount, status, shipping_address) VALUES (?, ?, ?, ?)",
      [userId, total_amount, "pending", shipping_address], // <-- LƯU VÀO CSDL KHÔNG QUA KIỂM TRA
      function (err) { ... }
    );
  });
  ```
* **Đề xuất khắc phục (Suggested Fix):**
  Truy vấn bảng `cart` và `products` để tính tổng số tiền `const calculatedTotal = SUM(cart.quantity * products.price)`, so sánh với `total_amount` gửi lên hoặc trực tiếp dùng `calculatedTotal` khi INSERT vào bảng `orders`.

---

### BUG-FR08-02: Giỏ hàng không được làm sạch sau khi thanh toán thành công
* **Bug ID:** `BUG-FR08-02`
* **Tiêu đề:** Bảng `cart` của người dùng không được xóa sau khi hoàn tất thanh toán `POST /api/checkout`.
* **Thành phần:** `Backend (server.js: /api/checkout)`
* **Mức độ nghiêm trọng:** `High (Lỗi logic nghiệp vụ)`
* **Mức độ ưu tiên:** `P1`
* **Tiền điều kiện:** User đăng nhập, giỏ hàng có sản phẩm.
* **Các bước tái hiện:**
  1. Thêm 1 sản phẩm vào giỏ hàng (`POST /api/cart`).
  2. Thực hiện thanh toán thành công (`POST /api/checkout`). Nhận phản hồi "Checkout successful".
  3. Gọi lại API xem giỏ hàng: `GET /api/cart`.
* **Kết quả thực tế:**
  - API `GET /api/cart` vẫn trả về danh sách các sản phẩm vừa mua.
  - Trên giao diện Web/Mobile, người dùng thấy giỏ hàng vẫn còn nguyên sản phẩm, nếu bấm thanh toán tiếp sẽ bị trừ tiền lần hai.
* **Kết quả mong đợi:**
  - Ngay sau khi INSERT thành công vào bảng `orders`, backend phải chạy lệnh `DELETE FROM cart WHERE user_id = ?` trong cùng một transaction.
* **Phân tích mã nguồn:** Trong handler `app.post("/api/checkout")`, sau lệnh `INSERT INTO orders` hoàn toàn không có lệnh xóa bảng `cart`.

---

### BUG-FR08-03: Chấp nhận thanh toán đơn hàng có tổng tiền âm hoặc bằng 0
* **Bug ID:** `BUG-FR08-03`
* **Tiêu đề:** Hệ thống không kiểm tra tính hợp lệ của số tiền thanh toán, cho phép tạo đơn với `total_amount <= 0`.
* **Thành phần:** `Backend / Input Validation`
* **Mức độ nghiêm trọng:** `High`
* **Mức độ ưu tiên:** `P2`
* **Các bước tái hiện:**
  1. Gửi request `POST /api/checkout` với body:
     ```json
     {
       "total_amount": -50000,
       "shipping_address": "Test Negative Amount"
     }
     ```
* **Kết quả thực tế:**
  - Hệ thống trả về `200 OK`, lưu đơn hàng với giá `-50000` vào CSDL.
* **Kết quả mong đợi:**
  - Backend phải kiểm tra `if (!total_amount || total_amount <= 0)` và trả về HTTP `400 Bad Request`.

---

### BUG-FR08-04: Vi phạm State Machine - Cho phép hủy đơn hàng đang giao (`shipping`)
* **Bug ID:** `BUG-FR08-04`
* **Tiêu đề:** Khách hàng có thể gọi API `PUT /api/orders/:id/cancel` để tự ý hủy đơn hàng đang ở trạng thái `shipping`.
* **Thành phần:** `Backend (server.js: line 145)`
* **Mức độ nghiêm trọng:** `High`
* **Mức độ ưu tiên:** `P1`
* **Mô tả chi tiết:**
  Theo SRS FR-10 và FR-20, đơn hàng chỉ được phép hủy khi ở trạng thái `pending` hoặc `confirmed`. Khi chuyển sang `shipping`, hàng đã rời kho và khách không được tự hủy.
  Tuy nhiên, trong `server.js`:
  ```javascript
  // Lẽ ra phải là: if (order.status !== 'pending' && order.status !== 'confirmed')
  if (order.status === "delivered" || order.status === "canceled") {
    return res.status(400).json({ error: "Cannot cancel this order." });
  }
  ```
  Điều kiện `if` chỉ chặn khi `delivered` hoặc `canceled`, do đó đơn hàng `shipping` lọt qua và bị chuyển thành `canceled` thành công!
* **Kết quả thực tế:** Gọi `PUT /api/orders/5/cancel` khi đơn ở trạng thái `shipping` thành công (HTTP 200).
* **Kết quả mong đợi:** Trả về HTTP `400 Bad Request`: "Không thể hủy đơn hàng đang vận chuyển".

---

### BUG-FR08-05: Lỗi biên Off - Từ chối coupon khi đơn hàng đạt đúng ngưỡng tối thiểu
* **Bug ID:** `BUG-FR08-05`
* **Tiêu đề:** Mã giảm giá `SAVE10` (yêu cầu đơn từ 300,000 ₫) bị từ chối khi tổng tiền đơn hàng bằng đúng 300,000 ₫.
* **Thành phần:** `Backend (server.js: /api/coupons/apply)`
* **Mức độ nghiêm trọng:** `Medium (Off-by-one Defect)`
* **Mức độ ưu tiên:** `P2`
* **Các bước tái hiện:**
  1. Thêm sản phẩm vào giỏ để tổng tiền đạt đúng $300,000$ ₫.
  2. Gửi request áp dụng mã: `POST /api/coupons/apply` với `code: "SAVE10"`, `total_amount: 300000`.
* **Kết quả thực tế:**
  - Hệ thống báo lỗi: "Đơn hàng chưa đạt giá trị tối thiểu".
* **Phân tích mã nguồn:**
  Mã nguồn sử dụng toán tử lớn hơn nghiêm ngặt (`>`):
  ```javascript
  if (total_amount > coupon.min_order_amount) // Sai lầm! Lẽ ra phải là >=
  ```
  Khi `total_amount = 300000` và `min_order_amount = 300000`, điều kiện trả về `false`, khiến khách hàng bị từ chối mã giảm giá oan uổng.

---

### BUG-FR08-06: Thiếu kiểm tra độ dài địa chỉ nhận hàng (`shipping_address`)
* **Bug ID:** `BUG-FR08-06`
* **Tiêu đề:** Hệ thống chấp nhận chuỗi địa chỉ rỗng `""` hoặc chuỗi địa chỉ vượt quá 255 ký tự.
* **Thành phần:** `Backend / Validation`
* **Mức độ nghiêm trọng:** `Medium`
* **Mức độ ưu tiên:** `P3`
* **Mô tả:** Trường `shipping_address` không có kiểm tra độ dài $1 \le L \le 255$. Người dùng có thể checkout mà không cần nhập địa chỉ, hoặc nhập chuỗi ký tự cực dài làm vỡ layout giao diện trên ứng dụng di động Mobile React Native.
