# BÁO CÁO PHÁT HIỆN LỖI PHẦN MỀM (BUG REPORTS)
## TÍNH NĂNG: FR-02 — ĐĂNG NHẬP & KHÓA TÀI KHOẢN (POOL A)

---

## 1. DANH MỤC LỖI PHÁT HIỆN

| Bug ID | Tóm tắt lỗi | Thành phần | Severity | Priority | Trạng thái |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **BUG-FR02-01** | Bộ đếm số lần đăng nhập sai tăng 2 đơn vị mỗi lần thay vì 1 đơn vị | Backend (`/api/login`) | **High** | **P1** | **Open** |
| **BUG-FR02-02** | Thời gian khóa tài khoản bị thiết lập sai thành 180 giây (3 phút) thay vì 30 giây | Backend (`/api/login`) | **Medium** | **P2** | **Open** |
| **BUG-FR02-03** | Mật khẩu được so sánh dạng Plaintext trong CSDL thay vì băm Bcrypt an toàn | Backend (`server.js: line 43`) | **Critical** | **P1** | **Open** |

---

## 2. CHI TIẾT CÁC BÁO CÁO LỖI

### BUG-FR02-01: Bộ đếm số lần sai tăng 2 đơn vị mỗi lần
* **Mô tả:** Trong file `backend/server.js` (dòng 52), mã nguồn viết:
  ```javascript
  const newAttempts = user.login_attempts + 2; // BUG: Lẽ ra phải là + 1
  ```
  Hệ quả: Người dùng chỉ cần nhập sai 2 lần là tài khoản đã bị khóa ngay lập tức (lần 1 tăng lên 2, lần 2 tăng lên 4 $\ge 3$ $\rightarrow$ Khóa), vi phạm đặc tả SRS quy định phải sau 3 lần sai.
* **Kỳ vọng:** `user.login_attempts + 1`.

### BUG-FR02-02: Khóa tài khoản 180 giây thay vì 30 giây
* **Mô tả:** Trong `backend/server.js` (dòng 55):
  ```javascript
  lockedUntil = new Date(Date.now() + 180000).toISOString(); // 180,000ms = 3 phút
  ```
  Hệ quả: Thời gian khóa bị kéo dài quá mức quy định trong môi trường demo kiểm thử (SRS FR-02 yêu cầu 30 giây = 30,000ms).
