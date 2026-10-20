# PHÂN TÍCH GIÁ TRỊ BIÊN (BOUNDARY VALUE ANALYSIS - BVA)
## TÍNH NĂNG: FR-02 — ĐĂNG NHẬP & KHÓA TÀI KHOẢN (POOL A)

---

## 1. XÁC ĐỊNH RANH GIỚI VÀ ĐIỂM BIÊN CHO CÁC BIẾN

### 1.1. Biến số lần đăng nhập sai: `failed_attempts` ($F \in \mathbb{N}$)
* **Điều kiện khóa tài khoản:** $F \ge 3$ (Miền đóng tại 3).
* **Ranh giới:** $B_F = 3$.
* **Điểm biên:**
  * **On-point:** $F = 3$ (Tài khoản bị khóa chính xác tại lần thử sai thứ 3).
  * **Off-point:** $F = 2$ (Lần thử sai thứ 2, tài khoản vẫn chưa bị khóa, còn 1 cơ hội cuối).
  * **In-point:** $F = 5$ (Số lần sai vượt quá ngưỡng khóa).
  * **Out-point:** $F = -1$ (Giá trị âm phi lý).

### 1.2. Biến thời gian khóa tài khoản: `lockout_duration` ($T \in \mathbb{N}$ giây)
* **Quy định:** Tài khoản tạm khóa trong $30$ giây ($0 \le T \le 30$).
* **Ranh giới:** $B_T = 30$ giây.
* **Điểm biên:**
  * **On-point:** $T = 30$ giây (Thời điểm kết thúc trạng thái khóa).
  * **Off-point:** $T = 31$ giây (Ngay sau khi hết thời gian khóa 1 giây, tài khoản đã mở khóa hoàn toàn).
  * **In-point:** $T = 15$ giây (Nằm giữa khoảng thời gian bị khóa).

---

### 1.3. Bảng Phân Tích Biên 7 Điểm (Robust BVA)
| Biến kiểm thử | `min - 1` | `min` | `min + 1` | `nominal` | `max - 1` | `max` | `max + 1` |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `failed_attempts` ($F$) | $-1$ *(Lỗi)* | $0$ *(Đạt)* | $1$ *(Đạt)* | $2$ *(Đạt)* | $2$ *(Đạt)* | $3$ *(Khóa)* | $4$ *(Khóa)* |
| `lockout_duration` ($T$) | $-1$s *(Lỗi)* | $0$s *(Khóa)* | $1$s *(Khóa)* | $15$s *(Khóa)* | $29$s *(Khóa)* | $30$s *(Khóa)* | $31$s *(Mở)* |


## 2. MA TRẬN TEST CASES BVA CHO FR-02

| Test ID | Biến kiểm thử | Loại điểm biên | Giá trị đầu vào | Kết quả mong đợi |
| :--- | :--- | :---: | :--- | :--- |
| **BVA_F_01** | `failed_attempts` | **Off-point** ($F=2$) | Nhập sai lần 2 ($F = 2$) | Báo lỗi 401: "Invalid email or password", tài khoản **CHƯA BỊ KHÓA**. |
| **BVA_F_02** | `failed_attempts` | **On-point** ($F=3$) | Nhập sai lần 3 ($F = 3$) | Báo lỗi 403 Forbidden: "Tài khoản bị tạm khóa trong 30 giây". |
| **BVA_T_01** | `lockout_duration`| **In-point** ($T=15s$) | Thử đăng nhập lúc $T=15s$ | Từ chối với HTTP 403 Forbidden, không cho phép đăng nhập. |
| **BVA_T_02** | `lockout_duration`| **On-point** ($T=30s$) | Thử đăng nhập lúc $T=30s$ | Vẫn từ chối với HTTP 403 (hoặc chuyển tiếp mở khóa an toàn). |
| **BVA_T_03** | `lockout_duration`| **Off-point** ($T=31s$)| Nhập đúng pass lúc $T=31s$ | **Đăng nhập THÀNH CÔNG** (HTTP 200 OK), reset $F = 0$. |
