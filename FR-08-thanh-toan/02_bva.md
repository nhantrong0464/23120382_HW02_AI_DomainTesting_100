# PHÂN TÍCH GIÁ TRỊ BIÊN (BOUNDARY VALUE ANALYSIS - BVA)
## TÍNH NĂNG: FR-08 — THANH TOÁN (CHECKOUT) TRÊN ESHOP SUT

---

## 1. CƠ SỞ LÝ THUYẾT VỀ PHÂN TÍCH GIÁ TRỊ BIÊN (BVA)

Phân tích giá trị biên (Boundary Value Analysis - BVA) là kỹ thuật kiểm thử phần mềm dựa trên tiên đề: **"Xác suất xuất hiện lỗi tại ranh giới của miền đầu vào cao hơn rất nhiều so với bên trong miền"**. Lỗi biên thường xuất phát từ việc lập trình viên nhầm lẫn các toán tử quan hệ (ví dụ: dùng `>` thay vì `>=`, `<` thay vì `<=`, hoặc sai sót trong điều kiện ngắt vòng lặp/bộ đếm off-by-one).

### Các định nghĩa điểm kiểm thử biên:
* **On-point (Điểm trên biên):** Điểm nằm chính xác trên đường biên phân chia (Boundary Line).
  * Nếu miền đóng ($\le$ hoặc $\ge$), On-point thuộc về miền hợp lệ.
  * Nếu miền mở ($<$ hoặc $>$), On-point không thuộc về miền hợp lệ (nằm ngoài miền).
* **Off-point (Điểm cận biên):** Điểm nằm sát cạnh On-point nhất có thể:
  * Nếu On-point thuộc miền hợp lệ, thì Off-point nằm ngay ngoài miền hợp lệ.
  * Nếu On-point nằm ngoài miền hợp lệ (miền mở), thì Off-point nằm ngay bên trong miền hợp lệ.
* **In-point (Điểm trong miền):** Điểm nằm sâu bên trong miền hợp lệ, cách xa các ranh giới.
* **Out-point (Điểm ngoài miền):** Điểm nằm sâu bên ngoài miền hợp lệ.

---

## 2. XÁC ĐỊNH RANH GIỚI VÀ CÁC ĐIỂM BIÊN CHO TỪNG BIẾN

### 2.1. Biến tổng tiền thanh toán: `total_amount` ($M \in \mathbb{R}$)
* **Điều kiện nghiệp vụ:** $M > 0$ ₫ (Miền mở tại 0).
* **Ranh giới:** $B_M = 0$.
* **Xác định các điểm BVA:**
  * **On-point:** $M = 0$ ₫ (Nằm ngay trên ranh giới; do điều kiện là $> 0$ nên $M = 0$ là **Không hợp lệ**).
  * **Off-point:** $M = 1$ ₫ (Điểm nguyên nhỏ nhất sát ranh giới nằm **Bên trong miền hợp lệ**).
  * **In-point:** $M = 500,000$ ₫ (Giá trị mua sắm điển hình, hợp lệ).
  * **Out-point:** $M = -1$ ₫ hoặc $-50,000$ ₫ (Nằm hẳn ngoài miền hợp lệ, giá trị âm phi lý).

### 2.2. Biến độ dài chuỗi địa chỉ nhận hàng: `shipping_address` ($L \in \mathbb{N}$)
* **Điều kiện nghiệp vụ:** $1 \le L \le 255$ ký tự (Miền đóng ở cả hai đầu).
* **Ranh giới dưới (Lower Boundary $B_{L1} = 1$):**
  * **On-point:** $L = 1$ (Chuỗi có độ dài 1 ký tự, thuộc miền hợp lệ).
  * **Off-point:** $L = 0$ (Chuỗi rỗng `""`, nằm ngay ngoài miền hợp lệ $\rightarrow$ **Không hợp lệ**).
* **Ranh giới trên (Upper Boundary $B_{L2} = 255$):**
  * **On-point:** $L = 255$ (Chuỗi có độ dài đúng 255 ký tự, thuộc miền hợp lệ).
  * **Off-point:** $L = 256$ (Chuỗi có độ dài 256 ký tự, nằm ngay ngoài miền hợp lệ $\rightarrow$ **Không hợp lệ**).
* **In-point:** $L = 35$ ký tự (Địa chỉ thực tế hợp lệ: "123 CMT8, Phường 5, Quận 3, TP.HCM").

### 2.3. Biến số lượng mặt hàng trong giỏ: `cart_item_count` ($Q \in \mathbb{N}$)
* **Điều kiện nghiệp vụ:** $Q \ge 1$ (Miền đóng tại 1).
* **Ranh giới:** $B_Q = 1$.
* **Xác định các điểm BVA:**
  * **On-point:** $Q = 1$ (Giỏ hàng có đúng 1 sản phẩm, **Hợp lệ**).
  * **Off-point:** $Q = 0$ (Giỏ hàng rỗng, nằm ngoài ranh giới $\rightarrow$ **Không hợp lệ**).
  * **In-point:** $Q = 5$ (Giỏ hàng có 5 sản phẩm).

