# NHẬT KÝ TƯƠNG TÁC VÀ THỰC THI AI (AI_LOG)
# PHẦN 2: THIẾT KẾ TEST CASE THEO KỸ THUẬT DOMAIN TESTING (PHÂN HOẠCH TƯƠNG ĐƯƠNG)
> **Dự án:** Kiểm thử chất lượng hệ thống thương mại điện tử EShop SUT  
> **Thành viên phụ trách:** Chuyên viên Kiểm thử Phần mềm (QA / Software Tester)  
> **Phương pháp kiểm định:** Domain Testing (Phân hoạch tương đương - Equivalence Partitioning & Phân tích giá trị biên - Boundary Value Analysis)  
> **Nguyên tắc thiết kế cốt lõi:** Giả định lỗi đơn (Single Fault Assumption) — Mỗi ca kiểm thử không hợp lệ chỉ phủ DUY NHẤT một lớp tương đương không hợp lệ, tất cả các biến còn lại đều nhận giá trị hợp lệ nhằm ngăn ngừa hiện tượng lỗi bị che khuất (Fault Masking).  
> **Phạm vi tính năng thực hiện:**
> 1. **FR-02:** Đăng nhập & Khóa tài khoản (Login and Account Lockout)
> 2. **FR-08:** Thanh toán & Toàn vẹn giá tiền (Checkout & Price Integrity)
> 3. **FR-16:** Import sản phẩm từ file CSV (Web Admin - Product Import)
> 4. **Mobile (FR-20 kết hợp FR-08, FR-09, FR-10):** Luồng Thanh toán Mobile, Áp dụng mã giảm giá & Máy trạng thái Hủy đơn hàng (Order State Machine)

---

## Prompt 8: Thiết kế Domain Testing cho FR-02 — Đăng nhập & Khóa tài khoản
* **Mục tiêu:** Áp dụng kỹ thuật phân hoạch tương đương và giá trị biên để kiểm thử toàn diện chức năng xác thực, kiểm soát số lần đăng nhập sai và cơ chế tự động khóa/mở tài khoản theo thời gian.
* **Đầu ra:** Bảng phân tích biến đầu vào, phân chia lớp tương đương và bộ Test Cases chuẩn 5 cột (ID, Lớp tương đương, Input, Expected result, Giải thích).

```text
VAI TRÒ: Kiểm thử viên Phần mềm (QA / Software Tester).
TÍNH NĂNG: FR-02: Đăng nhập & Khóa tài khoản (Login and Account Lockout).

MÔ TẢ NGHIỆP VỤ (SRS FR-02):
- Người dùng nhập Email và Mật khẩu. Email có định dạng type="email" (RFC 5322).
- Sau mỗi lần đăng nhập sai, hệ thống tăng bộ đếm failed_attempts lên đúng 1 đơn vị.
- Nếu đăng nhập sai liên tiếp từ 3 lần trở lên (failed_attempts >= 3), tài khoản bị tạm khóa 30 giây. Hệ thống trả về mã lỗi HTTP 403 Forbidden và thông báo phù hợp; không để lộ chi tiết nguyên nhân.
- Trong thời gian 30 giây bị khóa, dù người dùng nhập đúng mật khẩu thì hệ thống vẫn từ chối đăng nhập (HTTP 403 Forbidden).
- Sau 30 giây, nếu nhập đúng mật khẩu thì tài khoản được mở khóa tự động, đăng nhập thành công (HTTP 200 OK), cấp JWT Token và reset failed_attempts về 0.

HÃY ÁP DỤNG DOMAIN TESTING (PHÂN HOẠCH TƯƠNG ĐƯƠNG):
1. Liệt kê từng biến đầu vào và miền giá trị.
2. Chia lớp tương đương hợp lệ / không hợp lệ cho từng biến, giải thích lý do chia.
3. Thiết kế test case, mỗi test case chỉ phủ 1 lớp không hợp lệ (Single Fault Assumption).
4. Xuất bảng: ID | Lớp tương đương | Input | Expected result | Giải thích.
```

### Kết quả Thực thi & Phân tích Chi tiết từ AI:

#### 1. Liệt kê từng biến đầu vào và miền giá trị
* **Biến 1: `email` (String):** Địa chỉ email người dùng nhập. Miền giá trị: Tập hợp các chuỗi ký tự ASCII theo chuẩn định dạng RFC 5322 và trạng thái đăng ký trong hệ thống.
* **Biến 2: `password` (String):** Mật khẩu người dùng nhập. Miền giá trị: Chuỗi ký tự bất kỳ, độ dài $\ge 0$.
* **Biến 3: `failed_attempts` ($F \in \mathbb{N}$):** Bộ đếm số lần đăng nhập sai mật khẩu liên tiếp đã lưu trong CSDL của tài khoản. Miền giá trị: $\{0, 1, 2, 3, \dots\}$.
* **Biến 4: `lockout_time_elapsed` ($T \in \mathbb{R}_{\ge 0}$):** Thời gian trôi qua (tính bằng giây) kể từ thời điểm tài khoản bị khóa khi $F \ge 3$. Miền giá trị: $[0, +\infty)$.

#### 2. Phân chia lớp tương đương hợp lệ / không hợp lệ & Giải thích
* **Đối với biến `email`:**
  * **$E_1$ (Hợp lệ):** Email đúng cú pháp tiêu chuẩn và tài khoản đã đăng ký tồn tại trong CSDL (`test@eshop.com`). *Lý do:* Thỏa mãn nhận diện tài khoản người dùng hợp lệ.
  * **$E_2$ (Không hợp lệ):** Chuỗi rỗng `""`. *Lý do:* Bỏ trống trường bắt buộc, vi phạm validation giao diện và backend.
  * **$E_3$ (Không hợp lệ):** Định dạng email sai cú pháp (thiếu ký tự `@`, thiếu domain như `test_eshop.com`, `user@`). *Lý do:* Vi phạm ràng buộc định dạng `type="email"`.
  * **$E_4$ (Không hợp lệ):** Email đúng định dạng nhưng chưa từng đăng ký (`notfound@eshop.com`). *Lý do:* Không tìm thấy bản ghi người dùng trong CSDL.
* **Đối với biến `password`:**
  * **$P_1$ (Hợp lệ):** Mật khẩu khớp chính xác với mật khẩu mã hóa trong CSDL (`Test1234!`). *Lý do:* Xác thực thành công danh tính người dùng.
  * **$P_2$ (Không hợp lệ):** Chuỗi rỗng `""`. *Lý do:* Bỏ trống trường mật khẩu bắt buộc.
  * **$P_3$ (Không hợp lệ):** Mật khẩu không khớp với mật khẩu tài khoản (`WrongPass123!`). *Lý do:* Cung cấp thông tin xác thực sai.
* **Đối với biến `failed_attempts` ($F$):**
  * **$F_1$ (Hợp lệ - Chưa đạt ngưỡng khóa):** $F \in \{0, 1, 2\}$. *Lý do:* Tài khoản hoạt động bình thường, số lần sai chưa chạm ngưỡng khóa (Biên đóng: $F \le 2$).
  * **$F_2$ (Không hợp lệ - Ngưỡng kích hoạt khóa):** $F \ge 3$ (khi thực hiện sai lần thứ 3). *Lý do:* Vi phạm quy tắc bảo mật SRS, kích hoạt cơ chế tạm khóa chống tấn công Brute-force.
  * **$F_3$ (Không hợp lệ):** $F < 0$ (ví dụ: $F = -1$). *Lý do:* Dữ liệu số âm bất hợp lệ (phòng chống can thiệp biến trạng thái).
