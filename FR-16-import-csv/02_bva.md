# PHÂN TÍCH GIÁ TRỊ BIÊN (BOUNDARY VALUE ANALYSIS - BVA)
## TÍNH NĂNG: FR-16 — IMPORT SẢN PHẨM TỪ CSV (POOL C)

---

## 1. XÁC ĐỊNH RANH GIỚI VÀ ĐIỂM BIÊN

### 1.1. Biến giá sản phẩm `product_price` ($P \in \mathbb{R}$)
* **Điều kiện:** $P > 0$ ₫ (Biên mở tại 0).
* **Ranh giới:** $B_P = 0$.
* **Điểm biên:**
  * **On-point:** $P = 0$ ₫ (Nằm trên biên $\rightarrow$ Không hợp lệ).
  * **Off-point:** $P = 1$ ₫ (Nhỏ nhất trong miền $\rightarrow$ Hợp lệ).
  * **In-point:** $P = 250,000$ ₫ (Hợp lệ).
  * **Out-point:** $P = -1$ ₫ (Không hợp lệ).

### 1.2. Biến độ dài tên sản phẩm: `product_name` ($L_N \in \mathbb{N}$)
* **Điều kiện:** $1 \le L_N \le 100$ ký tự.
* **Điểm biên:**
  * On min: $L_N = 1$ (Hợp lệ).
  * Off min: $L_N = 0$ (Không hợp lệ).
  * On max: $L_N = 100$ (Hợp lệ).
  * Off max: $L_N = 101$ (Không hợp lệ).

---

### 1.3. Bảng Phân Tích Biên 7 Điểm (Robust BVA)
| Biến kiểm thử | `min - 1` | `min` | `min + 1` | `nominal` | `max - 1` | `max` | `max + 1` |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `product_name` ($L_N$) | $0$ ký tự *(Lỗi)* | $1$ ký tự *(Đạt)* | $2$ ký tự *(Đạt)* | $50$ ký tự *(Đạt)* | $99$ ký tự *(Đạt)* | $100$ ký tự *(Đạt)* | $101$ ký tự *(Lỗi)* |
| `product_price` ($P$ ₫) | $0$ ₫ *(Lỗi)* | $1$ ₫ *(Đạt)* | $2$ ₫ *(Đạt)* | $200,000$ ₫ *(Đạt)* | $99,999,999$ ₫ *(Đạt)* | $100,000,000$ ₫ *(Đạt)* | $100,000,001$ ₫ *(Lỗi)* |


## 2. MA TRẬN TEST CASES BVA CHO FR-16

| Test ID | Biến kiểm thử | Loại điểm biên | Giá trị đầu vào | Kết quả mong đợi |
| :--- | :--- | :---: | :--- | :--- |
| **BVA_P_01** | `price` | **On-point** ($P=0$) | Dòng sản phẩm có $P = 0$ ₫ | Báo lỗi: "Giá sản phẩm phải lớn hơn 0". |
| **BVA_P_02** | `price` | **Off-point** ($P=1$) | Dòng sản phẩm có $P = 1$ ₫ | Chấp nhận import thành công sản phẩm giá 1 ₫. |
| **BVA_P_03** | `price` | **Out-point** ($P=-1$) | Dòng sản phẩm có $P = -1$ ₫ | Báo lỗi và từ chối giá âm. |
| **BVA_N_01** | `name` | **Off min** ($L_N=0$) | Tên sản phẩm để rỗng `""` | Báo lỗi: "Thiếu tên sản phẩm". |
| **BVA_N_02** | `name` | **On min** ($L_N=1$) | Tên sản phẩm 1 ký tự `"A"` | Import thành công. |
| **BVA_N_03** | `name` | **On max** ($L_N=100$) | Tên sản phẩm đúng 100 ký tự | Import thành công. |
| **BVA_N_04** | `name` | **Off max** ($L_N=101$)| Tên sản phẩm 101 ký tự | Báo lỗi vượt quá 100 ký tự. |
