# BÁO CÁO KIỂM THỬ MIỀN (DOMAIN TESTING REPORT)
## TÍNH NĂNG: FR-02 — ĐĂNG NHẬP & KHÓA TÀI KHOẢN (POOL A)

---

## 1. TỔNG QUAN & ĐẶC TẢ NGHIỆP VỤ (SRS FR-02)
* **Thuộc nhóm tính năng:** Pool A — Authentication, Categories, and Products.
* **Mô tả nghiệp vụ:**
  1. Người dùng nhập `email` và `password` để đăng nhập hệ thống.
  2. Trường `email` phải có định dạng hợp lệ (`user@domain.com`).
  3. Mật khẩu phải khớp với mật khẩu đã lưu trong cơ sở dữ liệu.
  4. Cơ chế chống dò quét mật khẩu (Brute-force Protection): Mỗi lần đăng nhập thất bại, hệ thống tăng số lần thử sai `failed_attempts` lên đúng 1 đơn vị.
  5. Nếu đăng nhập sai liên tiếp từ 3 lần trở lên ($F \ge 3$), tài khoản bị tạm khóa trong vòng 30 giây (môi trường demo). Trong thời gian bị khóa, mọi yêu cầu đăng nhập dù đúng mật khẩu vẫn bị từ chối với HTTP 403 Forbidden.
  6. Sau khi hết 30 giây, tài khoản tự động mở khóa và đăng nhập thành công nếu nhập đúng mật khẩu.

---

## 2. LIỆT KÊ BIẾN ĐẦU VÀO VÀ MIỀN GIÁ TRỊ

| Tên biến | Kiểu dữ liệu | Miền giá trị lý thuyết | Ý nghĩa |
| :--- | :---: | :--- | :--- |
| `email` ($E$) | String | Chuỗi định dạng email RFC 5322 | Tài khoản đăng nhập của người dùng. |
| `password` ($P$) | String | Chuỗi ký tự độ dài $L \ge 0$ | Mật khẩu truy cập hệ thống. |
| `failed_attempts` ($F$) | Integer | $\mathbb{Z}_{\ge 0} = \{0, 1, 2, 3, \dots\}$ | Số lần nhập sai mật khẩu liên tiếp được lưu trong CSDL. |
| `lockout_duration` ($T$) | Integer | Thời gian trôi qua tính bằng giây ($s$) | Khoảng thời gian kể từ thời điểm tài khoản bị khóa. |

---

## 3. PHÂN CHIA LỚP TƯƠNG ĐƯƠNG & GIẢI THÍCH LÝ DO

### 3.1. Biến `email` ($E$)
* **$E_1$ (Hợp lệ):** Email đúng định dạng và đã đăng ký trong hệ thống (`test@eshop.com`).
* **$E_2$ (Không hợp lệ):** Email đúng định dạng nhưng chưa từng đăng ký (`unregistered@eshop.com`).
* **$E_3$ (Không hợp lệ):** Email để trống `""` ($L = 0$).
* **$E_4$ (Không hợp lệ):** Email sai định dạng (thiếu `@`, thiếu domain như `test_eshop.com`).

### 3.2. Biến `password` ($P$)
* **$P_1$ (Hợp lệ):** Mật khẩu chính xác khớp với tài khoản (`Test1234!`).
* **$P_2$ (Không hợp lệ):** Mật khẩu sai (`WrongPass!`).
* **$P_3$ (Không hợp lệ):** Mật khẩu để trống `""`.

### 3.3. Biến bộ đếm đăng nhập sai `failed_attempts` ($F$)
* **$F_1$ (Hợp lệ - Chưa bị khóa):** $0 \le F \le 2$. Tài khoản ở trạng thái bình thường.
* **$F_2$ (Không hợp lệ - Kích hoạt khóa):** $F \ge 3$. Tài khoản bị tạm khóa trong 30 giây. Điểm On-point là $F = 3$; Off-point là $F = 2$.

### 3.4. Biến thời gian khóa `lockout_duration` ($T$)
* **$T_1$ (Trong thời gian khóa):** $0 \le T \le 30$ giây. Hệ thống từ chối đăng nhập.
* **$T_2$ (Hết thời gian khóa):** $T > 30$ giây. Hệ thống tự động mở khóa. Điểm On-point là $T = 30$s; Off-point là $T = 31$s.

---

## 4. BẢNG TEST CASES DOMAIN TESTING (SINGLE FAULT ASSUMPTION)

| Test ID | Lớp tương đương | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected Result) | Kết quả thực tế trên SUT (Actual Result) | Trạng thái (Pass/Fail) |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC_FR02_01** | $E_1, P_1, F_1$ | `email: "test@eshop.com"`, pass đúng | Đăng nhập thành công (HTTP 200 OK), nhận Token. | Đăng nhập thành công, nhận JWT Token. | **PASS** |
| **TC_FR02_02** | $E_1$, **$P_2$ (Sai pass)** | Email đúng, mật khẩu sai lần 1 | HTTP 401 Unauthorized, bộ đếm $F$ tăng từ 0 lên 1. | HTTP 401, nhưng **bộ đếm F trong DB tăng vọt lên 2**! | **FAIL (BUG-FR02-01)** |
| **TC_FR02_03** | **$E_2$ (Chưa đăng ký)** | Email chưa đăng ký, pass bất kỳ | HTTP 401 Unauthorized, không lộ tài khoản. | HTTP 401: "Invalid email or password". | **PASS** |
| **TC_FR02_04** | **$E_3$ (Email rỗng)** | `email: ""` | Giao diện chặn: "Vui lòng nhập email". | Giao diện chặn submit thành công. | **PASS** |
| **TC_FR02_05** | **$P_3$ (Pass rỗng)** | `pass: ""` | Giao diện chặn: "Vui lòng nhập mật khẩu". | Giao diện chặn submit thành công. | **PASS** |
| **TC_FR02_06** | **$E_4$ (Email sai format)** | `email: "invalid_format"` | Báo lỗi định dạng email HTML5. | Trình duyệt báo lỗi email hợp lệ. | **PASS** |
| **TC_FR02_07** | **$F_2$ (Sai lần 3 - On)** | Nhập sai liên tiếp lần thứ 3 ($F=3$) | Khóa tài khoản trong 30 giây (HTTP 403 Forbidden). | Bị khóa ngay từ lần 2 do lỗi tăng 2 đơn vị; khóa 180s. | **FAIL (BUG-FR02-01)** |
| **TC_FR02_08** | **$T_1$ (Trong lúc khóa)** | Nhập **Pass ĐÚNG** tại giây thứ $T=10$s | **VẪN BỊ TỪ CHỐI** (HTTP 403 Forbidden). | HTTP 403 Forbidden: "Tài khoản đã bị khóa". | **PASS** |
| **TC_FR02_09** | **$T_2$ (Mở khóa sau 31s)**| Nhập Pass ĐÚNG tại giây thứ $T=31$s | Đăng nhập thành công (HTTP 200 OK), reset $F=0$. | Tài khoản vẫn bị khóa vì SUT khóa tới 180s (3 phút)! | **FAIL (BUG-FR02-02)** |
| **TC_FR02_10** | SQL Injection | `email: "' OR '1'='1' --"` | Trả về 401 Unauthorized an toàn. | Trả về 401 Unauthorized an toàn, không bị SQLi. | **PASS** |
