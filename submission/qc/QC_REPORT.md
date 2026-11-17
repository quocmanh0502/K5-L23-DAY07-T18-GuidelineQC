# Báo cáo QC — T18

- Ngày nhóm xác nhận hoàn thành công việc: **16/11/2026** (theo thông tin 102 cung cấp).
- Ngày đồng bộ hồ sơ và chạy kiểm tra bản tổng hợp: 17/11/2026.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn bbox 2D trên 40 ảnh; chưa đánh giá độ đại diện cho vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: `data/ATTRIBUTION.csv`; thông tin nguồn từng ảnh trong manifest.
- Manifest và checksum nhãn cuối: `batch_manifest.json`; `labels_calibrated.json` (40 ảnh/428 annotation), SHA-256 `091becad6c2656ae68b70818cf28c32a985e8d1026248842a81b6e48cee996ba`.
- Guideline và taxonomy khi gán / khi bàn giao: Guideline v1.0 khi gán, v1.1 khi chốt; taxonomy v1.0, định nghĩa và ngưỡng không đổi. Bản tổng hợp được chấp nhận theo xác nhận QC Pass của 102.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt; mỗi thành viên có đủ 40 ảnh, các file độc lập qua kiểm tra cấu trúc.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 261 cặp; IoU TB 0,79735 < 0,85; cùng lớp 254/261 = 97,3% >= 95%. IAA ban đầu chưa đạt, được giữ nguyên sau khi tạo bản tổng hợp.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 71/152 ở hai bộ độc lập, 5 box loại do ignore; 53 ca bất đồng ignore.
- Thuộc tính: Cùng mức che 125/261 = 47,9%; cùng truncated 261/261 = 100% trên cặp đã ghép.
- Phạm vi calibration: 595 ca trên 33 ảnh; phạm vi kiểm tra bản tổng hợp là toàn bộ 40 ảnh.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa có ca tham chiếu coach trong bằng chứng hiện có; chưa ghi kết luận đối chiếu.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Theo xác nhận của 102, nhóm đã đối chiếu ảnh, review chéo và chốt từng ca theo guideline. Kết quả chốt trùng khớp với bản tham khảo đã chuẩn bị. Nguồn từng ảnh ở mixed_draft_sources.csv; xác nhận QC ở review_confirmation.json và phần làm rõ quy trình ở review_clarification.json.
- Tác động v1.1 lên nhãn và ảnh: Không đổi nghĩa quy tắc. Nhãn cuối có 428 annotation: 400 đối tượng và 28 vùng ignore, đủ 40 ảnh. Không còn cờ can_review; kết quả QC lấy từ xác nhận của 102, không suy ra chỉ từ việc tắt cờ.
- Tham chiếu bằng chứng: labels_calibrated.json, labels_calibrated.csv, batch_manifest.json, bảng nguồn từng ảnh, nhật ký tạo bản tổng hợp và snapshot vòng 3. Hai bộ nhãn độc lập giữ nguyên.

## 4. Kết quả duyệt

- Vòng QC: 3
- Người review và ngày: 102 xác nhận nhóm đã review chéo, chốt từng ca và QC Pass cho toàn bộ 40 ảnh/428 annotation. Mốc nhóm hoàn thành là 16/11/2026; thông tin được ghi nhận, đồng bộ hồ sơ ngày 17/11/2026. Đây là kết quả nhóm xác nhận, không phải lượt xem ảnh mới của trợ lý.
- Đã duyệt / cả lô: 40/40 theo phạm vi xác nhận QC Pass của 102.
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0% theo phiếu ghi nhận xác nhận của 102.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/400 = 0% theo phiếu; 28 vùng ignore không thuộc mẫu số đối tượng.
- Vật bỏ sót, ảnh và vùng: 0 ca được báo trong xác nhận QC Pass.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: 33 ảnh ca khó và 7 ảnh còn lại được bao phủ theo xác nhận; không còn cờ review. Vòng 1 và vòng 2 giữ làm lịch sử; số vòng là phiên bản hồ sơ, không khẳng định ba lượt review độc lập.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script kiểm schema, tên ảnh, phạm vi và checksum; đúng vật, lớp, ranh giới và bỏ sót cần đọc ảnh.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: QC vòng 3 trả RELEASE từ phiếu ghi nhận xác nhận QC Pass của 102. Dùng cho bài thực hành và bàn giao riêng tư cho coach; không suy rộng sang dữ liệu vận hành.
- Nếu còn việc: Các file bàn giao đã đồng bộ để chốt bài. Mã commit cuối được ghi tại kênh nộp; không đưa work/ hay thông tin đăng nhập vào gói nộp.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: C005 → img_001.jpg → mục 2.2 guideline → nguồn trong mixed_draft_sources.csv → annotation trong labels_calibrated.json → quyết định calibration và snapshot vòng 3. IAA trước calibration giữ nguyên.

## 6. Công cụ và nguồn bằng chứng

Bản nhãn được công cụ chuẩn bị dùng làm tài liệu tham khảo. Theo xác nhận của 102, nhóm đã đối chiếu ảnh và guideline, review chéo và chốt từng ca; kết quả chốt trùng khớp với bản tham khảo. Nhãn cuối được chấp nhận theo kết quả review của nhóm. Nhật ký vẫn giữ cách tạo bản tham khảo, nguồn dữ liệu và checksum để đối chiếu.

AI hỗ trợ chuẩn bị tài liệu tham khảo và hồ sơ; quyết định nhãn cuối do nhóm chốt sau review. Trợ lý chạy các kiểm tra kỹ thuật và ghi nhận kết quả nhóm xác nhận. Các sao lưu cục bộ ở work/ không đưa vào gói nộp.