* **Đối với biến `lockout_time_elapsed` ($T$) khi tài khoản đang ở trạng thái khóa:**
  * **$T_1$ (Không hợp lệ - Trong thời gian khóa):** $0 \le T \le 30$ giây. *Lý do:* Chưa mãn hạn phạt 30 giây theo SRS.
  * **$T_2$ (Hợp lệ - Đã hết thời gian khóa):** $T > 30$ giây (ví dụ $T = 31$s). *Lý do:* Đã mãn hạn khóa 30 giây, cho phép mở lại quyền truy cập nếu mật khẩu đúng.

#### 3. Bảng Test Case chuẩn Domain Testing FR-02 (Single Fault Assumption)

| ID | Lớp tương đương | Input | Expected result | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| **TC_FR02_EP01** | $E_1, P_1, F_1$ (Tất cả hợp lệ) | `email: "test@eshop.com"`<br>`password: "Test1234!"`<br>$F = 0$ | Đăng nhập thành công (HTTP 200 OK). Trả về JWT Token. Reset $F = 0$. Chuyển hướng về trang chủ. | Kiểm thử trường hợp chuẩn mực (Happy Path). Tất cả biến đều thuộc lớp tương đương hợp lệ. |
| **TC_FR02_EP02** | **$E_2$ (Email rỗng)**, $P_1, F_1$ | `email: ""` (Rỗng)<br>`password: "Test1234!"`<br>$F = 0$ | Giao diện hiển thị cảnh báo: "Vui lòng nhập Email". Chặn không gửi request API. | Phủ lớp không hợp lệ $E_2$. Các biến còn lại ($P_1, F_1$) hợp lệ để xác định lỗi do trường Email rỗng gây ra. |
| **TC_FR02_EP03** | **$E_3$ (Email sai format)**, $P_1, F_1$ | `email: "invalid_email_format"`<br>`password: "Test1234!"`<br>$F = 0$ | Form hiển thị lỗi định dạng email không hợp lệ (HTML5 email validation). Chặn submit. | Phủ lớp không hợp lệ $E_3$. Chỉ duy nhất biến Email bị sai cú pháp để kiểm tra tính năng validate cú pháp. |
| **TC_FR02_EP04** | **$E_4$ (Email chưa đăng ký)**, $P_1, F_1$ | `email: "unregistered@eshop.com"`<br>`password: "Test1234!"`<br>$F = 0$ | Trả về HTTP 401 Unauthorized (hoặc 404). Hiển thị: "Email hoặc mật khẩu không chính xác". Không cấp token. | Phủ lớp không hợp lệ $E_4$. Kiểm tra an toàn bảo mật, thông báo lỗi chung không làm lộ sự tồn tại của người dùng (User Enumeration). |
| **TC_FR02_EP05** | $E_1$, **$P_2$ (Mật khẩu rỗng)**, $F_1$ | `email: "test@eshop.com"`<br>`password: ""` (Rỗng)<br>$F = 0$ | Giao diện hiển thị cảnh báo: "Vui lòng nhập Mật khẩu". Chặn không gửi request. | Phủ lớp không hợp lệ $P_2$. Chỉ duy nhất mật khẩu để trống nhằm kiểm tra validation trường mật khẩu bắt buộc. |
| **TC_FR02_EP06** | $E_1$, **$P_3$ (Sai mật khẩu lần 1)**, $F_1$ | `email: "test@eshop.com"`<br>`password: "WrongPass1!"`<br>$F = 0$ | Trả về HTTP 401 Unauthorized. Thông báo: "Email hoặc mật khẩu không chính xác". Bộ đếm tăng lên $F = 1$. Tài khoản CHƯA bị khóa. | Phủ lớp không hợp lệ $P_3$ tại biên dưới $F_1$. Kiểm tra bộ đếm `failed_attempts` tăng đúng 1 đơn vị theo yêu cầu SRS. |
| **TC_FR02_EP07** | $E_1$, **$P_3$ (Sai mật khẩu lần 2 - Biên Off)**, $F_1$ | `email: "test@eshop.com"`<br>`password: "WrongPass2!"`<br>$F = 1$ | Trả về HTTP 401 Unauthorized. Bộ đếm tăng lên $F = 2$. Tài khoản VẪN CHƯA bị khóa. | Kiểm tra điểm cận biên Off ($F = 2$). Xác nhận tài khoản vẫn còn 1 lần thử cuối cùng trước khi bị khóa. |
| **TC_FR02_EP08** | $E_1$, **$P_3$**, **$F_2$ (Kích hoạt khóa - Biên On)** | `email: "test@eshop.com"`<br>`password: "WrongPass3!"`<br>$F = 2$ | Trả về HTTP 403 Forbidden. Thông báo: "Tài khoản bị tạm khóa 30 giây do nhập sai nhiều lần". Bắt đầu tính giờ $T$. | Phủ lớp không hợp lệ $F_2$ tại điểm biên đóng $F = 3$. Hệ thống lập tức kích hoạt trạng thái tạm khóa tài khoản. |
| **TC_FR02_EP09** | $E_1, P_1$, $F_2$, **$T_1$ (Đúng pass khi đang khóa)** | `email: "test@eshop.com"`<br>`password: "Test1234!"` (Đúng)<br>$F = 3$, $T = 15$ giây | Trả về HTTP 403 Forbidden. Vẫn hiển thị thông báo tài khoản đang bị khóa và thời gian còn lại. Không cho đăng nhập. | Phủ lớp không hợp lệ $T_1$ (thời gian chưa hết hạn 30s). Chứng minh cơ chế bảo mật khóa hoàn toàn, ngăn chặn brute-force ngay cả khi đoán đúng pass trong lúc khóa. |
| **TC_FR02_EP10** | $E_1, P_1$, $F_2$, **$T_2$ (Đúng pass sau khi hết 30s)** | `email: "test@eshop.com"`<br>`password: "Test1234!"` (Đúng)<br>$F = 3$, $T = 31$ giây | Đăng nhập THÀNH CÔNG (HTTP 200 OK). Cấp JWT Token. Reset bộ đếm $F = 0$. Mở khóa hoàn toàn. | Phủ lớp hợp lệ $T_2$ tại biên kế tiếp ($T = 31$s). Kiểm tra cơ chế tự phục hồi (Auto-recovery) sau thời gian 30 giây khóa tạm thời. |
| **TC_FR02_EP11** | $E_1, P_1$, **$F_3$ (Số lần sai là số âm)** | `email: "test@eshop.com"`<br>`password: "Test1234!"`<br>$F = -1$ | Hệ thống tự chuẩn hóa $F = 0$ hoặc trả về HTTP 400 Bad Request. Không bị crash cơ sở dữ liệu. | Phủ lớp không hợp lệ $F_3$ (dữ liệu số âm ngoại lai). Kiểm tra khả năng xử lý an toàn dữ liệu biên ngoài (Out-point). |

