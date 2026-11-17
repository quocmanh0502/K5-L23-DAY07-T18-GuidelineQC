# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 261 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 261 | 0.797 | 97.3% | 47.9% | 100.0% | 71/152 | 5 | CHƯA ĐẠT |
| 101|102 | 40 | 261 | 0.797 | 97.3% | 47.9% | 100.0% | 71/152 | 5 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 595 ({'co_can_review': 65, 'iou_thap': 137, 'sai_thuoc_tinh': 136, 'bat_dong_ignore': 53, 'bo_sot_thua': 197, 'sai_lop': 7}), trên 33 ảnh cần calibration.