### 2.4. Biến ngưỡng đơn hàng tối thiểu của Coupon: `min_order_amount` ($T$)
Áp dụng mã giảm giá `SAVE10` với điều kiện $T \ge 300,000$ ₫:
* **Ranh giới:** $B_T = 300,000$ ₫ (Miền đóng).
* **Xác định các điểm BVA:**
  * **On-point:** $T = 300,000$ ₫ (**Hợp lệ** - Đủ điều kiện kích hoạt giảm 10% theo SRS).
  * **Off-point:** $T = 299,999$ ₫ (**Không hợp lệ** - Thiếu đúng 1 đồng, hệ thống bắt buộc phải từ chối coupon).
  * **In-point:** $T = 500,000$ ₫ (**Hợp lệ** - Đơn hàng giá trị cao).
  * *(Lưu ý đặc biệt từ mã nguồn SUT: Backend triển khai sai lệch với `if (total_amount > coupon.min_order_amount)`, do đó điểm On-point $T = 300,000$ ₫ sẽ bị lỗi nghiêm trọng).*

### 2.5. Biến số lượt dùng mã của người dùng: `coupon_user_usage` ($U \in \mathbb{N}$)
Mã `SAVE10` giới hạn tối đa 1 lần dùng cho mỗi khách hàng ($U < 1$):
* **Ranh giới:** $B_U = 1$.
* **Xác định các điểm BVA:**
  * **On-point:** $U = 1$ (Đã dùng 1 lần $\rightarrow$ $U < 1$ là False $\rightarrow$ **Không hợp lệ**, phải chặn).
  * **Off-point:** $U = 0$ (Chưa từng dùng $\rightarrow$ $U < 1$ là True $\rightarrow$ **Hợp lệ**, cho phép áp dụng).

### 2.6. Ranh giới chuyển đổi trạng thái hủy đơn (Order State Machine Boundary - FR-10)
Sơ đồ chuyển đổi trạng thái: `pending` $\rightarrow$ `confirmed` $\rightarrow$ `shipping` $\rightarrow$ `delivered`.
* **Biên trong (Hợp lệ để hủy):**
  * `pending`: Cho phép người dùng bấm hủy.
  * `confirmed`: Cho phép người dùng bấm hủy.
* **Biên ngoài (Ranh giới cấm hủy):**
  * `shipping`: Hàng đã xuất kho, **CẤM HỦY**. Nút Hủy trên Mobile phải bị ẩn hoàn toàn; API phải chặn và trả về HTTP `400 Bad Request`.
  * `delivered` / `canceled`: Trạng thái kết thúc tĩnh, không thể hoàn tác.

---

### 2.7. Bảng Phân Tích Biên 7 Điểm (Robust BVA Matrix)
Áp dụng đầy đủ 7 giá trị biên: `min - 1`, `min`, `min + 1`, `nominal`, `max - 1`, `max`, `max + 1`:

| Biến kiểm thử | `min - 1` | `min` | `min + 1` | `nominal` | `max - 1` | `max` | `max + 1` |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `shipping_address` ($L$) | $0$ ký tự *(Lỗi)* | $1$ ký tự *(Đạt)* | $2$ ký tự *(Đạt)* | $128$ ký tự *(Đạt)* | $254$ ký tự *(Đạt)* | $255$ ký tự *(Đạt)* | $256$ ký tự *(Lỗi)* |
| `cart_item_count` ($Q$) | $0$ SP *(Lỗi)* | $1$ SP *(Đạt)* | $2$ SP *(Đạt)* | $10$ SP *(Đạt)* | $99$ SP *(Đạt)* | $100$ SP *(Đạt)* | $101$ SP *(Lỗi)* |
| `total_amount` ($M$ ₫) | $0$ ₫ *(Lỗi)* | $1$ ₫ *(Đạt)* | $2$ ₫ *(Đạt)* | $500,000$ ₫ *(Đạt)* | $99,999,999$ ₫ *(Đạt)* | $100,000,000$ ₫ *(Đạt)* | $100,000,001$ ₫ *(Lỗi)* |


## 3. BẢNG TEST CASES CHI TIẾT THEO PHÂN TÍCH GIÁ TRỊ BIÊN (BVA MATRIX)