---

## Prompt 9: Thiết kế Domain Testing cho FR-08 — Thanh toán (Checkout)
* **Mục tiêu:** Kiểm thử quy trình tạo đơn hàng, xác thực tính toàn vẹn của tổng tiền thanh toán và kiểm soát dữ liệu địa chỉ nhận hàng.
* **Đầu ra:** Bảng phân tích biến đầu vào, phân chia lớp tương đương và bộ Test Cases chuẩn 5 cột (ID, Lớp tương đương, Input, Expected result, Giải thích).

```text
VAI TRÒ: Kiểm thử viên Phần mềm (QA / Software Tester).
TÍNH NĂNG: FR-08: Thanh toán (Checkout).

MÔ TẢ NGHIỆP VỤ (SRS FR-08):
- Chỉ người dùng đã đăng nhập (JWT Token hợp lệ) mới tiến hành thanh toán được.
- Đơn hàng phải có ít nhất 1 sản phẩm trong giỏ hàng (số lượng Q >= 1, tổng tiền M > 0).
- Tổng tiền thanh toán (total_amount) được tính tự động từ giỏ hàng trong CSDL; hệ thống không cho phép người dùng chỉnh sửa trực tiếp.
- Backend phải tự tính lại tổng tiền từ CSDL giỏ hàng; từ chối hoặc không chấp nhận giá trị total_amount do client gửi lên (phòng chống Parameter Tampering).
- Người dùng phải nhập địa chỉ giao hàng hợp lệ (shipping_address độ dài từ 1 đến 255 ký tự).
- Sau khi thanh toán thành công, giỏ hàng trong CSDL được xóa sạch 100%, đơn hàng chuyển trạng thái "pending".

HÃY ÁP DỤNG DOMAIN TESTING (PHÂN HOẠCH TƯƠNG ĐƯƠNG):
1. Liệt kê từng biến đầu vào và miền giá trị.
2. Chia lớp tương đương hợp lệ / không hợp lệ cho từng biến, giải thích lý do chia.
3. Thiết kế test case, mỗi test case chỉ phủ 1 lớp không hợp lệ (Single Fault Assumption).
4. Xuất bảng: ID | Lớp tương đương | Input | Expected result | Giải thích.
```

### Kết quả Thực thi & Phân tích Chi tiết từ AI:

#### 1. Liệt kê từng biến đầu vào và miền giá trị
* **Biến 1: `auth_token` (String/JWT):** Token xác thực người dùng trong Header `Authorization`. Miền giá trị: Chuỗi JWT hợp lệ, chuỗi token giả mạo, hoặc không có header.
* **Biến 2: `cart_item_count` ($Q \in \mathbb{N}$):** Số lượng mặt hàng có trong giỏ hàng của người dùng. Miền giá trị: $\{0, 1, 2, 3, \dots\}$.
* **Biến 3: `total_amount` ($M \in \mathbb{R}$):** Tổng số tiền thanh toán của đơn hàng (₫). Miền giá trị: $(-\infty, +\infty)$.
* **Biến 4: `shipping_address` (String):** Chuỗi địa chỉ nhận hàng, xét theo độ dài ký tự $L = \text{length}(address) \in \mathbb{N}$. Miền giá trị: $[0, +\infty)$.
* **Biến 5: `client_total_tamper` ($M_{client}$ so với $M_{server}$):** Mối quan hệ giữa giá tiền client gửi lên so với giá tiền backend tự tính từ CSDL giỏ hàng. Miền giá trị: $\{ M_{client} = M_{server}, M_{client} \ne M_{server} \}$.

#### 2. Phân chia lớp tương đương hợp lệ / không hợp lệ & Giải thích
* **Đối với biến `auth_token`:**
  * **$A_1$ (Hợp lệ):** Có JWT Token hợp lệ của người dùng đang đăng nhập. *Lý do:* Thỏa mãn điều kiện bắt buộc đăng nhập theo SRS FR-08.
  * **$A_2$ (Không hợp lệ):** Không gửi Token hoặc Token không hợp lệ / hết hạn. *Lý do:* Vi phạm kiểm soát xác thực người dùng.
* **Đối với biến `cart_item_count` ($Q$):**
  * **$Q_1$ (Hợp lệ):** $Q \ge 1$ (Giỏ hàng có từ 1 sản phẩm trở lên). *Lý do:* Có hàng hóa thực tế để thanh toán tạo đơn.
  * **$Q_2$ (Không hợp lệ):** $Q = 0$ (Giỏ hàng trống). *Lý do:* Không thể tạo đơn hàng từ một giỏ hàng rỗng.
* **Đối với biến `total_amount` ($M$):**
  * **$M_1$ (Hợp lệ):** $M > 0$ ₫ (Tổng tiền dương). *Lý do:* Giá trị tiền đơn hàng hợp lệ. Điểm biên Off là $M = 1$ ₫, In là $M = 500,000$ ₫.
  * **$M_2$ (Không hợp lệ):** $M = 0$ ₫. *Lý do:* Đơn hàng không có giá trị thanh toán, nằm ngay trên biên mở $M = 0$.
  * **$M_3$ (Không hợp lệ):** $M < 0$ ₫ (ví dụ: $M = -50,000$ ₫). *Lý do:* Giá trị tiền âm phi lý, có thể gây tổn thất tài chính nếu backend không chặn.
* **Đối với độ dài địa chỉ `shipping_address` ($L$):**
  * **$L_1$ (Hợp lệ):** $1 \le L \le 255$ ký tự. *Lý do:* Đáp ứng quy định độ dài tối thiểu và tối đa của trường địa chỉ trong CSDL. Biên On dưới: $L = 1$, Biên On trên: $L = 255$.
  * **$L_2$ (Không hợp lệ):** $L = 0$ ký tự (Chuỗi rỗng `""` hoặc chỉ có khoảng trắng). *Lý do:* Thiếu địa chỉ nhận hàng bắt buộc. Điểm cận biên Off dưới: $L = 0$.
  * **$L_3$ (Không hợp lệ):** $L > 255$ ký tự (ví dụ: $L = 256$). *Lý do:* Vượt quá giới hạn độ dài trường địa chỉ, có thể gây lỗi tràn cột CSDL. Điểm cận biên Off trên: $L = 256$.
* **Đối với tính toàn vẹn giá tiền (`client_total_tamper`):**
  * **$C_1$ (Hợp lệ):** Client gửi đúng tổng tiền trùng khớp với CSDL ($M_{client} = M_{server}$). *Lý do:* Không có hành vi can thiệp gian lận giá.
  * **$C_2$ (Không hợp lệ - Defect/Attack):** Client cố tình can thiệp gửi $M_{client} \ne M_{server}$ (ví dụ giỏ hàng 500,000 ₫ nhưng client gửi $M = 1,000$ ₫). *Lý do:* Vi phạm nghiêm ngặt yêu cầu "Backend tự tính lại tổng tiền, từ chối số tiền client gửi lên".

#### 3. Bảng Test Case chuẩn Domain Testing FR-08 (Single Fault Assumption)

