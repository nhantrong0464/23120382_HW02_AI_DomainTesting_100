# BÁO CÁO PHÁT HIỆN LỖI PHẦN MỀM (BUG REPORTS)
## TÍNH NĂNG: PHÂN HỆ MOBILE (FR-20) — THANH TOÁN & HỦY ĐƠN (POOL D)

---

## 1. DANH MỤC LỖI PHÁT HIỆN

| Bug ID | Tóm tắt lỗi | Thành phần | Severity | Priority | Trạng thái |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **BUG-FR20-01** | Lỗi bất đồng bộ bảo mật: Backend cho phép hủy đơn hàng đang giao (`shipping`) dù Mobile UI đã ẩn nút Hủy | Backend (`/api/orders/:id/cancel`) | **High** | **P1** | **Open** |
| **BUG-FR20-02** | Lỗi biên Coupon: Khách hàng mua đúng 300,000 ₫ trên Mobile bị từ chối áp dụng mã `SAVE10` | Backend (`/api/coupons/apply`) | **Medium** | **P2** | **Open** |
| **BUG-FR20-03** | Giỏ hàng không tự động reset trên giao diện Mobile sau khi hoàn tất đặt hàng | Mobile App (`App.js: line 235`) | **Medium** | **P2** | **Open** |

---

## 2. CHI TIẾT CÁC BÁO CÁO LỖI

### BUG-FR20-01: Backend cho phép hủy đơn hàng đang giao (`shipping`)
* **Mô tả:**
  Trên giao diện React Native (`App.js: line 440`), lập trình viên frontend ẩn nút Hủy khi đơn ở trạng thái `shipping`:
  ```jsx
  {(o.status === "pending" || o.status === "confirmed") && (
    <TouchableOpacity onPress={() => cancelOrder(o.id)}>
      <Text>Hủy đơn</Text>
    </TouchableOpacity>
  )}
  ```
  Tuy nhiên, trong `backend/server.js`:
  ```javascript
  if (order.status === "delivered" || order.status === "canceled") {
    return res.status(400).json({ error: "Cannot cancel this order." });
  }
  ```
  Backend không hề kiểm tra `order.status === 'shipping'`. Do đó, người dùng am hiểu công nghệ chỉ cần gọi thẳng API `PUT /api/orders/:id/cancel` qua Postman/cURL là có thể hủy thành công đơn hàng đã rời kho, gây thiệt hại nghiêm trọng về chi phí vận hành logistics!
* **Kỳ vọng:** Backend phải bổ sung điều kiện chặn:
  ```javascript
  if (order.status !== 'pending' && order.status !== 'confirmed') {
    return res.status(400).json({ error: "Chỉ được phép hủy đơn hàng ở trạng thái Chờ xác nhận hoặc Đã xác nhận." });
  }
  ```

### BUG-FR20-02: Từ chối mã giảm giá khi tổng tiền bằng đúng 300,000 ₫
* **Mô tả:** Người dùng trên Mobile chọn mua đơn hàng đúng 300,000 ₫ và nhập mã `SAVE10`. Giao diện hiển thị Toast đỏ: "Đơn hàng tối thiểu phải từ 300,000 ₫".
* **Nguyên nhân:** Backend dùng `total_amount > coupon.min_order_amount` (lớn hơn nghiêm ngặt) thay vì `>=`.
