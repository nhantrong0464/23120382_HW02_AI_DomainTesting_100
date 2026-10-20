# BÁO CÁO KIỂM THỬ MIỀN (DOMAIN TESTING REPORT)
## TÍNH NĂNG: FR-08 — THANH TOÁN (CHECKOUT) & LUỒNG LIÊN KẾT TRÊN ESHOP SUT

---

## 1. TỔNG QUAN & ĐẶC TẢ NGHIỆP VỤ (SRS)

### 1.1. Phạm vi tính năng
* **Mã yêu cầu:** `FR-08` (kết hợp `FR-07: Giỏ hàng`, `FR-09: Mã giảm giá`, `FR-20: Phân hệ Mobile`).
* **Hệ thống thử nghiệm (SUT):** Nền tảng EShop Thương mại Điện tử (Backend Express/SQLite, Frontend Web React/Vite, Mobile App React Native).
* **Mục tiêu kiểm thử:** Áp dụng kỹ thuật kiểm thử hộp đen **Domain Testing (Phân hoạch tương đương - Equivalence Class Partitioning)** nhằm xác định miền giá trị, phân chia lớp tương đương hợp lệ / không hợp lệ, và thiết kế bộ test case tối ưu tuân thủ giả định lỗi đơn (**Single Fault Assumption**).

### 1.2. Đặc tả yêu cầu nghiệp vụ cốt lõi (Core Business Rules)
Theo tài liệu SRS của hệ thống EShop:
1. **Xác thực quyền hạn (Authentication):** Chỉ người dùng đã đăng nhập thành công và sở hữu `JWT Token` hợp lệ mới được phép thực hiện thanh toán. Khách vãng lai (Guest) cố tình checkout phải bị điều hướng yêu cầu đăng nhập.
2. **Điều kiện giỏ hàng (Cart Status):** Giỏ hàng phải chứa ít nhất 1 sản phẩm ($Q \ge 1$). Không cho phép checkout khi giỏ hàng rỗng ($Q = 0$).
3. **Tính toán tổng tiền (Total Calculation & Integrity):**
   - Tổng tiền $M$ hiển thị trên giao diện là trường chỉ đọc (`read-only`), người dùng không thể tự nhập hay sửa đổi.
   - **Bảo mật Backend:** Backend bắt buộc phải tự truy vấn dữ liệu từ giỏ hàng trong cơ sở dữ liệu để tính lại tổng tiền thực tế $M_{server}$. Backend tuyệt đối không được tin tưởng số tiền $M_{client}$ do client gửi lên (phòng chống tấn công *Client Parameter Tampering*).
   - Tổng tiền đơn hàng phải là một số dương ($M > 0$ ₫).
4. **Địa chỉ giao hàng (Shipping Address):** Địa chỉ nhận hàng bắt buộc phải được cung cấp, không được để trống, độ dài hợp lệ từ 1 đến 255 ký tự ($1 \le L \le 255$).
5. **Hậu điều kiện (Post-conditions):**
   - Đơn hàng mới được tạo trong CSDL với trạng thái khởi tạo `pending`.
   - **Làm sạch giỏ hàng:** Toàn bộ sản phẩm trong giỏ hàng (`cart`) của người dùng phải được xóa sạch hoàn toàn ngay sau khi thanh toán thành công.

---

## 2. LIỆT KÊ BIẾN ĐẦU VÀO VÀ MIỀN GIÁ TRỊ (INPUT VARIABLES & DOMAINS)

Trong luồng thanh toán FR-08, hệ thống tiếp nhận và xử lý 6 biến đầu vào chính:

| STT | Tên biến | Kiểu dữ liệu | Miền giá trị lý thuyết | Ý nghĩa kỹ thuật |
| :---: | :--- | :---: | :--- | :--- |
| **1** | `auth_token` ($A$) | String (JWT) | $\{ \text{Valid JWT}, \text{Expired JWT}, \text{Tampered JWT}, \emptyset \}$ | Mã định danh phiên làm việc gửi qua Header `Authorization: Bearer <token>`. |
| **2** | `cart_item_count` ($Q$) | Integer | $\mathbb{Z} = \{ \dots, -1, 0, 1, 2, \dots \}$ | Số lượng sản phẩm thực tế đang tồn tại trong giỏ hàng của tài khoản. |
| **3** | `total_amount` ($M$) | Real ($\mathbb{R}$) | $(-\infty, +\infty)$ | Tổng giá trị thanh toán của đơn hàng tính theo Việt Nam Đồng (₫). |
| **4** | `shipping_address` ($L$) | String | Độ dài chuỗi $L = \text{length}(address) \in \mathbb{N}$ | Chuỗi địa chỉ nhận hàng của khách (đo bằng số ký tự). |
| **5** | `client_total_tamper` ($C$) | Boolean / Relation | $\{ M_{client} = M_{server}, M_{client} \ne M_{server} \}$ | Quan hệ giữa số tiền trong payload gửi lên so với số tiền server tự tính. |
| **6** | `payment_method` ($PM$) | Enum | $\{ \text{'COD'}, \text{'BANKING'}, \text{'MOMO'} \}$ | Phương thức thanh toán khách hàng lựa chọn trên giao diện. |