| ID | Lớp tương đương | Input | Expected result | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| **TC_FR08_EP01** | $A_1, Q_1, M_1, L_1, C_1$ (Tất cả hợp lệ) | Token hợp lệ<br>$Q = 2$ sản phẩm trong giỏ<br>$M = 500,000$ ₫<br>`address: "123 Le Loi, Q1, TP.HCM"` ($L = 23$) | Tạo đơn hàng THÀNH CÔNG (HTTP 201 Created). Mã đơn sinh tự động, trạng thái `pending`. **Giỏ hàng trong CSDL bị xóa sạch 100%**. | Kiểm thử Happy Path chuẩn. Tất cả các biến đều thuộc lớp tương đương hợp lệ. |
| **TC_FR08_EP02** | **$A_2$ (Chưa đăng nhập)**, $Q_1, M_1, L_1$ | `Authorization: ""` (Không truyền token)<br>$Q = 1$, $M = 200,000$ ₫<br>`address: "123 Le Loi"` | API trả về HTTP 401 Unauthorized. Giao diện yêu cầu đăng nhập trước khi thanh toán. Không tạo đơn hàng. | Phủ lớp không hợp lệ $A_2$. Chỉ duy nhất trạng thái xác thực bị thiếu; các thông tin giỏ hàng và địa chỉ đều hợp lệ. |
| **TC_FR08_EP03** | $A_1$, **$Q_2$ (Giỏ hàng trống)**, $M_1, L_1$ | Token hợp lệ<br>$Q = 0$ (Giỏ rỗng)<br>$M = 0$ ₫<br>`address: "123 Le Loi, Q1"` | Nút thanh toán bị vô hiệu hóa (disabled). Nếu gửi API trả về HTTP 400 Bad Request: "Giỏ hàng rỗng, không thể tạo đơn". | Phủ lớp không hợp lệ $Q_2$. Kiểm tra xử lý trường hợp người dùng cố tình checkout khi giỏ hàng chưa có sản phẩm. |
| **TC_FR08_EP04** | $A_1, Q_1$, **$M_2$ (Tổng tiền = 0)**, $L_1$ | Token hợp lệ, $Q = 1$<br>$M = 0$ ₫ (Sản phẩm có giá 0đ)<br>`address: "123 Le Loi, Q1"` | API từ chối với HTTP 400 Bad Request: "Tổng tiền đơn hàng phải lớn hơn 0". Không tạo đơn. | Phủ lớp không hợp lệ $M_2$ tại điểm biên mở $M = 0$ ₫. Đảm bảo hệ thống không tạo hóa đơn với giá trị bằng 0. |
| **TC_FR08_EP05** | $A_1, Q_1$, **$M_3$ (Tổng tiền số âm)**, $L_1$ | Token hợp lệ, $Q = 1$<br>$M = -100,000$ ₫ (Can thiệp số âm)<br>`address: "123 Le Loi, Q1"` | Backend phát hiện giá trị âm, trả về HTTP 400 Bad Request. Đơn hàng bị hủy bỏ, ghi nhận cảnh báo bảo mật. | Phủ lớp không hợp lệ $M_3$ (Out-point). Ngăn chặn lỗi bảo mật kinh điển dẫn đến việc người dùng nhận tiền thay vì trả tiền. |
| **TC_FR08_EP06** | $A_1, Q_1, M_1$, **$L_2$ (Địa chỉ rỗng - Off min)** | Token hợp lệ, $Q = 1$<br>$M = 250,000$ ₫<br>`address: ""` ($L = 0$ ký tự) | Form hiển thị lỗi: "Vui lòng nhập địa chỉ nhận hàng". Chặn submit request tạo đơn. | Phủ lớp không hợp lệ $L_2$ (cận biên Off dưới $L = 0$). Kiểm tra kiểm soát bắt buộc trường địa chỉ. |
| **TC_FR08_EP07** | $A_1, Q_1, M_1$, $L_1$ (Địa chỉ 1 ký tự - On min) | Token hợp lệ, $Q = 1$<br>$M = 250,000$ ₫<br>`address: "H"` ($L = 1$ ký tự) | Chấp nhận tạo đơn thành công (HTTP 201 Created). | Kiểm thử giá trị biên On dưới hợp lệ ($L = 1$). Xác nhận hệ thống chấp nhận chuỗi địa chỉ tối thiểu 1 ký tự. |
| **TC_FR08_EP08** | $A_1, Q_1, M_1$, **$L_3$ (Địa chỉ 256 ký tự - Off max)** | Token hợp lệ, $Q = 1$<br>$M = 250,000$ ₫<br>`address:` chuỗi 256 ký tự 'A' ($L = 256$) | Báo lỗi: "Địa chỉ giao hàng không được vượt quá 255 ký tự" hoặc tự động cắt ngắn an toàn. | Phủ lớp không hợp lệ $L_3$ (cận biên Off trên $L = 256$). Kiểm tra phòng chống lỗi tràn bộ đệm hoặc lỗi CSDL. |
| **TC_FR08_EP09** | $A_1, Q_1, M_1, L_1$, **$C_2$ (Gian lận giá tiền)** | Token hợp lệ, $Q = 1$<br>Giỏ hàng thực tế = $500,000$ ₫<br>Client can thiệp gửi $M_{client} = 1,000$ ₫<br>`address: "123 Le Loi"` | Backend tự động tính toán lại từ DB ra $500,000$ ₫; bỏ qua giá client gửi và tạo đơn với giá đúng 500,000 ₫ (hoặc từ chối với 400). | Phủ lớp kiểm thử tính toàn vẹn $C_2$. Xác nhận backend không tin tưởng dữ liệu giá tiền từ phía client theo đúng SRS FR-08. |

---

## Prompt 10: Thiết kế Domain Testing cho FR-16 — Import Sản phẩm từ CSV (Web Admin)
* **Mục tiêu:** Kiểm thử chức năng đọc và nạp sản phẩm hàng loạt từ file CSV, kiểm tra ràng buộc kiểu dữ liệu, định dạng RFC 4180 và cơ chế giao dịch nguyên tử (Atomic Rollback: all-or-nothing).
* **Đầu ra:** Bảng phân tích biến đầu vào, phân chia lớp tương đương và bộ Test Cases chuẩn 5 cột (ID, Lớp tương đương, Input, Expected result, Giải thích).

```text
VAI TRÒ: Kiểm thử viên Phần mềm (QA / Software Tester).
TÍNH NĂNG: FR-16: Import Sản phẩm từ CSV (Web Admin).

MÔ TẢ NGHIỆP VỤ (SRS FR-16):
- Chỉ tài khoản Admin (role = 'admin') mới có quyền thực hiện import sản phẩm.
- Đuôi tệp bắt buộc là .csv. Dòng đầu tiên là header: name,price,description,imageUrl,category_id.
- Hỗ trợ chuẩn RFC 4180 (cho phép trường description chứa dấu phẩy nếu bọc trong ngoặc kép "").
- Ràng buộc dữ liệu từng dòng:
  + name: bắt buộc, không được để trống, tối đa 255 ký tự.
  + price: bắt buộc, phải là số dương (> 0).
  + category_id: bắt buộc, phải là mã danh mục đã tồn tại trong CSDL.
- Tính nguyên tử (Atomic Rollback - All-or-nothing): Nếu phát hiện lỗi ở bất kỳ dòng nào trong tệp CSV, toàn bộ quá trình import phải bị HỦY BỎ (Rollback), không có bất kỳ sản phẩm nào được lưu vào CSDL.
- Hệ thống trả về báo cáo chi tiết: số dòng thành công, số dòng lỗi và nguyên nhân cụ thể.

HÃY ÁP DỤNG DOMAIN TESTING (PHÂN HOẠCH TƯƠNG ĐƯƠNG):
1. Liệt kê từng biến đầu vào và miền giá trị.
2. Chia lớp tương đương hợp lệ / không hợp lệ cho từng biến, giải thích lý do chia.
3. Thiết kế test case, mỗi test case chỉ phủ 1 lớp không hợp lệ (Single Fault Assumption).
4. Xuất bảng: ID | Lớp tương đương | Input | Expected result | Giải thích.
```

