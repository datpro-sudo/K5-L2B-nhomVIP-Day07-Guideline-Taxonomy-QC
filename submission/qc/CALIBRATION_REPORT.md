# Báo cáo calibration — thành viên 102

Theo xác nhận của 102, nhóm đã họp, trao đổi và kiểm chứng theo tiến trình học tập, dựa trên guideline và hiểu biết của nhóm. 102 được giao ghi nhận và tổng hợp kết quả calibration. Hồ sơ không tự bổ sung người tham dự, ngày họp hoặc phê duyệt chưa được cung cấp.

## Kết quả bàn giao

- 40 ảnh, 337 annotation; năm ảnh rỗng được giữ nguyên.
- 1.133 ca IAA có đủ dòng quyết định; bảy dòng CAL-IMG bổ sung phạm vi ảnh, tổng 1.140 dòng.
- Nguồn nhãn thực tế theo từng ảnh ở calibration_selected_sources.csv; ánh xạ annotation ở calibration_provenance.json. Đây là dấu vết của bộ dữ liệu bàn giao, không phải quy tắc gán nhãn theo mã thành viên.
- Giữ nguyên 215 cờ can_review trên 30 ảnh; CSV có 323 dòng đối tượng và JSON có thêm 14 ignore_region.
- Kiểm tra kỹ thuật bằng code: lớp, bbox và toàn bộ payload khớp nguồn đã ghi nhận; ID/ảnh/case_id hợp lệ; nguồn độc lập và IAA giữ nguyên hash. Chi tiết ở calibration_verification.json.

## Pipeline và bảo toàn dữ liệu

Đã dùng prepare_calibration.py, COCO phiên bản do script cấp và build_final.py. Script gốc không bị sửa. Do bước build chỉ xuất các thuộc tính taxonomy và tự tính area, bước bàn giao khôi phục nguyên payload nguồn (gồm occluded, rotation, segmentation, area và thuộc tính của ignore_region) rồi cập nhật hash manifest, có log/dấu vết trước và sau.

Lượt chỉnh tài liệu này thay cách diễn đạt và tên một số file hồ sơ; không thay đổi 337 annotation. Helper tái lập bàn giao nằm ở work/calibration_handoff/rebuild.py. Tên COCO phiên bản hiện hành và lệnh thực tế được ghi trong calibration_provenance.json.

## Phần còn lại

Kiểm tra bằng code không thay cho kiểm tra đúng vật/biên/lớp/thuộc tính trên ảnh. Nhóm đã họp và kiểm chứng theo xác nhận của 102; không quy hoạt động đó thành việc AI đã xem đủ từng ảnh hoặc đã được reviewer độc lập phê duyệt. Người 4 còn phải QC, xử lý cờ review và xác nhận trạng thái phù hợp. Chưa tự tuyên bố Release; không commit/push.
