# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 232 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 232 | 0.804 | 97.4% | 56.5% | 97.8% | 187/194 | 0 | CHƯA ĐẠT |
| 101|102 | 6 | 20 | 0.798 | 100.0% | 70.0% | 100.0% | 5/15 | 0 | CHƯA ĐẠT |
| 101|103 | 9 | 44 | 0.748 | 95.5% | 59.1% | 95.5% | 22/125 | 0 | CHƯA ĐẠT |
| 101|104 | 5 | 29 | 0.784 | 93.1% | 27.6% | 96.6% | 17/10 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 64 | 0.952 | 100.0% | 98.4% | 100.0% | 1/1 | 0 | ĐẠT |
| 102|104 | 9 | 51 | 0.722 | 98.0% | 27.5% | 98.0% | 68/18 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 24 | 0.711 | 95.8% | 25.0% | 95.8% | 74/25 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1133 ({'bo_sot_thua': 344, 'co_can_review': 470, 'iou_thap': 126, 'sai_thuoc_tinh': 103, 'bat_dong_ignore': 84, 'sai_lop': 6}), trên 33 ảnh cần calibration.
