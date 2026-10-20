# THƯ MỤC MINH CHỨNG KIỂM THỬ: FR-16 (POOL C)

* `BUG_FR16_01_user_bypass_admin_import.png`: Postman gửi request Import với Bearer token của User thường, trả về 200 OK thay vì 403 Forbidden.
* `BUG_FR16_02_partial_insert_no_rollback.png`: Bảng `products` hiển thị các bản ghi bị insert dở dang dù file CSV có dòng lỗi.
