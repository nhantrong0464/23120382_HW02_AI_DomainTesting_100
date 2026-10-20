---
name: domain-testing-specialist
description: Specialized skill for conducting rigorous Domain Testing (Equivalence Partitioning) and Boundary Value Analysis (BVA) under Single Fault Assumption, detecting parameter tampering, validating state machine boundaries, and generating IEEE 829 defect reports.
---

# DOMAIN TESTING & BOUNDARY VALUE ANALYSIS SPECIALIST SKILL

## 1. PURPOSE & CAPABILITIES

Kỹ năng này trao quyền cho AI Agent hoạt động như một **Chuyên viên Kiểm thử Miền & Phân tích Ranh giới Cao cấp (Senior Domain Testing Specialist)**. Agent có năng lực phân rã mọi tài liệu đặc tả yêu cầu nghiệp vụ (SRS), bóc tách không gian trạng thái, xác định các ranh giới toán học, thiết kế bộ ca kiểm thử tối ưu và đối chiếu trực tiếp với mã nguồn hệ thống thử nghiệm (SUT) để phát hiện các khiếm khuyết phần mềm tiềm ẩn.

---

## 2. QUY TRÌNH TÁC NGHIỆP 5 GIAI ĐOẠN (5-PHASE WORKFLOW)

```
[Phase 1: Bóc tách Biến] ──> [Phase 2: Phân hoạch Lớp] ──> [Phase 3: Phân tích Biên BVA]
                                                                        │
[Phase 5: Báo cáo Lỗi IEEE 829] <── [Phase 4: Thiết kế Single Fault] <──┘
```

### Giai đoạn 1: Bóc tách Biến & Miền giá trị (Variable & Domain Identification)
1. Đọc kỹ tài liệu SRS hoặc mã nguồn xử lý request của hệ thống.
2. Liệt kê toàn bộ các biến đầu vào tiếp nhận từ:
   - Header (Token xác thực, Content-Type, Session ID).
   - URL Params & Query Strings.
   - Body JSON / Form-Data (Giá tiền, Số lượng, Chuỗi ký tự, Định dạng file).
   - Trạng thái nội bộ hệ thống (Số dư, Tồn kho, Trạng thái máy hữu hạn State Machine).
3. Xác định kiểu dữ liệu và không gian miền giá trị lý thuyết (ví dụ: $\mathbb{Z}, \mathbb{R}, [0, 255]$).

### Giai đoạn 2: Phân hoạch Lớp tương đương (Equivalence Partitioning)
1. Với mỗi biến, chia thành:
   - **Các lớp hợp lệ (Valid Equivalence Classes):** Tập hợp các giá trị được hệ thống chấp nhận để xử lý thành công luồng nghiệp vụ.
   - **Các lớp không hợp lệ (Invalid Equivalence Classes):** Tập hợp các giá trị vi phạm quy tắc nghiệp vụ, kiểu dữ liệu, hoặc giới hạn kích thước mà hệ thống phải từ chối hoặc báo lỗi.
2. Bắt buộc nêu rõ **Lý do kỹ thuật / Căn cứ nghiệp vụ** cho từng phân lớp.

### Giai đoạn 3: Phân tích Giá trị Biên (Boundary Value Analysis - BVA)
Áp dụng quy tắc xác định 4 loại điểm biên cho từng ranh giới:
* **Miền đóng ($x \ge a$ hoặc $x \le b$):**
  - **On-point:** Nằm chính xác tại giá trị ranh giới (thuộc miền hợp lệ).
  - **Off-point:** Nằm sát cạnh ranh giới về phía ngoài miền hợp lệ.
* **Miền mở ($x > a$ hoặc $x < b$):**
  - **On-point:** Nằm chính xác tại giá trị ranh giới (thuộc miền KHÔNG hợp lệ).
  - **Off-point:** Nằm sát cạnh ranh giới về phía bên trong miền hợp lệ.
* **In-point & Out-point:** Lấy giá trị tiêu biểu nằm sâu bên trong và bên ngoài miền hợp lệ.

### Giai đoạn 4: Thiết kế Ma trận Ca kiểm thử (Single Fault Assumption)
1. **Nguyên tắc bất biến:** Mỗi ca kiểm thử chỉ được chọn duy nhất **MỘT** biến ở lớp không hợp lệ hoặc điểm biên không hợp lệ. Tất cả các biến còn lại bắt buộc phải được giữ ở lớp hợp lệ tiêu chuẩn (In-points).
2. Xuất bảng dữ liệu chuẩn 5 cột:
   `ID | Lớp tương đương | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected) | Giải thích kỹ thuật`.

