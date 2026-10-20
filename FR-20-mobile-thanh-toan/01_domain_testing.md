# BÁO CÁO KIỂM THỬ MIỀN (DOMAIN TESTING REPORT)
## TÍNH NĂNG: PHÂN HỆ MOBILE (FR-20) — LUỒNG THANH TOÁN & MÁY TRẠNG THÁI HỦY ĐƠN (POOL D)

---

## 1. TỔNG QUAN & ĐẶC TẢ NGHIỆP VỤ (SRS FR-20, FR-09, FR-10)
* **Thuộc nhóm tính năng:** Pool D — Mobile App (React Native).
* **Mô tả nghiệp vụ:**
  1. **Luồng Thanh toán Mobile:** Người dùng đăng nhập, có sản phẩm trong giỏ hàng, nhập địa chỉ nhận hàng, chọn phương thức thanh toán (`COD`, `BANKING`, `MOMO`) và tiến hành thanh toán.
  2. **Áp dụng Mã giảm giá (FR-09 trên Mobile):**
     - Mã `SAVE10`: Giảm 10%, yêu cầu đơn hàng tối thiểu $\ge 300,000$ ₫, hạn dùng đến 2099, giới hạn 1 lần dùng / user.
     - Nếu tổng đơn $< 300,000$ ₫: Từ chối áp dụng mã, hiển thị Toast cảnh báo đỏ.
     - Nếu đã dùng 1 lần: Báo lỗi hết lượt dùng.
  3. **Máy trạng thái Hủy đơn hàng (Order State Machine - FR-10):**
     - Chuỗi trạng thái: `pending` $\rightarrow$ `confirmed` $\rightarrow$ `shipping` $\rightarrow$ `delivered` (hoặc `canceled`).
     - **Quy tắc ranh giới:** Trên Mobile App, người dùng **CHỈ ĐƯỢC PHÉP** bấm nút "Hủy đơn hàng" khi đơn ở trạng thái `pending` hoặc `confirmed`.
     - Khi đơn chuyển sang `shipping`, nút Hủy trên Mobile phải **BỊ ẨN** hoàn toàn; nếu can thiệp gọi API, hệ thống phải chặn với HTTP `400 Bad Request`.
     - Trạng thái `delivered` và `canceled` là trạng thái kết thúc (Final States), không thể hủy lại.

---

## 2. LIỆT KÊ BIẾN ĐẦU VÀO VÀ MIỀN GIÁ TRỊ

| Tên biến | Kiểu dữ liệu | Miền giá trị lý thuyết | Ý nghĩa trên Mobile |
| :--- | :---: | :--- | :--- |
| `order_total` ($T$) | Real | $[0, +\infty)$ | Tổng giá trị đơn hàng trước khi áp mã giảm giá (₫). |
| `coupon_code` ($CP$) | String | Chuỗi mã ký tự | Mã giảm giá nhập tại ô TextInput trên Mobile UI. |
| `coupon_usage` ($U$) | Integer | $\mathbb{Z}_{\ge 0}$ | Số lần tài khoản hiện tại đã áp dụng mã này thành công. |
| `shipping_address` ($L$) | String | Độ dài chuỗi $L \ge 0$ | Chuỗi địa chỉ nhập tại form Checkout Mobile. |
| `order_status` ($S$) | Enum | $\{ \text{'pending'}, \text{'confirmed'}, \text{'shipping'}, \text{'delivered'}, \text{'canceled'} \}$ | Trạng thái của đơn hàng khi người dùng thao tác Hủy đơn. |

---

## 3. PHÂN CHIA LỚP TƯƠNG ĐƯƠNG & GIẢI THÍCH LÝ DO

### 3.1. Biến `order_total` ($T$) với mã `SAVE10` (Ngưỡng $\ge 300,000$ ₫)
* **$T_1$ (Hợp lệ):** $T \ge 300,000$ ₫. Thỏa mãn điều kiện tối thiểu. Điểm On-point: $T = 300,000$ ₫; In-point: $T = 500,000$ ₫.
* **$T_2$ (Không hợp lệ):** $0 < T < 300,000$ ₫. Chưa đạt giá trị tối thiểu. Điểm Off-point: $T = 299,999$ ₫.

### 3.2. Biến `coupon_code` ($CP$)
* **$CP_1$ (Hợp lệ):** Mã đang hoạt động và còn hạn (`SAVE10`).
* **$CP_2$ (Không hợp lệ):** Mã không tồn tại (`INVALIDCODE`).
* **$CP_3$ (Không hợp lệ):** Mã đã hết hạn (`EXPIRED`).

