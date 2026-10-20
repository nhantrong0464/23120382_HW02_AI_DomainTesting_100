# THƯ MỤC HÌNH ẢNH MINH CHỨNG KIỂM THỬ (DEFECT EVIDENCE SCREENSHOTS)
## TÍNH NĂNG: FR-08 — THANH TOÁN (CHECKOUT) TRÊN ESHOP SUT

---

## 1. MỤC ĐÍCH & QUY ĐỊNH ĐẶT TÊN ẢNH

Thư mục này lưu trữ các hình ảnh chụp màn hình (Screenshots) minh chứng thực tế cho quá trình thực thi kiểm thử miền (Domain Testing) và kiểm thử giá trị biên (BVA) trên hệ thống EShop SUT. Mọi hình ảnh phải được đối chiếu trực tiếp với các mã lỗi (Bug IDs) trong file `04_bug_reports.md`.

### Quy chuẩn đặt tên tệp ảnh:
* `BUG_[ID]_[Tên_Lỗi]_[Giao_Diện].png` (Ví dụ: `BUG_FR08_01_tamper_price_postman.png`)
* `TC_[ID]_[Trạng_Thái].png` (Ví dụ: `TC_FR08_01_checkout_success.png`)

---

## 2. BẢNG ĐỐI CHIẾU HÌNH ẢNH VÀ MÃ LỖI THỰC TẾ

| STT | Tên tệp ảnh minh chứng | Mã lỗi / Test ID liên kết | Mô tả nội dung hình ảnh |
| :---: | :--- | :---: | :--- |
| **1** | `BUG_FR08_01_price_tampering.png` | `BUG-FR08-01` / `TC_FR08_10` | Chụp màn hình Postman gửi request `POST /api/checkout` với `total_amount = 1000`, backend trả về `200 OK` và CSDL lưu đúng 1.000 ₫. |
| **2** | `BUG_FR08_02_cart_not_cleared.png` | `BUG-FR08-02` / `TC_FR08_01` | Màn hình giao diện giỏ hàng sau khi bấm thanh toán thành công: Sản phẩm vẫn còn nguyên trong giỏ hàng thay vì bị xóa sạch. |
| **3** | `BUG_FR08_03_negative_amount.png` | `BUG-FR08-03` / `TC_FR08_06` | Postman gửi request số tiền âm `-50000`, backend chấp nhận lưu đơn hàng số tiền âm thành công. |
| **4** | `BUG_FR08_04_cancel_shipping_order.png`| `BUG-FR08-04` / `BVA_ST_02` | Màn hình gọi API `PUT /api/orders/:id/cancel` khi đơn hàng có trạng thái `shipping`, hệ thống vẫn cho phép chuyển sang `canceled`. |
| **5** | `BUG_FR08_05_coupon_boundary_reject.png`| `BUG-FR08-05` / `BVA_CP_01` | Màn hình giao diện Mobile / Web báo Toast đỏ: "Đơn hàng chưa đủ điều kiện" khi tổng tiền đúng bằng ngưỡng $300,000$ ₫. |
| **6** | `BUG_FR08_06_empty_address_overflow.png`| `BUG-FR08-06` / `BVA_L_01` | Giao diện cho phép submit địa chỉ rỗng mà không có thông báo validation lỗi. |

---

## 3. HƯỚNG DẪN BỔ SUNG MINH CHỨNG
Khi quay video demo hoặc thực thi kiểm thử thủ công bổ sung trên trình duyệt, các ảnh chụp màn hình tương ứng được lưu trực tiếp vào thư mục này để phục vụ công tác thanh tra chất lượng (Quality Audit).