### Kết quả Thực thi & Phân tích Chi tiết từ AI:

#### 1. Liệt kê từng biến đầu vào và miền giá trị
* **Biến 1: `user_role` (String/JWT Role):** Quyền của người thực hiện thao tác import. Miền giá trị: `admin`, `user`, `guest`.
* **Biến 2: `file_extension` (String):** Định dạng đuôi mở rộng của tệp tải lên. Miền giá trị: Chuỗi phần mở rộng tệp (`.csv`, `.xlsx`, `.txt`, `.png`,...).
* **Biến 3: `product_name` (String):** Tên sản phẩm trong dòng CSV, xét theo độ dài $N = \text{length}(name) \in \mathbb{N}$. Miền giá trị: $[0, +\infty)$.
* **Biến 4: `product_price` ($P \in \mathbb{R}$):** Giá bán sản phẩm trong dòng CSV. Miền giá trị: $(-\infty, +\infty)$.
* **Biến 5: `category_id` ($C \in \mathbb{Z}$):** Khóa ngoại danh mục sản phẩm. Miền giá trị: Tập hợp ID trong bảng `categories` và các số nguyên khác.
* **Biến 6: `rfc_4180_format` (Boolean):** Khả năng bọc trường có chứa dấu phẩy bằng dấu nháy kép `""`. Miền giá trị: `{ Đúng chuẩn RFC 4180, Sai cú pháp RFC }`.
* **Biến 7: `batch_consistency` (Enum):** Trạng thái hợp lệ của tập các dòng trong tệp CSV. Miền giá trị: `{ 100% dòng hợp lệ, Có ít nhất 1 dòng sai }`.

#### 2. Phân chia lớp tương đương hợp lệ / không hợp lệ & Giải thích
* **Đối với biến `user_role`:**
  * **$R_1$ (Hợp lệ):** Người dùng có `role = 'admin'`. *Lý do:* Quyền quản trị viên hợp lệ theo SRS FR-12 và FR-16.
  * **$R_2$ (Không hợp lệ):** Người dùng là `role = 'user'` hoặc chưa đăng nhập. *Lý do:* Không đủ thẩm quyền quản trị.
* **Đối với biến `file_extension`:**
  * **$EXT_1$ (Hợp lệ):** Tệp có đuôi `.csv` và mime type chuẩn `text/csv`. *Lý do:* Đúng định dạng quy định.
  * **$EXT_2$ (Không hợp lệ):** Tệp có đuôi khác `.csv` (ví dụ: `.xlsx`, `.txt`, `.json`). *Lý do:* Sai loại tệp cho phép.
* **Đối với biến `product_name` ($N$):**
  * **$N_1$ (Hợp lệ):** $1 \le N \le 255$ ký tự. *Lý do:* Đáp ứng quy định độ dài tên sản phẩm.
  * **$N_2$ (Không hợp lệ):** $N = 0$ ký tự (Cột `name` để trống). *Lý do:* Vi phạm quy tắc tên sản phẩm bắt buộc.
  * **$N_3$ (Không hợp lệ):** $N > 255$ ký tự (ví dụ: $N = 256$). *Lý do:* Vượt quá độ dài trường CSDL.
* **Đối với biến `product_price` ($P$):**
  * **$P_1$ (Hợp lệ):** $P > 0$ ₫ (Số dương thực sự). *Lý do:* Giá bán sản phẩm hợp lệ. Điểm biên On dưới: $P = 1$ ₫; In: $P = 150,000$ ₫.
  * **$P_2$ (Không hợp lệ):** $P = 0$ ₫. *Lý do:* Nằm trên biên mở của $P > 0$, giá bằng 0 không hợp lệ khi kinh doanh thương mại.
  * **$P_3$ (Không hợp lệ):** $P < 0$ ₫ (ví dụ: $P = -20,000$ ₫). *Lý do:* Giá âm phi lý về kinh tế.
  * **$P_4$ (Không hợp lệ):** $P$ không phải số (ví dụ: chuỗi `"Free"`, `"abc"`). *Lý do:* Sai kiểu dữ liệu số thực.
* **Đối với biến `category_id` ($C$):**
  * **$C_1$ (Hợp lệ):** $C$ tồn tại trong bảng `categories` (ví dụ: $C \in \{1, 2, 3\}$). *Lý do:* Khóa ngoại tham chiếu hợp lệ.
  * **$C_2$ (Không hợp lệ):** $C$ không tồn tại trong CSDL (ví dụ: $C = 9999$). *Lý do:* Vi phạm toàn vẹn khóa ngoại (Foreign Key Integrity).
* **Đối với quy tắc Atomic Rollback (`batch_consistency`):**
  * **$B_1$ (Hợp lệ):** $100\%$ các dòng trong file đều hợp lệ. *Lý do:* Đạt điều kiện commit giao dịch.
  * **$B_2$ (Kích hoạt Rollback):** Tệp có ít nhất một dòng vi phạm (ví dụ: 4 dòng đúng, 1 dòng sai giá). *Lý do:* Kích hoạt cơ chế Rollback toàn bộ để tránh dữ liệu dở dang (Inconsistent State).

#### 3. Bảng Test Case chuẩn Domain Testing FR-16 (Single Fault Assumption)