### 3.3. Biến số lượt dùng mã `coupon_usage` ($U$)
* **$U_1$ (Hợp lệ):** $U = 0$ (Chưa từng dùng).
* **$U_2$ (Không hợp lệ):** $U \ge 1$ (Đã dùng hết 1 lượt cho phép).

### 3.4. Biến trạng thái đơn hàng khi thực hiện HỦY ĐƠN ($S$)
* **$S_{valid}$ (Hợp lệ):** $S \in \{ \text{'pending'}, \text{'confirmed'} \}$. Cho phép người dùng bấm hủy.
* **$S_{border}$ (Không hợp lệ - Biên chặn):** $S = \text{'shipping'}$. Hàng đang giao, cấm hủy. Nút Hủy bị ẩn trên Mobile UI; API chặn trả về 400.
* **$S_{final}$ (Không hợp lệ - Đóng vĩnh viễn):** $S \in \{ \text{'delivered'}, \text{'canceled'} \}$.

---

## 4. BẢNG TEST CASES DOMAIN TESTING CHO MOBILE (SINGLE FAULT ASSUMPTION)

| Test ID | Lớp tương đương | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected Result) | Kết quả thực tế trên SUT (Actual Result) | Trạng thái (Pass/Fail) |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC_MOB_01** | $T_1, CP_1, U_1, L_1$ | $T=500k$, mã `SAVE10`, $U=0$ | Giảm 50k (10%), còn 450k. Đặt hàng thành công. | Áp dụng mã thành công, tạo đơn 450k. | **PASS** |
| **TC_MOB_02** | **$T_2$ (Dưới ngưỡng)** | $T=299,999$ ₫, mã `SAVE10` | Báo Toast lỗi: "Đơn hàng tối thiểu phải từ 300,000 ₫". | Toast đỏ từ chối mã giảm giá thành công. | **PASS** |
| **TC_MOB_03** | $T_1$ (Đúng ngưỡng 300k) | $T=300,000$ ₫, mã `SAVE10` | Áp dụng thành công, giảm 30k, còn 270k. | Báo Toast lỗi: **"Không thể áp dụng mã" do code dùng `>` thay vì `>=`!** | **FAIL (BUG-FR20-02)** |
| **TC_MOB_04** | **$CP_2$ (Mã sai)** | Mã `INVALIDCODE` | Báo lỗi: "Mã giảm giá không tồn tại". | Toast báo mã không tồn tại. | **PASS** |
| **TC_MOB_05** | **$CP_3$ (Mã hết hạn)**| Mã `EXPIRED` | Báo lỗi: "Mã giảm giá đã hết hạn sử dụng". | Toast báo mã đã hết hạn. | **PASS** |
| **TC_MOB_06** | **$U_2$ (Hết lượt dùng)**| Mã `SAVE10`, $U=1$ | Báo lỗi: "Bạn đã sử dụng hết lượt cho phép". | Toast báo đã hết lượt áp dụng. | **PASS** |
| **TC_MOB_07** | **$L_2$ (Địa chỉ rỗng)**| `address: ""` | Cảnh báo đỏ; nút Đặt hàng bị disabled. | Cảnh báo đỏ dưới ô nhập, nút bị disabled. | **PASS** |
| **TC_MOB_08** | $S_{valid}$ (`pending`) | Đơn hàng `status: 'pending'` | Nút "Hủy đơn" hiển thị, hủy chuyển sang `canceled`. | Hủy đơn thành công trên Mobile. | **PASS** |
| **TC_MOB_09** | $S_{valid}$ (`confirmed`)| Đơn hàng `status: 'confirmed'` | Nút "Hủy đơn" hiển thị, hủy chuyển sang `canceled`. | Hủy đơn thành công trên Mobile. | **PASS** |
| **TC_MOB_10** | **$S_{border}$ (`shipping`)**| Đơn hàng `status: 'shipping'` | Nút Hủy bị ẩn; gửi API hủy bị chặn HTTP 400. | Nút bị ẩn trên UI, nhưng **gửi API trực tiếp thì Backend cho hủy thành công**! | **FAIL (BUG-FR20-01)** |
| **TC_MOB_11** | **$S_{final}$ (Đã kết thúc)**| Đơn hàng `delivered` | Nhãn tĩnh, không có nút Hủy. | Không có nút Hủy, hiển thị nhãn tĩnh. | **PASS** |
