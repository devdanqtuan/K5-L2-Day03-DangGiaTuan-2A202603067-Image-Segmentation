# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202603067
- Họ và tên: Đặng Gia Tuấn
- Ngày / CVAT local: 11/11/2026 / CVAT Local (localhost:8080)
- Công cụ đã dùng: CVAT Local, Brush, Polygon

---

## 1. Bài đã nộp

Toàn bộ 9 task (14 ảnh) đã được hoàn thành, kiểm tra chất lượng (QA) và xuất khẩu đúng định dạng chuẩn theo quy định của bài lab Day 03.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| :--- | :--- | :---: | :---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | **14 / 14** | **100** |

---

## 2. Một quyết định trước khi dùng gợi ý

- **Ảnh, vị trí và object Medium đầu tiên tự vẽ:** Ảnh `000000181542.jpg`, chiếc xe ô tô con (`car`) màu bạc đỗ ở góc dưới bên trái của khung hình.
- **Class và quy tắc tôi dùng để chọn biên:** Gán nhãn `car`. Quy tắc chọn biên: phóng to 200%, vẽ bám sát mép thân xe, mép lốp xe tiếp giáp với mặt đường; không vẽ lấn sang bóng đổ (shadow) của gầm xe trên mặt đường và không cắt xén gương chiếu hậu của xe.
- **Nếu dùng gợi ý sau đó (SAM):** Vùng gợi ý của mô hình bắt viền thân xe khá tốt nhưng ở phần lốp xe tiếp giáp bóng đường bị lem nhẹ khoảng 2-3 pixel ra nền nhựa; tôi đã dùng công cụ chỉnh sửa đỉnh polygon để kéo lùi biên về sát mép lốp cao su của bánh xe.
- **Lý do:** Đảm bảo ranh giới mask khít với vật thể thực tế nhìn thấy, tránh làm giảm IoU do ăn vào nền đường.

---

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** Task `cp2_slice`, ảnh `000000017627.jpg`, khu vực hai chiếc xe ô tô đỗ sát rạt song song nhau.
- **Lỗi thuộc loại:** Gộp/tách nhầm (2 vật thể bị gộp dính làm một).
- **Bằng chứng tôi nhìn thấy:** Thuật toán AI nhận diện một bounding box lớn bao trùm cả hai xe và sinh ra một mask đa giác dính liền cả 2 thân xe thành 1 object duy nhất.
- **Quy tắc và hành động sửa:** Theo quy tắc của `cp2_slice`, hai vật thể cùng class đứng sát nhau bắt buộc phải là **2 instance/object riêng biệt**. Tôi đã xóa mask gộp, tự vẽ chia tách đường ranh giới khe hẹp giữa 2 xe, tạo thành 2 object `car` riêng biệt trên danh sách Objects của CVAT.
- **Sau sửa đã Save và export lại chưa?:** Đã bấm Save (Ctrl + S) trên CVAT và xuất khẩu lại file `cp2_slice.zip` vào thư mục `submissions/`.
- **Kết quả kiểm tra cấu trúc:** Chạy `scripts/inspect_submissions.py` trả về `[OK] cp2_slice: cp2_slice.zip (annotations: 9 polygon)`.

---

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| STT | Ảnh / Vị trí | Hai cách hiểu có thể | Quy tắc / Chứng cứ đối chiếu | Quyết định hoặc câu hỏi cho coach |
| :---: | :--- | :--- | :--- | :--- |
| **1** | Task `cp4_curb`, ảnh `7d83710e-4697c3b2.jpg`, đoạn mép đường và vỉa hè có màu nhựa đường gần như tương đồng. | 1. Tô gộp toàn bộ thành `road` vì màu sắc giống hệt nhau.<br>2. Tách riêng `road` và `sidewalk`. | Quy tắc trong `guideline-mini-sheet.md`: Ranh giới road–sidewalk phân định theo **chức năng và gờ bó vỉa (curb)**, không chọn theo màu sắc ảnh. | **Quyết định:** Dựa theo mép vát của gờ bó vỉa để kẻ đường phân định rõ ràng giữa làn xe chạy (`road`) và phần hè đi bộ (`sidewalk`). |
| **2** | Task `cp5_occlusion`, ảnh `000000336232.jpg`, chiếc xe ô tô bị cột đèn che cắt ngang chia làm 2 phần nhìn thấy rời nhau. | 1. Tách thành 2 object `car` khác nhau cho 2 nửa.<br>2. Giữ nguyên là 1 instance duy nhất có 2 mảng. | Quy tắc `cp5_occlusion`: Một vật thể bị che khuất ngắt quãng vẫn là **một instance duy nhất**; không được tự ý vẽ xuyên qua cột che. | **Quyết định:** Gán cả 2 mảng polygon rời rạc đó vào cùng 1 object ID của chiếc xe đó, không tạo 2 object độc lập. |
| **3** | Task `cp3_thin`, ảnh `839f7736-abe28069.jpg`, các cột đèn (`pole`) và biển báo (`traffic sign`) có thân rất mảnh. | 1. Dùng brush to vẽ nhanh nhưng sẽ bị lem nhiều sang nền trời (`sky`).<br>2. Phóng to zoom lớn và thu nhỏ kích thước brush. | Quy tắc `cp3_thin`: Nét mảnh cần phóng to và dùng brush nhỏ (2-3px) để tránh làm tăng vùng nhãn sai. | **Quyết định:** Phóng to tối đa, dùng brush kích thước nhỏ vẽ bám sát thân cột kim loại, phần nền trời phía sau giữ nguyên nhãn `sky`. |

