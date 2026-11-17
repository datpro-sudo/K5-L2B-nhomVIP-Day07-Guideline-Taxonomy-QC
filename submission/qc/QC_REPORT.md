# Báo cáo QC — NhomVIP — Nguyễn Văn Nhất (104)

Ngày: 17/11/2026. Người phụ trách: 104. 



## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Kiểm nhãn bounding box của 40 ảnh lab Day 07; chưa Release để bàn giao sử dụng.
- Nguồn và phép sử dụng dữ liệu: data/images và data/ATTRIBUTION.csv; giữ ghi công và giấy phép từng ảnh.
- Manifest và checksum nhãn cuối: submission/qc/batch_manifest.json; SHA-256 labels_calibrated.json: 0d9cdba74901548d72e38a19c17cc3f6db84c5e9f0838ea965bee2649f160807. current_batch() đã kiểm chuỗi bằng chứng thành công sau chuyển mã. Snapshot của vòng 1 nằm trong reviews/round-01.
- Guideline và taxonomy khi gán / khi bàn giao: Guideline v1.0 khi gán, v1.1 khi bàn giao; taxonomy v1.0. Chuyển mã theo member_ids.json, không thay định nghĩa lớp, rules hoặc nhãn.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80. Mỗi thành viên 101, 102, 103, 104 có 20 ảnh; bốn file COCO qua validate_labels.py --all.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 232 cặp ghép, IoU TB 0,804, cùng lớp 97,4%. Kết quả tính lại giữ nguyên sau đổi mã; IoU trước calibration chưa đạt 0,85.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 187/194 ở hai phía các cặp; 0 loại do ignore trong thống kê tổng; 84 ca bất đồng ignore, tổng 1.133 ca trên 33 ảnh.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa có xác nhận tham chiếu coach. Không suy luận hai người cùng sai chỉ từ lỗi ở bản cuối.

## 3. Calibration và chuyển định danh

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Giữ nguyên 1.140 dòng quyết định, gồm 1.133 case IAA và 7 dòng CAL-IMG. Chỉ đổi nguoi_chot theo ánh xạ được người dùng xác nhận; không tạo phê duyệt mới. Câu chữ lịch sử có mã cũ được đọc theo member_ids.json.
- Tác động v1.1 lên nhãn và ảnh: Giữ nguyên 40 ảnh, 337 annotation (323 đối tượng, 14 ignore_region) và toàn bộ payload. Còn 215 cờ can_review trên 30 ảnh; lượt này không tự xóa cờ hoặc sửa nhãn.
- Tham chiếu bằng chứng khi Rework: Các lỗi mới ở reviews/round-01/qc_review.csv. Cần người sửa xác nhận, sửa theo phiên bản rồi mở vòng mới, giữ nguyên vòng 1.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 104 là người phụ trách hồ sơ, 17/11/2026. AI thực hiện rà ảnh; . Phương pháp ở reviews/round-01/REVIEW_METHOD.md.
- Đã duyệt / cả lô: Đã xem overlay 40/40 ảnh; phiếu có kết quả cho 34/40 ảnh, sáu ảnh còn câu hỏi để trống.
- Ảnh lỗi / ảnh đã duyệt: 10/34 = 29,4% ,sáu ảnh chưa có kết luận.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 13/274 = 4,7%; loại ignore_region khỏi mẫu số đối tượng. Đây là số trong phạm vi 34 ảnh có kết quả.
- Vật bỏ sót, ảnh và vùng: 2 phát hiện: người áo tím trong tuk-tuk trái img_016 và người áo trắng quay lưng gần mũ hoa img_021; vùng cụ thể ghi trong phiếu.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó có kết quả 27/33, lỗi 10/27; phần còn lại 7/7, chưa phát hiện lỗi trong 7 ảnh đó. Ghi 7 lỗi vị trí/kích thước, 4 sai lớp, 3 thừa và 2 bỏ sót. Ví dụ img_036 #303 là xe tải mang lớp motorcycle; img_013 #130 là người mang lớp car. 
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script kiểm checksum, biên box và cấu trúc; sai lớp motorcycle trên xe tải vẫn qua schema, cần xem ảnh mới phát hiện.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Chuỗi checksum đã hợp lệ nhưng còn sáu ảnh chưa có kết luận, 215 cờ review và chưa có xác nhận người kiểm. Hồ sơ chuyển nội bộ nhóm để sửa; không Release.
- Nếu còn việc: Người sửa kiểm và sửa các lỗi đã ghi, giải quyết cờ review; người phụ trách làm rõ sáu ảnh 001, 004, 009, 019, 022, 024 theo pending_images.csv. Hai thành viên 101 và 103 bổ sung ghi chú cá nhân thật. Sau sửa, build nhãn theo phiên bản và mở vòng 2 để kiểm lại.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: reviews/round-01/qc_review.csv → img_036.jpg #303 → R-02 → labels_snapshot.json bbox [37.78,36.89,860,650].

## 6. Chuyển mã và kiểm gói nộp

Mã báo cáo: 101 — Nguyễn Tiến Đạt; 102 — Lê Minh Thắng; 103 — Trần Anh Tùng; 104 — Nguyễn Văn Nhất. Bảng đối chiếu MSHV được lưu tại TEAM.md và member_ids.json.

Đã chạy lại split_assign theo MSHV với thứ tự tương thích mã cũ. Kiểm chứng: mỗi ảnh vẫn đúng cặp người được giao trước đây; byte nhãn độc lập, nhãn cuối và CSV không đổi; toàn bộ case_id và số đo IAA được bảo toàn. Chỉ tái tạo bằng chứng cho việc đổi định danh, không phê duyệt nội dung nhãn. Xem migration_mshv/verification.json và commands.json.

Các file calibration_provenance.json, calibration_verification.json, calibration_protected_inputs.json cùng preflight_2A202603278/integrity_audit.json là hồ sơ trước chuyển mã; đường dẫn/hash cũ trong đó mang tính lịch sử, không dùng để xác nhận trạng thái hiện tại. Bản trước chuyển nằm ở work/mshv_migration/before, được gitignore.

Phiếu chính thức nằm ở reviews/round-01/qc_review.csv theo phiên bản repo này.

## 7. Lệnh kiểm tra

```powershell
python scripts/validate_labels.py --all
python scripts/qc_report.py --round 1
python scripts/check_submission.py --round 1
```