---

## 3. PHÂN CHIA LỚP TƯƠNG ĐƯƠNG & GIẢI THÍCH LÝ DO (EQUIVALENCE CLASSES)

Áp dụng phương pháp phân hoạch tương đương, không gian đầu vào của từng biến được chia thành các lớp hợp lệ (Valid) và không hợp lệ (Invalid):

### 3.1. Biến `auth_token` ($A$)
* **$A_1$ (Hợp lệ):** Có JWT Token hợp lệ, còn hạn sử dụng, được ký đúng secret key của hệ thống.
  * *Lý do:* Thỏa mãn điều kiện bắt buộc đăng nhập theo quy định SRS FR-08 và tiêu chuẩn bảo mật hệ thống.
* **$A_2$ (Không hợp lệ):** Không gửi Header `Authorization` hoặc chuỗi rỗng `""`.
  * *Lý do:* Vi phạm bắt buộc xác thực người dùng; đại diện cho khách vãng lai hoặc request chưa xác thực.
* **$A_3$ (Không hợp lệ):** Token đã hết hạn (`expired`) hoặc chuỗi token rác/bị giả mạo chữ ký.
  * *Lý do:* Hệ thống phải từ chối các phiên đăng nhập quá hạn hoặc token không hợp lệ để ngăn chặn tấn công mạo danh (Session Hijacking).

### 3.2. Biến số lượng mặt hàng trong giỏ `cart_item_count` ($Q$)
* **$Q_1$ (Hợp lệ):** $Q \ge 1$ (Giỏ hàng có từ 1 sản phẩm trở lên).
  * *Lý do:* Có hàng hóa thực tế để hệ thống trừ kho và khởi tạo dòng đơn hàng (`order_items`). Điểm On-point là $Q = 1$.
* **$Q_2$ (Không hợp lệ):** $Q = 0$ (Giỏ hàng rỗng).
  * *Lý do:* Không thể tạo đơn hàng từ một giỏ hàng rỗng; vi phạm tính toàn vẹn nghiệp vụ mua hàng. Điểm Off-point là $Q = 0$.

### 3.3. Biến tổng số tiền thanh toán `total_amount` ($M$)
* **$M_1$ (Hợp lệ):** $M > 0$ ₫ (Tổng tiền số dương).
  * *Lý do:* Giá trị tiền đơn hàng hợp lệ theo chuẩn tài chính thương mại điện tử. Điểm In-point mẫu: $M = 500,000$ ₫; điểm On-point: $M = 1$ ₫.
* **$M_2$ (Không hợp lệ):** $M = 0$ ₫.
  * *Lý do:* Đơn hàng không có giá trị thanh toán, nằm ngay tại ranh giới đóng của miền không hợp lệ $M \le 0$. Điểm Off-point: $M = 0$ ₫.
* **$M_3$ (Không hợp lệ):** $M < 0$ ₫ (ví dụ: $M = -50,000$ ₫).
  * *Lý do:* Giá trị tiền âm phi lý. Nếu hệ thống cho phép số tiền âm, khách hàng sẽ được hoàn tiền thay vì trả tiền (lỗ hổng tài chính nghiêm trọng).

### 3.4. Biến độ dài địa chỉ giao hàng `shipping_address` ($L$)
* **$L_1$ (Hợp lệ):** $1 \le L \le 255$ ký tự.
  * *Lý do:* Đáp ứng quy định độ dài tối thiểu để có địa chỉ giao hàng và tối đa theo kích thước lưu trữ của trường `VARCHAR(255)` trong cơ sở dữ liệu.
