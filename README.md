# BÀI TẬP LỚN HW02: KIỂM THỬ MIỀN (DOMAIN TESTING) & PHÂN TÍCH GIÁ TRỊ BIÊN (BVA) KẾT HỢP TRÍ TUỆ NHÂN TẠO

> **Môn học:** Kiểm thử phần mềm (Software Testing)  
> **Hệ thống thử nghiệm (SUT):** EShop Platform (Web Admin, Frontend Web React, Mobile React Native, Backend Express/SQLite)  
> **Sinh viên thực hiện:** 23120382  
> **Kho lưu trữ GitHub:** [https://github.com/nhantrong0464/23120382_HW02_AI_DomainTesting_100](https://github.com/nhantrong0464/23120382_HW02_AI_DomainTesting_100)  
> **Video Demo Kỹ thuật (YouTube Unlisted):** 📺 [https://youtu.be/H6GeEQFtO9U](https://youtu.be/H6GeEQFtO9U)

---

## 📺 VIDEO DEMO KỸ THUẬT (DEMO RECORDING)
* **Link xem video:** [https://youtu.be/H6GeEQFtO9U](https://youtu.be/H6GeEQFtO9U)
* **Nội dung trình bày:**
  1. Giới thiệu cấu trúc Agent Skill (`agent-skill/SKILL.md`) và quy trình 5 bước tự động hóa Domain Testing & BVA.
  2. Thực thi thử nghiệm trên tính năng trọng điểm **FR-08: Thanh toán (Checkout)**.
  3. Phân tích khoảng cách AI Gap Analysis: Nhận diện điểm mù của AI khi bỏ sót kịch bản can thiệp tham số (*Client Parameter Tampering*).
  4. Thực nghiệm trên hệ thống SUT thực tế chứng minh lỗ hổng mua đơn hàng 500.000 ₫ với giá 1.000 ₫ (`BUG-FR08-01`).
  5. Trình bày hồ sơ kiểm định AI Audit Report, phê bình năng lực AI Critique và lịch sử Git Commit.

---

## 📂 CẤU TRÚC DỰ ÁN THEO 4 POOLS BẮT BUỘC

Dự án được phân bổ đồng đều theo 4 nhóm chức năng (Pools) của đề bài:

```text
23120382_HW02_AI_DomainTesting_100/
│
├── 📁 FR-02-dang-nhap-khoa-tai-khoan/    [POOL A — Authentication, Categories & Products]
│   ├── 01_domain_testing.md              # Ma trận lớp tương đương & kết quả thực thi SUT
│   ├── 02_bva.md                         # Robust BVA 7 giá trị (min-1, min, min+1, nominal, max-1, max, max+1)
│   ├── 03_ai_gap_analysis.md             # Phân tích khoảng cách năng lực AI
│   ├── 04_bug_reports.md                 # Báo cáo lỗi chi tiết chuẩn IEEE 829
│   └── screenshots/README.md             # Chỉ mục ảnh minh chứng lỗi
│
├── 📁 FR-08-thanh-toan/                  [POOL B — Shopping Cart & Checkout Web]
│   ├── 01_domain_testing.md              # Domain Testing Single Fault 6 biến đầu vào
│   ├── 02_bva.md                         # BVA biên mở tổng tiền và độ dài địa chỉ
│   ├── 03_ai_gap_analysis.md             # Điểm mù Parameter Tampering của AI
│   ├── 04_bug_reports.md                 # Lỗi mua hàng 500k giá 1k & không xóa giỏ hàng
│   └── screenshots/README.md
│
├── 📁 FR-16-import-csv/                  [POOL C — Web Admin Management]
│   ├── 01_domain_testing.md              # Domain Testing cấu trúc CSV & phân quyền Admin
│   ├── 02_bva.md                         # Robust BVA cho giá tiền và độ dài tên sản phẩm
│   ├── 03_ai_gap_analysis.md             # Phân tích thiếu sót Atomic Transaction Rollback
│   ├── 04_bug_reports.md                 # Lỗi user thường import được & chèn dữ liệu dở dang
│   └── screenshots/README.md
│
├── 📁 FR-20-mobile-thanh-toan/           [POOL D — Mobile App React Native]
│   ├── 01_domain_testing.md              # Checkout Mobile, Coupon SAVE10 & State Machine Hủy đơn
│   ├── 02_bva.md                         # BVA biên đóng coupon 300k và lượt dùng
│   ├── 03_ai_gap_analysis.md             # Lỗi bất đồng bộ bảo mật giữa Mobile UI và Backend
│   ├── 04_bug_reports.md                 # Lỗi cho phép hủy đơn hàng shipping qua API
│   └── screenshots/README.md
│
├── 📁 agent-skill/
│   └── SKILL.md                          # Agentic Skill định chuẩn tự động hóa Domain Testing & BVA
├── 📄 AI_Audit_Report.md                 # Bảng kiểm định 5 cột (Tool, Time, Prompt, Output, Sửa gì) + Toàn văn AI_LOG
├── 📄 AI_Critique.md                     # Bản phản biện học thuật 270 từ (3 luận điểm bắt buộc)
├── 📄 AI_LOG.md                          # Toàn văn nhật ký tương tác AI cho 4 tính năng
└── 📄 git_commit_log.txt                 # Minh chứng lịch sử 18 Git commits tuần tự từng bước
```

---

## 🔍 TỔNG HỢP CÁC LỖI (DEFECTS) ĐÃ PHÁT HIỆN TRÊN ESHOP SUT

| Mã lỗi | Phân hệ / File mã nguồn | Mô tả lỗi thực tế | Mức độ nghiêm trọng |
| :--- | :--- | :--- | :---: |
| **BUG-FR08-01** | Backend (`/api/checkout`) | **Lỗ hổng Parameter Tampering:** Backend nhận trực tiếp `total_amount` từ client mà không tự tính lại từ CSDL giỏ hàng, cho phép mua đơn hàng 500k với giá 1k. | **Critical (Blocker)** |
| **BUG-FR08-02** | Backend (`/api/checkout`) | **Vi phạm hậu điều kiện:** Bảng giỏ hàng (`cart`) không bị xóa sạch trong CSDL sau khi hoàn tất thanh toán. | **High** |
| **BUG-FR02-01** | Backend (`/api/login`) | **Lỗi logic bộ đếm:** Bộ đếm đăng nhập sai bị cộng nhảy cóc 2 đơn vị (`user.login_attempts + 2`) thay vì 1 đơn vị, khóa tài khoản chỉ sau 2 lần sai. | **High** |
| **BUG-FR02-02** | Backend (`/api/login`) | **Sai thời gian khóa:** Hệ thống thiết lập thời gian khóa 180 giây (3 phút) thay vì 30 giây như SRS. | **Medium** |
| **BUG-FR16-01** | Backend (`/api/admin/import-products`) | **Thiếu phân quyền Role:** Khách hàng thông thường (`role: "user"`) vẫn gọi được API import sản phẩm của Admin. | **Critical** |
| **BUG-FR16-02** | Backend (`server.js`) | **Thiếu Atomic Rollback:** Hệ thống chèn từng dòng đơn lẻ, không rollback toàn bộ khi gặp dòng CSV lỗi. | **High** |
| **BUG-FR20-01** | Backend & Mobile App | **Bất đồng bộ State Machine:** Nút Hủy bị ẩn trên Mobile khi đơn hàng đang giao (`shipping`), nhưng gửi API trực tiếp thì backend vẫn cho hủy thành công. | **High** |
| **BUG-FR20-02** | Backend (`/api/coupons/apply`) | **Lỗi biên toán tử:** Backend dùng `total_amount > min_order_amount` (lớn hơn nghiêm ngặt) khiến đơn hàng đúng 300,000 ₫ bị từ chối coupon `SAVE10`. | **Medium** |

---

## 📋 HỒ SƠ BẮT BUỘC
* **Báo cáo Thẩm định AI:** [AI_Audit_Report.md](file:///d:/Ki%E1%BB%83m%20th%E1%BB%AD%20ph%E1%BA%A7n%20m%E1%BB%81m/23120382_HW02_AI_DomainTesting_100/AI_Audit_Report.md) (Bảng kiểm định 5 cột chi tiết).
* **Bản Phê bình AI:** [AI_Critique.md](file:///d:/Ki%E1%BB%83m%20th%E1%BB%AD%20ph%E1%BA%A7n%20m%E1%BB%81m/23120382_HW02_AI_DomainTesting_100/AI_Critique.md) (Dung lượng 270 từ, nêu đủ 3 luận điểm).
* **Lịch sử Git Commit:** [git_commit_log.txt](file:///d:/Ki%E1%BB%83m%20th%E1%BB%AD%20ph%E1%BA%A7n%20m%E1%BB%81m/23120382_HW02_AI_DomainTesting_100/git_commit_log.txt) (Xuất từ lệnh `git log --graph`).