| ID | Lớp tương đương | Input | Expected result | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| **TC_FR16_EP01** | $R_1, EXT_1, N_1, P_1, C_1, B_1$ (Tất cả hợp lệ) | Role Admin<br>File `valid_products.csv`<br>Dòng: `"Bàn phím cơ",1500000,"Mô tả",http://...,1` | Import thành công 100% (HTTP 200 OK). Tất cả sản phẩm được thêm vào CSDL. Hiển thị báo cáo: "Thành công: X dòng, Lỗi: 0 dòng". | Kiểm thử trường hợp chuẩn mực (Happy Path). Mọi trường và dòng dữ liệu đều hợp lệ. |
| **TC_FR16_EP02** | **$R_2$ (Không có quyền Admin)**, $EXT_1, N_1, P_1$ | Role User thường (`role = 'user'`)<br>Tải lên file `valid_products.csv` | Trả về HTTP 403 Forbidden. Hiển thị thông báo: "Bạn không có quyền thực hiện thao tác này". Không lưu dữ liệu. | Phủ lớp không hợp lệ $R_2$. Chỉ duy nhất quyền hạn người dùng bị sai nhằm kiểm tra cơ chế phân quyền (Access Control). |
| **TC_FR16_EP03** | $R_1$, **$EXT_2$ (Sai định dạng file)**, $N_1, P_1$ | Role Admin<br>Tải lên file `products.xlsx` (Excel) hoặc `products.txt` | Giao diện và API chặn tệp, trả về HTTP 400 Bad Request: "Chỉ chấp nhận định dạng tệp .csv". Không phân tích file. | Phủ lớp không hợp lệ $EXT_2$. Kiểm tra kiểm soát loại định dạng tệp tải lên theo SRS FR-16. |
| **TC_FR16_EP04** | $R_1, EXT_1$, **$N_2$ (Tên sản phẩm rỗng)**, $P_1, C_1$ | Role Admin, file CSV chứa dòng:<br>`"",200000,"Mô tả",,1` ($N = 0$) | Báo lỗi tại dòng tương ứng: "Tên sản phẩm không được để trống". **Kích hoạt Atomic Rollback**: 0 sản phẩm nào được lưu vào DB. | Phủ lớp không hợp lệ $N_2$. Chỉ duy nhất cột tên sản phẩm bị để trống để kiểm tra validation tên bắt buộc. |
| **TC_FR16_EP05** | $R_1, EXT_1$, **$P_2$ (Giá = 0 ₫ - Biên On)**, $N_1, C_1$ | Role Admin, file CSV chứa dòng:<br>`"Sản phẩm 0Đ",0,"Mô tả",,1` ($P = 0$) | Báo lỗi: "Giá sản phẩm phải lớn hơn 0". **Kích hoạt Atomic Rollback toàn bộ file**. Không có sản phẩm nào được tạo. | Phủ lớp không hợp lệ $P_2$ tại biên mở $P = 0$. Đảm bảo không cho phép nhập sản phẩm giá 0 đồng vào kho hàng. |
| **TC_FR16_EP06** | $R_1, EXT_1$, $P_1$ (Giá = 1 ₫ - Biên Off hợp lệ), $N_1$ | Role Admin, file CSV chứa dòng:<br>`"Kẹo ngậm",1,"Mô tả",,1` ($P = 1$ ₫) | Import THÀNH CÔNG (HTTP 200 OK). Sản phẩm được thêm với giá đúng 1 ₫. | Kiểm thử giá trị cận biên Off hợp lệ ($P = 1$ ₫). Đảm bảo số dương nhỏ nhất vẫn được hệ thống chấp nhận bình thường. |
| **TC_FR16_EP07** | $R_1, EXT_1$, **$P_3$ (Giá số âm)**, $N_1, C_1$ | Role Admin, file CSV chứa dòng:<br>`"Chuột lỗi",-50000,"Mô tả",,1` ($P < 0$) | Báo lỗi giá âm. Trả về HTTP 400. Toàn bộ file bị Rollback, CSDL giữ nguyên trạng thái cũ. | Phủ lớp không hợp lệ $P_3$ (Out-point). Đảm bảo giá trị âm bị từ chối triệt để. |
| **TC_FR16_EP08** | $R_1, EXT_1$, **$P_4$ (Giá không phải số)**, $N_1, C_1$ | Role Admin, file CSV chứa dòng:<br>`"Tai nghe","Mười ngàn",,1` | Báo lỗi sai định dạng kiểu dữ liệu số tại cột `price`. Rollback toàn bộ giao dịch. | Phủ lớp không hợp lệ $P_4$. Kiểm tra khả năng bắt lỗi kiểu dữ liệu (Data Type Parsing). |
| **TC_FR16_EP09** | $R_1, EXT_1, N_1, P_1$, **$C_2$ (Danh mục không tồn tại)** | Role Admin, file CSV chứa dòng:<br>`"Màn hình",3000000,"",,9999` ($C = 9999$) | Báo lỗi vi phạm danh mục: "Category ID 9999 không tồn tại trong hệ thống". Rollback toàn bộ 100%. | Phủ lớp không hợp lệ $C_2$. Xác minh ràng buộc toàn vẹn quan hệ giữa bảng Sản phẩm và Danh mục. |
| **TC_FR16_EP10** | $R_1, EXT_1, N_1, P_1, C_1$, **RFC 4180 (Có dấu phẩy bọc nháy kép)** | Role Admin, file CSV chứa dòng:<br>`"Laptop Gaming",20000000,"Màn hình 144Hz, Ram 16GB, SSD 512GB",,1` | Parse tệp chính xác theo chuẩn RFC 4180. Trường description lưu trọn vẹn cả chuỗi có dấu phẩy. Không bị nhảy lệch cột. | Kiểm thử tính năng hỗ trợ chuẩn RFC 4180 theo SRS FR-16. Đảm bảo parser CSV không chia tách nhầm cột khi gặp dấu phẩy trong ngoặc kép. |
| **TC_FR16_EP11** | $R_1, EXT_1$, **$B_2$ (Kiểm thử thực nghiệm Rollback)** | Role Admin, file CSV gồm 5 dòng:<br>- Dòng 1, 2, 3: Dữ liệu hợp lệ.<br>- Dòng 4: Sai giá ($P = -10,000$).<br>- Dòng 5: Dữ liệu hợp lệ. | Báo lỗi tại dòng 4. **Xác nhận thực tế trong CSDL**: Không có bất kỳ dòng nào (kể cả dòng 1, 2, 3) được lưu vào DB (Toàn vẹn 100%). | Phủ lớp $B_2$. Chứng minh nguyên lý giao dịch nguyên tử All-or-nothing, không để lại rác dữ liệu một phần trong hệ thống. |

---

## Prompt 11: Thiết kế Domain Testing cho Phân hệ Mobile (FR-20) — Luồng Thanh toán, Áp dụng Coupon & State Machine Hủy đơn (FR-10)
* **Mục tiêu:** Kiểm thử chức năng thanh toán trên ứng dụng di động React Native, bao gồm xác thực thông tin đơn, áp dụng mã giảm giá (ràng buộc ngưỡng tiền tối thiểu, hạn sử dụng, số lượt dùng) và kiểm soát máy trạng thái hủy đơn hàng tuân thủ nghiêm ngặt FR-10.
* **Đầu ra:** Bảng phân tích biến đầu vào, phân chia lớp tương đương và bộ Test Cases chuẩn 5 cột (ID, Lớp tương đương, Input, Expected result, Giải thích).