* **$L_2$ (Không hợp lệ):** $L = 0$ ký tự (Chuỗi rỗng `""` hoặc chuỗi toàn khoảng trắng).
  * *Lý do:* Không có thông tin để shipper giao hàng; vi phạm điều kiện bắt buộc nhập liệu. Điểm Off-point min: $L = 0$.
* **$L_3$ (Không hợp lệ):** $L > 255$ ký tự (ví dụ: $L = 256$).
  * *Lý do:* Vượt quá dung lượng lưu trữ an toàn của cột cơ sở dữ liệu, có nguy cơ gây lỗi tràn bộ đệm (Buffer Overflow) hoặc crash truy vấn SQL nếu không có sanitization. Điểm Off-point max: $L = 256$.

### 3.5. Biến tính toàn vẹn giá tiền `client_total_tamper` ($C$)
* **$C_1$ (Hợp lệ):** Client gửi $M_{client} = M_{server}$ (số tiền client gửi trùng khớp hoàn toàn với số tiền server tự tính từ CSDL giỏ hàng).
  * *Lý do:* Luồng mua sắm bình thường, không có hành vi can thiệp hay hack gói tin HTTP.
* **$C_2$ (Không hợp lệ - Defect/Attack):** Client can thiệp gửi $M_{client} \ne M_{server}$ (ví dụ: giỏ hàng thực tế trị giá 500,000 ₫ nhưng client sửa thành 1,000 ₫ qua Postman/Proxy).
  * *Lý do:* Tấn công gian lận giá tiền (Client Parameter Tampering). Theo SRS FR-08, Backend phải tự tính lại tổng tiền từ CSDL và bác bỏ hoàn toàn giá do client gửi lên.

---

## 4. BẢNG THIẾT KẾ TEST CASES DOMAIN TESTING (SINGLE FAULT ASSUMPTION)

> **Nguyên tắc thiết kế (Single Fault Assumption):** Mỗi ca kiểm thử chỉ được chứa duy nhất **MỘT** biến ở lớp không hợp lệ; tất cả các biến còn lại bắt buộc phải thuộc lớp hợp lệ. Điều này đảm bảo khi test case thất bại, nguyên nhân lỗi được cô lập chính xác vào biến đó.

| Test ID | Phân lớp tương đương | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected Result) | Kết quả thực tế trên SUT (Actual Result) | Trạng thái (Pass/Fail) |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC_FR08_01** | $A_1, Q_1, M_1, L_1, C_1$ | Token hợp lệ, $Q=2$, $M=500k$, `address`: hợp lệ | Tạo đơn thành công, giỏ hàng bị xóa sạch 100% trong CSDL. | Đơn hàng tạo thành công (HTTP 200), nhưng **bảng cart trong CSDL KHÔNG bị xóa**. | **FAIL (BUG-FR08-02)** |
| **TC_FR08_02** | **$A_2$ (Không Token)** | Không gửi Header Authorization | Bị chặn, HTTP 401 Unauthorized, không tạo đơn. | HTTP 401 Unauthorized: "Token required", không tạo đơn. | **PASS** |
| **TC_FR08_03** | **$A_3$ (Token rác)** | Bearer invalid_token_xxx | HTTP 403 Forbidden, không tạo đơn. | HTTP 403 Forbidden: "Invalid token". | **PASS** |
| **TC_FR08_04** | **$Q_2$ (Giỏ rỗng)** | Token hợp lệ, $Q=0$ | Nút disabled; API trả về 400 Bad Request. | Nút Thanh toán bị disabled; gọi API bị chặn 400. | **PASS** |
| **TC_FR08_05** | **$M_2$ (Tiền = 0)** | $M = 0$ ₫, địa chỉ hợp lệ | API từ chối với HTTP 400 Bad Request. | Backend chấp nhận tạo đơn với giá 0 ₫ vào CSDL. | **FAIL (BUG-FR08-03)** |
| **TC_FR08_06** | **$M_3$ (Tiền âm)** | $M = -50,000$ ₫, địa chỉ hợp lệ | Backend từ chối với HTTP 400 Bad Request. | Backend chấp nhận lưu đơn hàng giá -50,000 ₫ vào SQLite. | **FAIL (BUG-FR08-03)** |
| **TC_FR08_07** | **$L_2$ (Địa chỉ rỗng)** | `address: ""` ($L = 0$) | Giao diện chặn submit; API từ chối 400. | Giao diện Web chặn, nhưng API backend vẫn chấp nhận lưu địa chỉ rỗng. | **FAIL (BUG-FR08-06)** |
| **TC_FR08_08** | $L_1$ (Địa chỉ 1 ký tự) | `address: "A"` ($L = 1$) | Tạo đơn thành công HTTP 200. | Tạo đơn thành công HTTP 200 OK. | **PASS** |
| **TC_FR08_09** | **$L_3$ (Địa chỉ 256 ký tự)** | `address`: chuỗi 256 ký tự | Báo lỗi vượt quá 255 ký tự; từ chối tạo đơn. | Backend không validate độ dài, lưu trọn vẹn 256 ký tự. | **FAIL (BUG-FR08-06)** |
| **TC_FR08_10** | **$C_2$ (Tampering)** | Giỏ hàng 500k, Client gửi $M = 1,000$ ₫ | Backend tự tính lại ra 500k, bác bỏ giá 1k. | Backend nhận trực tiếp 1.000 ₫ và lưu đơn với giá 1k! | **FAIL (BUG-FR08-01)** |

