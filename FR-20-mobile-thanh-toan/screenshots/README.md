# THƯ MỤC MINH CHỨNG KIỂM THỬ: FR-20 MOBILE (POOL D)

* `BUG_FR20_01_cancel_shipping_api_bypass.png`: Postman gửi request hủy đơn hàng đang ở trạng thái `shipping`, backend trả về HTTP 200 "Hủy đơn thành công" dù nút Hủy trên Mobile đã bị ẩn.
* `BUG_FR20_02_mobile_toast_coupon_error.png`: Ảnh chụp màn hình ứng dụng React Native hiển thị Toast đỏ từ chối mã giảm giá tại ngưỡng 300,000 ₫.