```text
VAI TRÒ: Kiểm thử viên Phần mềm (QA / Software Tester).
TÍNH NĂNG: Phân hệ Mobile (React Native) — FR-20 kết hợp FR-08, FR-09 (Mã giảm giá) và FR-10 (State Machine).

MÔ TẢ NGHIỆP VỤ:
1. Luồng Thanh toán Mobile:
   - Người dùng đăng nhập, có sản phẩm trong giỏ hàng, nhập địa chỉ nhận hàng và chọn phương thức thanh toán.
2. Áp dụng Mã Giảm Giá (FR-09):
   - Mã 'SAVE10': Giảm 10%, Ngưỡng đơn hàng tối thiểu min_order_amount = 300,000 ₫, Hạn dùng 2099-12-31, Giới hạn max_uses_per_user = 1.
   - Nếu tổng đơn < 300,000 ₫ -> Từ chối mã, báo lỗi chưa đủ điều kiện.
   - Nếu người dùng đã dùng mã 1 lần -> Từ chối mã do hết lượt cho phép.
   - Nếu mã hết hạn (ví dụ mã 'EXPIRED') -> Từ chối mã.
3. State Machine Hủy đơn hàng trên Mobile (FR-10 & FR-20):
   - Đơn hàng có 5 trạng thái: pending -> confirmed -> shipping -> delivered (hoặc canceled).
   - Trên Mobile, người dùng CHỈ ĐƯỢC PHÉP bấm nút "Hủy đơn hàng" khi đơn ở trạng thái 'pending' hoặc 'confirmed'.
   - Khi đơn chuyển sang 'shipping', nút Hủy trên Mobile phải BỊ ẨN hoàn toàn; nếu cố tình gọi API hủy thì hệ thống trả về HTTP 400 Bad Request.
   - Trạng thái 'delivered' và 'canceled' là trạng thái kết thúc (Final State), không thể thay đổi hay hủy lại.

HÃY ÁP DỤNG DOMAIN TESTING (PHÂN HOẠCH TƯƠNG ĐƯƠNG):
1. Liệt kê từng biến đầu vào và miền giá trị.
2. Chia lớp tương đương hợp lệ / không hợp lệ cho từng biến, giải thích lý do chia.
3. Thiết kế test case, mỗi test case chỉ phủ 1 lớp không hợp lệ (Single Fault Assumption).
4. Xuất bảng: ID | Lớp tương đương | Input | Expected result | Giải thích.
```

### Kết quả Thực thi & Phân tích Chi tiết từ AI:

#### 1. Liệt kê từng biến đầu vào và miền giá trị
* **Biến 1: `order_total` ($T \in \mathbb{R}^+$):** Tổng giá trị đơn hàng trước giảm giá (₫). Miền giá trị: $(0, +\infty)$.
* **Biến 2: `coupon_code` (String):** Chuỗi mã giảm giá người dùng nhập trên Mobile. Miền giá trị: Mã có trong hệ thống (`SAVE10`, `BIGBUY`, `EXPIRED`), mã không tồn tại, hoặc để trống.
* **Biến 3: `coupon_user_usage` ($U \in \mathbb{N}$):** Số lần người dùng hiện tại đã dùng thành công mã này trong quá khứ. Miền giá trị: $\{0, 1, 2, \dots\}$.
* **Biến 4: `shipping_address` (String):** Chuỗi địa chỉ nhận hàng nhập trên Mobile UI. Miền giá trị: Chuỗi ký tự độ dài $L \ge 0$.
* **Biến 5: `payment_method` (Enum):** Hình thức thanh toán người dùng lựa chọn trên Mobile. Miền giá trị: `COD`, `BANKING`, `MOMO`.
* **Biến 6: `order_status` ($S \in \mathcal{S}$):** Trạng thái hiện tại của đơn hàng khi người dùng thao tác Hủy đơn trên Mobile. Miền giá trị: $\mathcal{S} = \{ \text{'pending'}, \text{'confirmed'}, \text{'shipping'}, \text{'delivered'}, \text{'canceled'} \}$.

#### 2. Phân chia lớp tương đương hợp lệ / không hợp lệ & Giải thích
* **Đối với biến `order_total` ($T$) khi áp mã `SAVE10` ($\text{ngưỡng} \ge 300,000$ ₫):**
  * **$T_1$ (Hợp lệ):** $T \ge 300,000$ ₫. *Lý do:* Thỏa mãn điều kiện C3 của coupon theo SRS FR-09. Điểm biên On: $T = 300,000$ ₫; In: $T = 500,000$ ₫.
  * **$T_2$ (Không hợp lệ):** $0 < T < 300,000$ ₫. *Lý do:* Chưa đạt giá trị tối thiểu để kích hoạt khuyến mãi. Điểm cận biên Off: $T = 299,999$ ₫.
* **Đối với biến `coupon_code`:**
  * **$CP_1$ (Hợp lệ):** Mã đang hoạt động và còn hạn (`SAVE10`). *Lý do:* Đáp ứng điều kiện C1 và C2.
  * **$CP_2$ (Không hợp lệ):** Mã không tồn tại trong hệ thống (`INVALIDCODE`). *Lý do:* Vi phạm điều kiện C1 (mã tồn tại).
  * **$CP_3$ (Không hợp lệ):** Mã đã hết hạn sử dụng (`EXPIRED`, hết hạn năm 2020). *Lý do:* Vi phạm điều kiện C2 (còn hạn dùng).
* **Đối với số lượt dùng mã `coupon_user_usage` ($U$) với `SAVE10` ($\text{max} = 1$ lần):**
  * **$U_1$ (Hợp lệ):** $U = 0$ ($U < 1$ là True). *Lý do:* Người dùng chưa từng áp dụng mã này, thỏa mãn điều kiện C5.
  * **$U_2$ (Không hợp lệ):** $U \ge 1$ ($U < 1$ là False, ví dụ $U = 1$). *Lý do:* Đã sử dụng hết hạn mức cho phép theo điều kiện C5.
* **Đối với máy trạng thái đơn hàng khi thực hiện hành vi HỦY ĐƠN ($S$):**
  * **$S_{valid}$ (Hợp lệ - Cho phép Mobile hủy):** $S \in \{ \text{'pending'}, \text{'confirmed'} \}$. *Lý do:* SRS FR-10 và FR-20 quy định người dùng được phép tự hủy khi đơn chưa giao đi.
  * **$S_{border}$ (Không hợp lệ - Biên chặn chuyển đổi):** $S = \text{'shipping'}$. *Lý do:* Hàng đang vận chuyển, User không được tự hủy. Nút Hủy phải bị ẩn trên Mobile; API chặn trả về 400.
  * **$S_{final}$ (Không hợp lệ - Trạng thái kết thúc):** $S \in \{ \text{'delivered'}, \text{'canceled'} \}$. *Lý do:* Trạng thái đóng vĩnh viễn, không cho phép hủy lại hay khôi phục.

#### 3. Bảng Test Case chuẩn Domain Testing Phân hệ Mobile (Single Fault Assumption)