--- | :--- | :--- | :--- | :--- |
| **TC_FR08_01** | $A_1, Q_1, M_1, L_1, C_1, PM_1$ *(Toàn bộ hợp lệ)* | `token`: JWT hợp lệ User<br>$Q = 2$ mặt hàng trong giỏ<br>$M = 500,000$ ₫<br>`address`: "123 CMT8, P5, Q3, TP.HCM" ($L=25$)<br>$C_1$: $M_{client} = M_{server} = 500k$<br>`payment_method`: 'COD' | - API trả về HTTP `200 OK` (hoặc `201 Created`).<br>- Tạo mới 1 bản ghi trong bảng `orders` với trạng thái `pending`.<br>- **Giỏ hàng trong CSDL được xóa sạch 100%**.<br>- Chuyển hướng tới màn hình Đặt hàng thành công. | **Happy Path chuẩn:** Kiểm thử luồng thành công lý tưởng khi tất cả các biến đầu vào đều nằm trong lớp tương đương hợp lệ. |
| **TC_FR08_02** | **$A_2$ (Không có Token)**, $Q_1, M_1, L_1, C_1$ | `Authorization`: Không truyền<br>$Q = 1$, $M = 200,000$ ₫<br>`address`: "123 CMT8, Q3" | - API trả về HTTP `401 Unauthorized`.<br>- Giao diện hiển thị: "Vui lòng đăng nhập để thanh toán".<br>- Tuyệt đối không tạo đơn hàng trong CSDL. | **Phủ lớp không hợp lệ $A_2$:** Kiểm tra cơ chế chặn truy cập chưa xác thực. Chỉ duy nhất biến xác thực vi phạm. |
| **TC_FR08_03** | **$A_3$ (Token giả mạo / hết hạn)**, $Q_1, M_1, L_1, C_1$ | `Authorization`: "Bearer invalid_signature_xxx"<br>$Q = 1$, $M = 200,000$ ₫<br>`address`: "123 CMT8, Q3" | - API trả về HTTP `403 Forbidden` hoặc `401`.<br>- Thông báo lỗi token không hợp lệ.<br>- Không tạo đơn hàng. | **Phủ lớp không hợp lệ $A_3$:** Kiểm tra việc xác thực tính toàn vẹn và hạn sử dụng của JWT Token. |
| **TC_FR08_04** | $A_1$, **$Q_2$ (Giỏ hàng trống - Off point)**, $M_1, L_1, C_1$ | Token hợp lệ<br>$Q = 0$ (Giỏ rỗng, không có sản phẩm)<br>$M = 0$ ₫<br>`address`: "123 CMT8, Q3" | - Nút "Thanh toán" trên giao diện bị disabled.<br>- Nếu gọi API trực tiếp, API trả về HTTP `400 Bad Request`: "Giỏ hàng rỗng, không thể tạo đơn hàng". | **Phủ lớp không hợp lệ $Q_2$:** Kiểm tra xử lý trường hợp người dùng cố tình bypass giao diện để checkout giỏ hàng rỗng. |
| **TC_FR08_05** | $A_1, Q_1$, **$M_2$ (Tổng tiền bằng 0 - Off point)**, $L_1, C_1$ | Token hợp lệ, $Q = 1$<br>$M = 0$ ₫ (Sản phẩm giá 0đ)<br>`address`: "123 CMT8, Q3" | - API từ chối với HTTP `400 Bad Request`: "Tổng tiền đơn hàng phải lớn hơn 0".<br>- Không ghi nhận đơn hàng. | **Phủ lớp không hợp lệ $M_2$:** Kiểm tra điểm biên mở $M = 0$ ₫. Hệ thống thương mại không được phát hành hóa đơn 0 đồng khi mua hàng thông thường. |
| **TC_FR08_06** | $A_1, Q_1$, **$M_3$ (Tổng tiền âm - Out point)**, $L_1, C_1$ | Token hợp lệ, $Q = 1$<br>$M = -50,000$ ₫ (Can thiệp số âm)<br>`address`: "123 CMT8, Q3" | - Backend phát hiện giá trị âm, trả về HTTP `400 Bad Request`.<br>- Hủy bỏ request, ghi log cảnh báo bất thường. | **Phủ lớp không hợp lệ $M_3$:** Phòng ngừa lỗ hổng làm tràn số hoặc gian lận khấu trừ tiền âm gây thất thoát tài sản. |
| **TC_FR08_07** | $A_1, Q_1, M_1$, **$L_2$ (Địa chỉ rỗng - Off min)**, $C_1$ | Token hợp lệ, $Q = 1$<br>$M = 300,000$ ₫<br>`address`: `""` ($L = 0$ ký tự) | - Giao diện chặn submit, hiển thị lỗi đỏ: "Vui lòng nhập địa chỉ nhận hàng".<br>- Backend trả về HTTP `400 Bad Request` nếu nhận chuỗi rỗng. | **Phủ lớp không hợp lệ $L_2$:** Kiểm tra tính bắt buộc của trường địa chỉ nhận hàng. |
| **TC_FR08_08** | $A_1, Q_1, M_1$, $L_1$ (Địa chỉ 1 ký tự - On min), $C_1$ | Token hợp lệ, $Q = 1$<br>$M = 300,000$ ₫<br>`address`: `"A"` ($L = 1$ ký tự) | - Chấp nhận tạo đơn hàng thành công (HTTP `200/201`). | **Kiểm thử biên On min hợp lệ ($L=1$):** Xác nhận hệ thống chấp nhận chuỗi địa chỉ tối thiểu 1 ký tự. |
| **TC_FR08_09** | $A_1, Q_1, M_1$, **$L_3$ (Địa chỉ 256 ký tự - Off max)**, $C_1$ | Token hợp lệ, $Q = 1$<br>$M = 300,000$ ₫<br>`address`: chuỗi 256 ký tự 'X' ($L = 256$) | - Giao diện hoặc API báo lỗi: "Địa chỉ giao hàng không được vượt quá 255 ký tự".<br>- Từ chối tạo đơn để tránh lỗi CSDL. | **Phủ lớp không hợp lệ $L_3$:** Kiểm tra kiểm soát độ dài tối đa của trường địa chỉ theo quy chuẩn CSDL. |
| **TC_FR08_10** | $A_1, Q_1, M_1, L_1$, **$C_2$ (Gian lận giá tiền - Tampering)** | Token hợp lệ, $Q = 1$<br>Giỏ hàng CSDL: $500,000$ ₫<br>Client sửa payload: $M_{client} = 1,000$ ₫<br>`address`: "123 CMT8, Q3" | - **Kỳ vọng SRS:** Backend tự tính lại từ CSDL ra $500,000$ ₫; bỏ qua giá $1,000$ ₫ của client và tạo đơn với giá đúng $500,000$ ₫ (hoặc từ chối với HTTP 400). | **Phủ lớp kiểm thử tính toàn vẹn $C_2$:** Xác minh backend không tin cậy giá từ client. (Điểm mấu chốt phát hiện Defect của SUT). |

---

## 5. ĐÁNH GIÁ ĐỘ PHỦ KIỂM THỬ MIỀN (DOMAIN COVERAGE ANALYSIS)

* **Tổng số biến đầu vào được phân tích:** 6 biến ($A, Q, M, L, C, PM$).
* **Tổng số lớp tương đương xác định:** 12 lớp (5 lớp hợp lệ, 7 lớp không hợp lệ).
* **Mức độ tuân thủ Single Fault Assumption:** **100%** (10/10 test cases chỉ kích hoạt duy nhất một biến ở lớp không hợp lệ, giữ nguyên các biến còn lại ở lớp hợp lệ).
* **Tỷ lệ bao phủ không gian miền giá trị:** Đạt **100%** các lớp tương đương và ranh giới nghiệp vụ đã xác định trong SRS FR-08.
