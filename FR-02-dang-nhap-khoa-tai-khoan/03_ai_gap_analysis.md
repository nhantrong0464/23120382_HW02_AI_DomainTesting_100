# PHÂN TÍCH KHOẢNG CÁCH NĂNG LỰC AI (AI GAP ANALYSIS)
## TÍNH NĂNG: FR-02 — ĐĂNG NHẬP & KHÓA TÀI KHOẢN (POOL A)

---

## 1. SO SÁNH KẾT QUẢ AI VÀ TIÊU CHUẨN KIỂM THỬ THỰC TẾ

| Tiêu chí | Kết quả AI sinh ban đầu | Yêu cầu Chuyên gia / Thực tế SUT | Khoảng cách & Đánh giá |
| :--- | :--- | :--- | :--- |
| **Kiểm tra cơ chế tăng bộ đếm** | AI chỉ kiểm tra đăng nhập sai 1 lần (HTTP 401) và sai 3 lần (HTTP 403). | Kiểm tra tính đơn nhất của bước tăng: Mỗi lần sai $F$ chỉ được tăng đúng 1 đơn vị ($F = F + 1$). | **Nghiêm trọng:** Code SUT thực tế bị lỗi `newAttempts = user.login_attempts + 2` (tăng 2 đơn vị mỗi lần). AI không phát hiện ra nếu không đối soát CSDL sau mỗi bước. |
| **Kiểm tra thời gian khóa** | AI giả định sau 30 giây tài khoản sẽ mở, nhưng không đưa ra test case kiểm tra việc thử lại ngay khi đang bị khóa. | Bắt buộc test case thử nhập đúng mật khẩu ở giây thứ 10 để chứng minh tài khoản vẫn bị khóa an toàn. | **Trung bình:** Bỏ qua kiểm tra tính toàn vẹn của trạng thái khóa trong khoảng thời gian hiệu lực. |
| **Thời gian khóa thực tế** | AI tin rằng hệ thống khóa 30 giây như SRS. | Kiểm tra đối soát thời gian khóa thực tế trong code SUT (`180000ms = 180s = 3 phút`). | **Cao:** Code SUT bị cấu hình sai thời gian khóa (gấp 6 lần so với SRS). |