| ID | Lớp tương đương | Input | Expected result | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| **TC_MOB_EP01** | $T_1, CP_1, U_1, L_1, PM_1$ (Happy Path) | $T = 500,000$ ₫<br>Mã `SAVE10`<br>$U = 0$ (Chưa dùng)<br>`address: "123 CMT8, Q3, TP.HCM"`<br>Phương thức: `COD` | Áp dụng mã thành công: Giảm 50,000 ₫ (10%). Tổng tiền còn lại $450,000$ ₫. Bấm "Đặt hàng" $\rightarrow$ Tạo đơn thành công, trạng thái `pending`, bộ đếm lượt dùng tăng $U = 1$. | Kiểm thử luồng thanh toán và áp dụng coupon hoàn chỉnh trên Mobile với tất cả biến hợp lệ. |
| **TC_MOB_EP02** | **$T_2$ (Dưới ngưỡng tối thiểu - Off min)**, $CP_1, U_1$ | $T = 299,999$ ₫<br>Nhập mã `SAVE10`<br>$U = 0$ | Giao diện hiển thị Toast lỗi: "Đơn hàng tối thiểu phải từ 300,000 ₫ để áp dụng mã này". Tiền giữ nguyên $299,999$ ₫. | Phủ lớp không hợp lệ $T_2$ tại điểm cận biên Off ($T = 299,999$ ₫). Kiểm tra chặn coupon khi thiếu 1 đồng. |
| **TC_MOB_EP03** | $T_1$ (Đúng ngưỡng tối thiểu - On min), $CP_1, U_1$ | $T = 300,000$ ₫<br>Nhập mã `SAVE10`<br>$U = 0$ | Áp dụng mã THÀNH CÔNG: Giảm đúng 30,000 ₫ (10%). Tổng tiền thanh toán cập nhật còn $270,000$ ₫. | Kiểm thử điểm biên On hợp lệ ($T = 300,000$ ₫). Xác nhận hệ thống chấp nhận mã ngay tại ranh giới đóng. |
| **TC_MOB_EP04** | $T_1$, **$CP_2$ (Mã không tồn tại)**, $U_1$ | $T = 500,000$ ₫<br>Nhập mã `INVALIDCODE`<br>$U = 0$ | Báo lỗi trên màn hình: "Mã giảm giá không tồn tại". Tổng tiền không đổi. | Phủ lớp không hợp lệ $CP_2$. Chỉ duy nhất chuỗi mã giảm giá bị sai để kiểm tra xử lý mã rác. |
| **TC_MOB_EP05** | $T_1$, **$CP_3$ (Mã đã hết hạn)**, $U_1$ | $T = 500,000$ ₫<br>Nhập mã `EXPIRED`<br>$U = 0$ | Báo lỗi: "Mã giảm giá đã hết hạn sử dụng". Không áp dụng chiết khấu. | Phủ lớp không hợp lệ $CP_3$. Kiểm tra kiểm soát điều kiện thời gian hết hạn của coupon. |
| **TC_MOB_EP06** | $T_1, CP_1$, **$U_2$ (Hết lượt sử dụng - Biên On)** | $T = 500,000$ ₫<br>Nhập mã `SAVE10`<br>$U = 1$ (User đã dùng 1 lần trước đó) | Báo lỗi: "Bạn đã sử dụng hết lượt cho phép đối với mã giảm giá này". Từ chối áp dụng mã. | Phủ lớp không hợp lệ $U_2$ tại điểm On ($U = 1$). Ngăn chặn người dùng lạm dụng lặp lại mã giảm giá 1 lần. |
| **TC_MOB_EP07** | $T_1, CP_1, U_1$, **$L_2$ (Địa chỉ rỗng trên Mobile)** | $T = 500,000$ ₫, mã `SAVE10`<br>`address: ""` (Để trống) | Mobile hiển thị cảnh báo đỏ dưới ô nhập địa chỉ: "Vui lòng nhập địa chỉ nhận hàng". Nút "Đặt hàng" bị disabled. | Phủ lớp không hợp lệ $L_2$ trên giao diện Mobile. Đảm bảo trải nghiệm người dùng không bị crash khi quên nhập địa chỉ. |
| **TC_MOB_EP08** | $S_{valid}$ (Hủy đơn khi `pending`) | Đơn hàng đang ở trạng thái `status: 'pending'` | Màn hình Chi tiết đơn hàng hiển thị nút "Hủy đơn hàng". Bấm nút và xác nhận $\rightarrow$ Trạng thái chuyển sang `canceled` thành công. | Kiểm thử tính năng hủy hợp lệ khi đơn mới tạo theo đúng State Machine FR-10 và FR-20. |
| **TC_MOB_EP09** | $S_{valid}$ (Hủy đơn khi `confirmed`) | Đơn hàng đang ở trạng thái `status: 'confirmed'` | Nút "Hủy đơn hàng" vẫn hiển thị. Bấm hủy $\rightarrow$ Đơn hàng chuyển sang `canceled` thành công. | Kiểm thử tính năng hủy hợp lệ khi Admin đã xác nhận nhưng chưa giao hàng. |
| **TC_MOB_EP10** | **$S_{border}$ (Cấm hủy khi `shipping` - Biên ranh giới)** | Đơn hàng đang ở trạng thái `status: 'shipping'` (Đang giao) | **Nút "Hủy đơn hàng" BỊ ẨN hoàn toàn trên Mobile UI**. Nếu dùng công cụ can thiệp gọi API `PUT /api/orders/:id/cancel` $\rightarrow$ Trả về HTTP 400 Bad Request: "Không thể hủy đơn hàng đang vận chuyển". | Phủ lớp không hợp lệ ranh giới $S_{border}$. Đảm bảo ràng buộc nghiệp vụ khắt khe: Khi hàng đã xuất kho vận chuyển, khách hàng không được quyền tự ý hủy. |
| **TC_MOB_EP11** | **$S_{final}$ (Cấm chuyển đổi trạng thái kết thúc)** | Đơn hàng đang ở trạng thái `status: 'delivered'` (Đã giao) hoặc `canceled` (Đã hủy) | Giao diện hiển thị nhãn trạng thái tĩnh đóng. Không có bất kỳ tùy chọn Hủy hay sửa đổi trạng thái nào. | Phủ lớp $S_{final}$. Đảm bảo tính bất biến của trạng thái kết thúc trong sơ đồ máy trạng thái FR-10. |

---

## 4. Bảng Tổng Hợp Độ Phủ Domain Testing Toàn Diện

| Tính năng nghiệp vụ | Số lượng Test Case | Các lớp tương đương & Điểm biên đã bao phủ | Tuân thủ Single Fault | Trạng thái |
| :--- | :---: | :--- | :---: | :---: |
| **FR-02: Đăng nhập & Khóa tài khoản** | 11 Cases | $E_1 \rightarrow E_4$, $P_1 \rightarrow P_3$, $F$ On/Off ($F=2, 3$), Time On/Off ($T=30s, 31s$), $F=-1$ | 100% | Đạt yêu cầu |
| **FR-08: Thanh toán (Checkout)** | 9 Cases | $A_1 \rightarrow A_2$, $Q_1 \rightarrow Q_2$, $M$ On/Off ($M=0, 1$), $M < 0$, Address On/Off ($L=0, 1, 255, 256$), Giá giả mạo | 100% | Đạt yêu cầu |
| **FR-16: Import CSV (Web Admin)** | 11 Cases | $R_1 \rightarrow R_2$, $EXT_1 \rightarrow EXT_2$, Name On/Off ($N=0, 1$), Price On/Off ($P=0, 1$), $P < 0$, RFC 4180, Atomic Rollback | 100% | Đạt yêu cầu |
| **Mobile: Thanh toán & State Machine (FR-20)** | 11 Cases | Coupon Total On/Off ($T=299k, 300k$), Coupon Expiry/Codes, Usage On/Off ($U=0, 1$), Address, State Machine ($S_{valid}, S_{border}, S_{final}$) | 100% | Đạt yêu cầu |
| **TỔNG CỘNG** | **42 Cases** | **Bao phủ 100% không gian miền giá trị, các điểm biên On/Off/In/Out và State Transitions** | **100%** | **Sẵn sàng bàn giao** |
