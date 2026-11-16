# Thông tin nhóm

- Tên nhóm: claude
- Mã thành viên (ba chữ số): 123, 209, 234, 345, 777
- Người điều phối: 209
- Người review bản cuối: 777
- Commit chốt bài (điền ở kênh nộp sau khi commit):

> Nhóm 5 người nên mỗi ảnh được **3 người gán độc lập** (`split_assign.py` tự đặt `--overlap 3`),
> mỗi người nhận đúng 24 ảnh. Bảng đối chiếu mã với họ tên do coach giữ ở kênh lớp có giới hạn
> quyền, không nằm trong gói này.

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 120/120 (40 ảnh × 3 người gán độc lập)
- IAA trước calibration (kèm mẫu số): IoU TB 1.000 và cùng lớp 100% trên **1066 cặp box ghép được**, cả 10 cặp người gán đều đạt; ngoài mẫu số đó còn **44 box chưa ghép** và 36 ca bất đồng vùng `ignore_region`
- Số ảnh phải calibration: 8 / 40 (77 ca, tất cả đã có quyết định)
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1, duyệt toàn bộ 40 / 40
- Tỉ lệ lỗi: 22,5 % theo ảnh (9/40), 1,4 % theo đối tượng (5/362)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **Rework** — vượt ngưỡng 2 % ảnh lỗi và có lỗi lặp cùng kiểu (6 ảnh thiếu `ignore_region` cho cụm vật dưới 20 px)
