# Báo cáo QC — nhóm claude — lô L23-01 (40 ảnh)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Huấn luyện mô hình phát hiện người và phương tiện 2D trên ảnh đường phố. Lô này được tuyển chọn để luyện ca biên (tượng, ma-nơ-canh, xe đồ chơi, xe chở xe, phản chiếu), nên **không đại diện cho phân bố dữ liệu vận hành** và không dùng để đo độ chính xác nghiệm thu của mô hình.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh Wikimedia Commons, giấy phép CC BY, CC BY-SA, CC0 hoặc public domain, ghi công từng ảnh trong `data/ATTRIBUTION.csv`. Chia sẻ lại phải giữ ghi công; ảnh CC BY-SA yêu cầu chia sẻ tương tự.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `labels_calibrated.json` SHA-256 `661d181a35aa97304a2958e87899c7a514f57d710eeb354140c293167b6b5479`, `labels_calibrated.csv` SHA-256 `87e814983c9ef09a5f0dfad4d1ce2f50c936e9be709f3154e64d3494da7d855d`.
- Guideline và taxonomy khi gán / khi bàn giao: gán theo **guideline v1.0** đã khóa với **taxonomy v1.0**; bàn giao theo **guideline v1.1** (thay đổi loại `semantic`) với **cùng taxonomy v1.0** — v1.1 chỉ sửa ba quy tắc chọn vật, không thêm bớt lớp hay thuộc tính nào.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120. Nhóm 5 người nên mỗi ảnh được **3 người gán độc lập**; mỗi người nhận đúng 24 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1066 cặp box ghép được trên tổng 10 cặp người gán; IoU trung bình **1.000**; cùng lớp **100%** (1066/1066). Cả mười cặp người gán đều ĐẠT cả hai ngưỡng 0.85 và 95%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: **44 box chưa ghép** (36 của người đứng trước trong cặp, 8 của người sau); 0 box bị loại do nằm trong vùng ignore của cả hai; **36 ca bất đồng vùng `ignore_region`**, ở `img_021` và `img_030`.
- Hai chỉ số IAA đạt nhưng lô vẫn còn bất đồng: đây là kết quả đáng chú ý nhất của Pha 4, và nhóm 5 người làm nó rõ hơn nhóm nhỏ. Mười cặp người gán, không cặp nào trượt ngưỡng nào, mà lô vẫn có 77 ca cần xem trên 8 ảnh. Lý do: IoU trung bình và tỉ lệ cùng lớp **chỉ tính trên cặp box ghép được**, nên chỗ một người vẽ còn người kia không vẽ gì thì không tạo ra cặp nào và không hạ chỉ số nào. Phân loại 77 ca: `bat_dong_ignore` 36, `co_can_review` 19, `bo_sot_thua` 16, `iou_thap` 2, `sai_thuoc_tinh` 2, `can_review_anh` 2.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_007` — cả ba người gán ảnh đó (209, 345, 777) đều gọi chiếc Toyota Hilux trắng là `car`. Phóng to 3 lần thấy rõ thành thùng hở và giá nóc sau cabin, nên theo định nghĩa `truck` (thùng tách khỏi cabin) đó là `truck`. IAA không phát hiện được vì **cả ba cùng sai giống nhau**: ba box khớp nhau, cùng lớp, nên các cặp này được tính là đồng thuận hoàn hảo. Ca thứ hai cùng kiểu: `img_040`, cả ba người gán đều gán `car` cho chiếc tuk-tuk biển ใล-5725. Chỉ có mắt người đối chiếu guideline mới bắt được hai ca này.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 77/77 ca trong `calibration_candidates.csv` đều có một dòng trong `calibration_decisions.csv`, ghi rõ quyết định, mã quy tắc, người chốt và người kiểm lại. Ba câu hỏi để ngỏ của v1.0 được chốt như sau — **H-A** (xe nằm trên xe khác) bằng R-07 viết lại: phương tiện được chở cao từ 20 px có box riêng, box xe chở không bao nó; **H-B** (cụm dày đặc) bằng R-09 viết lại: `ignore_region` chỉ dùng cho vật dưới 20 px hoặc cụm không tách được biên, và vùng phải vẽ lùi để không chồng box đã gán; **H-C** (vật ba chiều không phải xe hoạt động) bằng R-10 mở rộng: phép thử "có tự di chuyển được trên đường, hoặc có phải người sống không".
- Tác động v1.1 lên nhãn và ảnh: thay đổi `semantic`, 68/77 quyết định có `pham_vi = lo`, nên **cả 40 ảnh được rà lại**, không chỉ 8 ảnh bị cờ. Taxonomy không đổi nên không cần mapping lớp. Thay đổi thấy được trên nhãn cuối: `img_002` và `img_024` giữ box riêng cho xe được chở; `img_033` bỏ box trên ma-nơ-canh; `img_019` giữ box `car` cho xe hóa trang; `img_021` và `img_030` giữ `ignore_region` cho phần không tách được biên. R-16 còn hạ `occluded_level` cho 89 vật từng bị chấm quá cao do nằm lọt trong box bao của một xe lớn hơn.
- Tham chiếu bằng chứng khi Rework: vòng 1 giữ nguyên trong `submission/qc/reviews/round-01/` kèm bản chụp nhãn `labels_snapshot.json` và `review_manifest.json`. Vòng 2 sẽ mở sau khi sửa, không ghi đè vòng 1.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 123, 209, 234, 345, 777 — mỗi ảnh do một người **không** gán ảnh đó duyệt (nhóm 5 người nên mỗi ảnh luôn còn 2 người chưa gán), ngày 16/11/2026.
- Đã duyệt / cả lô: 40/40 ảnh, duyệt toàn bộ lô, không lấy mẫu.
- Ảnh lỗi / ảnh đã duyệt: 9/40 = 22.5%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 5/362 = 1.4%.
- Vật bỏ sót, ảnh và vùng: 6 ca, ở `img_009`, `img_013`, `img_014`, `img_018`, `img_019`, `img_021`; vùng ghi cụ thể trong `qc_review.csv`.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: nhóm `ca_kho` 4/8 ảnh có lỗi (50.0%), nhóm `con_lai` 5/32 (15.6%). Lỗi lặp rõ nhất là `bo_sot` 6 lần, **tất cả cùng một nguyên nhân**: chỗ nào có cụm vật dưới 20 px thì người gán bỏ qua đúng theo R-09 nhưng quên vẽ `ignore_region` bắt buộc. Đây là lỗi hệ thống của quy trình, không phải lỗi của một người.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: máy chặn được — `validate_labels.py` báo ngay nếu một box dùng lớp ngoài taxonomy, vượt khung ảnh hoặc thiếu thuộc tính, và `qc_compare.py` dừng hẳn khi chưa đủ 120/120 lượt gán. Máy **không** chặn được — `img_007` gán `car` cho xe bán tải: box ôm sát, lớp hợp lệ, thuộc tính đủ, ba người gán giống hệt nhau, mọi script đều xanh; chỉ người mở ảnh gốc và đọc lại định nghĩa `truck` mới thấy sai.

## 5. Quyết định bàn giao

- Quyết định: Rework
- Lý do, người nhận và phạm vi sử dụng: tỉ lệ ảnh có lỗi 22.5% vượt xa ngưỡng 2%, và có lỗi lặp cùng một kiểu (6 ca thiếu `ignore_region`). Lô **chưa bàn giao** cho bên huấn luyện mô hình; bản nhãn hiện tại chỉ dùng nội bộ nhóm làm đầu vào cho vòng sửa.
- Nếu còn việc: (a) thêm `ignore_region` cho 5 vùng đã ghi ở `img_009`, `img_013`, `img_018`, `img_019`, `img_021` — 209 xử lý; (b) sửa lớp `img_007` id 77 thành `truck` và `img_040` id 364 thành `three_wheeler`, đồng thời thu box 364 cho khỏi ôm chiếc SUV phía sau — 234 xử lý; (c) vẽ lại ba box tay ở `img_030` (id 253, 254, 255) cho ôm sát — 345 xử lý; (d) gán hai xe máy nhòe ở `img_014` kèm `can_review` — 123 xử lý. Người gộp và chạy lại chuỗi bàn giao: 777, và **chỉ một người chạy**, vì tên tệp COCO sau sửa sinh từ SHA-256 của bảng quyết định. Sau đó mở vòng QC 2; bằng chứng cần kiểm lại là `labels_snapshot.json` của vòng 1 so với bản mới.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: thử với ca C046 — dòng `C046` trong `calibration_decisions.csv` trỏ tới `img_024.jpg`, quy tắc `R-07`, phiên bản `v1.1`, người chốt 209; mở mục 2 của `guideline_v1.1_calibrated.md` đọc được quy tắc xe được chở có box riêng; mở `data/images/img_024.jpg` thấy chiếc ô tô vàng trên nóc chiếc ô tô đen; trong `labels_calibrated.json` ảnh đó có box `car` riêng cho chiếc vàng. Bốn mắt xích khớp nhau.