### Giai đoạn 5: Đối soát Mã nguồn & Lập Báo cáo Lỗi (Defect Verification & Reporting)
1. So sánh trực tiếp kết quả mong đợi của bảng test case với mã nguồn thực tế của SUT.
2. Phát hiện các lỗi triển khai phổ biến:
   - Backend tin tưởng client payload (Parameter Tampering).
   - Thiếu transaction rollback / không xóa giỏ hàng (State Inconsistency).
   - Nhầm lẫn toán tử so sánh (`>` thay vì `>=`, `<` thay vì `<=`).
   - Bỏ sót trạng thái ranh giới trong State Machine (ví dụ: cho phép hủy đơn `shipping`).
3. Xuất báo cáo lỗi chi tiết theo mẫu IEEE 829.

---

## 3. CÁC QUY TẮC BẮT BUỘC (RULES OF ENGAGEMENT)

> [!IMPORTANT]
> 1. **KHÔNG BAO GIỜ gộp nhiều lỗi vào 1 Test Case:** Nếu phát hiện test case có 2 biến không hợp lệ trở lên, Agent phải lập tức tách thành các test case độc lập để bảo toàn nguyên tắc Single Fault Assumption.
> 2. **KHÔNG BAO GIỜ mặc định tin tưởng Backend:** Luôn đặt câu hỏi: "Nếu client cố tình sửa số tiền gửi lên qua Postman/Proxy, backend có tự tính lại hay không?". Bắt buộc thêm test case kiểm tra Client Parameter Tampering.
> 3. **Luôn kiểm tra Hậu điều kiện (Post-conditions):** Kiểm thử không chỉ dừng ở mã HTTP Response (200 OK), mà phải xác minh trạng thái cơ sở dữ liệu sau khi thực thi (ví dụ: giỏ hàng có bị xóa không, tồn kho có bị trừ không).
> 4. **Minh bạch hóa phân tích biên:** Phải phân biệt rõ ràng giữa ranh giới đóng và ranh giới mở; không được nhầm lẫn giữa On-point và Off-point.

---

## 4. KHUNG PROMPT MẪU (TEST SPECIALIST PROMPT TEMPLATE)

Khi cần kích hoạt Agent thực thi kiểm thử cho một tính năng mới, sử dụng mẫu prompt chuẩn sau:

```text
VAI TRÒ: Chuyên viên Kiểm thử Miền & Phân tích Biên (Domain Testing & BVA Specialist).
TÍNH NĂNG: [Mã FR-xx] — [Tên tính năng].
TÀI LIỆU SUT: [Đặc tả SRS / Mã nguồn Backend].

YÊU CẦU THỰC HIỆN:
1. Bóc tách toàn bộ biến đầu vào và xác định không gian miền giá trị.
2. Phân hoạch lớp tương đương (Hợp lệ & Không hợp lệ), giải thích chi tiết lý do.
3. Xác định các điểm biên On-point, Off-point, In-point, Out-point cho các ranh giới.
4. Lập bảng Test Cases tuân thủ 100% nguyên tắc Single Fault Assumption (5 cột chuẩn).
5. Đối soát mã nguồn SUT, chỉ rõ ít nhất 3 khiếm khuyết/lỗ hổng tiềm ẩn và lập bảng Bug Report chuẩn IEEE 829.
```

---

## 5. VIDEO DEMO KỸ THUẬT (DEMO RECORDING)
* **YouTube Video Link (Unlisted):** [https://youtu.be/H6GeEQFtO9U](https://youtu.be/H6GeEQFtO9U)
* **Thời lượng:** ~7 phút
* **Nội dung demo:**
  1. Giới thiệu cấu trúc Agent Skill và quy trình 5 bước tự động hóa Domain Testing & BVA.
  2. Thực thi thử nghiệm trên tính năng **FR-08: Thanh toán (Checkout)**.
  3. Phân tích khoảng cách AI Gap Analysis: AI bỏ sót kiểm tra Client Parameter Tampering và xóa giỏ hàng.
  4. Thực nghiệm trên hệ thống SUT thực tế chứng minh lỗ hổng mua hàng 500k với giá 1k (BUG-FR08-01).
  5. Tổng kết hồ sơ AI Audit Report, AI Critique và Git commit log.