| Test ID | Biến kiểm thử biên | Loại điểm biên | Giá trị đầu vào (Input) | Kết quả mong đợi (Expected Result) | Nhận xét kỹ thuật |
| :--- | :--- | :---: | :--- | :--- | :--- |
| **BVA_M_01** | `total_amount` | **On-point** ($M=0$) | $M = 0$ ₫ | API trả về HTTP `400 Bad Request`: "Tổng tiền phải lớn hơn 0". Không tạo đơn. | Kiểm tra biên mở $M > 0$. Xác nhận hệ thống không chấp nhận đơn hàng 0 đồng. |
| **BVA_M_02** | `total_amount` | **Off-point** ($M=1$) | $M = 1$ ₫ | Chấp nhận thanh toán thành công (HTTP `201 Created`). | Kiểm tra giá trị dương nhỏ nhất có thể thanh toán. |
| **BVA_M_03** | `total_amount` | **Out-point** ($M=-1$) | $M = -1$ ₫ | Backend từ chối với HTTP `400 Bad Request`. | Kiểm tra chặn giá trị tiền âm ngay sát biên 0. |
| **BVA_L_01** | `shipping_address` | **Off min** ($L=0$) | `address: ""` ($L = 0$) | Giao diện chặn submit; API trả về HTTP `400 Bad Request`: "Địa chỉ không được để trống". | Kiểm tra cận biên dưới của trường địa chỉ. |
| **BVA_L_02** | `shipping_address` | **On min** ($L=1$) | `address: "A"` ($L = 1$) | Chấp nhận tạo đơn thành công (HTTP `201 Created`). | Kiểm tra độ dài địa chỉ hợp lệ tối thiểu 1 ký tự. |
| **BVA_L_03** | `shipping_address` | **On max** ($L=255$) | `address:` chuỗi 255 ký tự 'X' | Chấp nhận tạo đơn thành công, lưu đủ 255 ký tự vào CSDL SQLite. | Kiểm tra độ dài tối đa cho phép của cột `VARCHAR(255)`. |
| **BVA_L_04** | `shipping_address` | **Off max** ($L=256$) | `address:` chuỗi 256 ký tự 'X' | Giao diện hiển thị lỗi độ dài; API từ chối với HTTP `400 Bad Request`. | Kiểm tra cận biên trên vượt quá giới hạn an toàn. |
| **BVA_Q_01** | `cart_item_count` | **On-point** ($Q=1$) | Giỏ hàng có 1 sản phẩm | Cho phép thanh toán thành công, trừ kho đúng 1 sản phẩm. | Kiểm tra biên số lượng tối thiểu để kích hoạt checkout. |
| **BVA_Q_02** | `cart_item_count` | **Off-point** ($Q=0$) | Giỏ hàng rỗng ($Q=0$) | Vô hiệu hóa nút thanh toán; API trả về HTTP `400 Bad Request`. | Kiểm tra chặn checkout khi giỏ hàng trống. |
| **BVA_CP_01** | `min_order_amount` | **On-point** ($T=300k$) | $T = 300,000$ ₫, mã `SAVE10` | Áp dụng mã THÀNH CÔNG, giảm 30,000 ₫ (10%), tổng thanh toán còn $270,000$ ₫. | **Điểm bẫy lỗi SUT:** Kiểm tra toán tử so sánh ngưỡng `>=` vs `>`. |
| **BVA_CP_02** | `min_order_amount` | **Off-point** ($T=299,999k$) | $T = 299,999$ ₫, mã `SAVE10` | Báo lỗi: "Đơn hàng tối thiểu phải từ 300,000 ₫ để áp dụng mã này". Tiền giữ nguyên. | Kiểm tra chặn coupon khi thiếu đúng 1 đồng. |
| **BVA_US_01** | `coupon_user_usage`| **Off-point** ($U=0$) | User chưa từng dùng mã ($U=0$) | Áp dụng mã thành công, tăng số lần dùng lên $U = 1$. | Kiểm tra điều kiện chưa chạm ngưỡng giới hạn. |
| **BVA_US_02** | `coupon_user_usage`| **On-point** ($U=1$) | User đã dùng 1 lần ($U=1$) | Từ chối mã, báo lỗi: "Bạn đã sử dụng hết lượt cho phép". | Kiểm tra điểm chạm ranh giới giới hạn sử dụng 1 lần. |
| **BVA_ST_01** | State Machine | **Biên trong** (`confirmed`) | Đơn hàng trạng thái `confirmed` | Nút "Hủy đơn" hiển thị; bấm hủy chuyển thành công sang `canceled`. | Kiểm tra trạng thái cuối cùng người dùng còn được phép hủy. |
| **BVA_ST_02** | State Machine | **Biên chặn** (`shipping`) | Đơn hàng trạng thái `shipping` | **Nút "Hủy đơn" BỊ ẨN trên Mobile**; API chặn trả về HTTP `400 Bad Request`. | **Điểm bẫy lỗi SUT:** Kiểm tra ngăn chặn hủy đơn hàng đang vận chuyển. |

---

## 4. KẾT LUẬN & ĐÁNH GIÁ KỸ THUẬT

1. **Hiệu quả phát hiện lỗi của BVA:** BVA chứng minh sự vượt trội khi nhắm trúng các lỗi tiềm ẩn nguy hiểm nhất của hệ thống EShop, đặc biệt là lỗi so sánh toán tử sai (`total_amount > coupon.min_order_amount`) và lỗi điều kiện phủ định trong State Machine (`order.status === "delivered" || order.status === "canceled"`).
2. **Tính bổ trợ cho Domain Testing:** Khi kết hợp Domain Testing với BVA, độ phủ kiểm thử tăng từ mức độ phân lớp rộng sang kiểm soát chi tiết từng đơn vị ranh giới, đảm bảo hệ thống không có "vùng tối" nghiệp vụ.
