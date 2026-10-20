# PHÂN TÍCH GIÁ TRỊ BIÊN (BOUNDARY VALUE ANALYSIS - BVA)
## TÍNH NĂNG: PHÂN HỆ MOBILE (FR-20) — THANH TOÁN, COUPON & STATE MACHINE (POOL D)

---

## 1. XÁC ĐỊNH RANH GIỚI VÀ ĐIỂM BIÊN

### 1.1. Biến ngưỡng đơn hàng Coupon: `min_order_amount` ($T$)
* **Điều kiện:** $T \ge 300,000$ ₫ (Miền đóng tại 300k).
* **Điểm biên:**
  * **On-point:** $T = 300,000$ ₫ (Hợp lệ - Đủ điều kiện giảm giá).
  * **Off-point:** $T = 299,999$ ₫ (Không hợp lệ - Thiếu 1 đồng, hệ thống phải từ chối).
  * **In-point:** $T = 500,000$ ₫ (Hợp lệ).

### 1.2. Biến số lượt dùng mã: `coupon_usage` ($U$)
* **Điều kiện:** Giới hạn 1 lần dùng ($U < 1$).
* **Điểm biên:**
  * **Off-point:** $U = 0$ (Chưa từng dùng $\rightarrow$ Hợp lệ).
  * **On-point:** $U = 1$ (Đã dùng 1 lần $\rightarrow$ Chạm ngưỡng cấm $\rightarrow$ Không hợp lệ).

### 1.3. Ranh giới Máy trạng thái Hủy đơn (State Machine Boundary)
* **Biên trong (Cho phép Mobile hủy):** `pending` và `confirmed`.
* **Biên chặn (Cấm Mobile hủy):** `shipping`.
* **Biên kết thúc (Đóng vĩnh viễn):** `delivered` và `canceled`.

---

### 1.4. Bảng Phân Tích Biên 7 Điểm (Robust BVA)
| Biến kiểm thử | `min - 1` | `min` | `min + 1` | `nominal` | `max - 1` | `max` | `max + 1` |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `order_total` ($T$ ₫) | $299,998$ ₫ *(Lỗi)* | $299,999$ ₫ *(Lỗi)* | $300,000$ ₫ *(Đạt)* | $500,000$ ₫ *(Đạt)* | $99,999,999$ ₫ *(Đạt)* | $100,000,000$ ₫ *(Đạt)* | $100,000,001$ ₫ *(Lỗi)* |
| `coupon_usage` ($U$) | $-1$ *(Lỗi)* | $0$ *(Đạt)* | $1$ *(Khóa)* | $2$ *(Khóa)* | $4$ *(Khóa)* | $5$ *(Khóa)* | $6$ *(Khóa)* |


## 2. MA TRẬN TEST CASES BVA CHO MOBILE

| Test ID | Biến kiểm thử | Loại điểm biên | Giá trị đầu vào | Kết quả mong đợi |
| :--- | :--- | :---: | :--- | :--- |
| **BVA_MOB_CP01** | `min_order_amount` | **On-point** ($T=300k$) | $T = 300,000$ ₫, mã `SAVE10` | Áp dụng mã thành công, giảm 30k, còn 270,000 ₫. |
| **BVA_MOB_CP02** | `min_order_amount` | **Off-point** ($T=299,999k$) | $T = 299,999$ ₫, mã `SAVE10` | Báo Toast lỗi: "Đơn hàng tối thiểu phải từ 300,000 ₫". |
| **BVA_MOB_US01** | `coupon_usage` | **Off-point** ($U=0$) | User chưa từng dùng ($U=0$) | Áp dụng mã thành công. |
| **BVA_MOB_US02** | `coupon_usage` | **On-point** ($U=1$) | User đã dùng 1 lần ($U=1$) | Từ chối mã, báo lỗi hết lượt cho phép. |
| **BVA_MOB_ST01** | State Machine | **Biên trong** (`confirmed`) | Đơn hàng trạng thái `confirmed` | Nút "Hủy đơn" hiển thị, bấm hủy chuyển sang `canceled`. |
| **BVA_MOB_ST02** | State Machine | **Biên chặn** (`shipping`) | Đơn hàng trạng thái `shipping` | **Nút "Hủy đơn" BỊ ẨN trên Mobile UI**; API chặn trả về 400. |
