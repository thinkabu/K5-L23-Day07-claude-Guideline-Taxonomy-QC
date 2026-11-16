# Guideline gán nhãn — bản cá nhân — 123

- Phiên bản: v0.1 (bản cá nhân Pha 1)
- Ngày: 16/11/2026
- Người viết: 123

## 1. Mục tiêu

Nhãn cho mô hình phát hiện 2D dùng trên đường châu Á. Điều tôi chú ý khi xem 40 ảnh: phần lớn
ảnh **không phải** cảnh đường cao tốc sạch sẽ, mà là phố đông, chợ, bãi đỗ, có cả ảnh chụp
trong nhà và ảnh trắng đen. Một guideline chỉ mô tả xe cộ trên đường trống sẽ không dùng được
cho lô này.

Lỗi nguy hiểm nhất là bỏ sót người. Lỗi nguy hiểm thứ hai là gán box cho thứ không tồn tại,
vì nó dạy mô hình phanh trước một cái bóng.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa một câu | Ví dụ | Phản ví dụ, dễ nhầm |
|---|---|---|---|
| `car` | Xe bốn bánh chở người, thân liền khối. | `img_012`, `img_015`, `img_040` taxi | Bán tải thuộc `truck` |
| `van` | Xe thùng kín, cao hơn car, ngắn hơn bus. | `img_005`, `img_016` | `img_009` thuộc `bus` |
| `bus` | Xe khách thân dài **trên 7 m**. | `img_009`, `img_037` | `van` |
| `truck` | Thùng hoặc bệ chở tách khỏi cabin, gồm bán tải. | `img_001`, `img_002`, `img_007`, `img_012` | `van` |
| `tuktuk` | Xe ba bánh **có động cơ**, chở khách hoặc hàng. | `img_004`, `img_016`, `img_035`, `img_040` | `cyclo` |
| `cyclo` | Xe ba bánh **đạp chân**. | `img_023` | `tuktuk`, `bicycle` |
| `motorcycle` | Xe hai bánh có động cơ. | `img_006`, `img_008`, `img_022` mô tô có thùng bên | `bicycle` |
| `bicycle` | Xe đạp hai bánh. | `img_003`, `img_006`, `img_029` | `motorcycle` ở xa |
| `person` | Người thật. | `img_014`, `img_034`, `img_029` | Tượng, ma-nơ-canh, hình in |
| `ignore_region` | Vùng bỏ qua. | Bãi xe xa trong `img_018` | Không có |

**Vì sao tôi tách `tuktuk` và `cyclo`:** một chiếc có động cơ chạy 40 km/h, một chiếc đạp chân
chạy 10 km/h. Với bài toán dự đoán quỹ đạo thì đó là hai vật khác hẳn nhau. Tôi biết mỗi lớp
sẽ rất ít mẫu, và sẽ nghe nhóm nếu có lập luận ngược lại.

**Ranh giới `van` và `bus` theo chiều dài thân trên 7 m.** Nhược điểm tôi tự thấy: không đo
được mét trên ảnh, phải ước lượng theo số bánh và số ô cửa.

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | 0, 1, 2, 3 | 0 không bị che; 1 dưới 50%; 2 từ 50%; 3 quá tối hoặc quá mờ để nói |
| `truncated` | no, yes | Chạm mép ảnh |
| `can_review` | tick | Lưỡng lự |

Ảnh ban đêm `img_013`, `img_031` và ảnh nhòe `img_014` nên để `occluded_level = 3` khi không
đọc nổi thân xe, thay vì đoán mức 0.

## 3. Quy tắc vẽ box

1. Ôm sát phần thấy, không đoán phần khuất, ±2 px.
2. Một vật một box.
3. **Người điều khiển xe: gán box `person` riêng khi nhìn thấy cả đầu và thân.** Chỉ thấy mỗi
   cái mũ bảo hiểm nhô lên sau yên xe thì không gán, vì không đủ hình người để mô hình học.
4. Hàng gắn trên xe nằm trong box xe.
5. Vật cao dưới 20 px: gộp vào `ignore_region`.
6. Không gán vật ảo: phản chiếu, hình in, tượng đài.
7. Xe trong nhà, trong phòng trưng bày: vẫn gán, vì `img_015` và `img_007` chụp trong nhà mà
   xe vẫn là xe.

## 4. Ca khó tôi tìm được trong 40 ảnh

| Ảnh | Tình huống | Quyết định ban đầu của tôi | Lý do |
|---|---|---|---|
| `img_019` | Xe hóa trang hình con chó trong đám diễu hành, có biển số thật | **Không gán** | Hình dáng không giống lớp nào trong danh sách. Gán bừa vào `car` sẽ làm bẩn lớp `car` |
| `img_028` | Bán tải cổ thật, trên thùng chở một **tượng ngựa trắng** | Gán `truck` cho xe, **không** gán con ngựa | Xe thật thì gán; tượng thì không, dù nó ngồi trên một chiếc xe thật |
| `img_037` | Bán tải gắn thùng camper lớn phía sau | **Một box** cho cả xe và thùng | Thùng camper gắn cứng vào xe, không tách rời |
| `img_015` | Xe trong phòng trưng bày, sàn bóng phản chiếu rõ từng chiếc | Gán xe thật, không gán hình dưới sàn | Phản chiếu |
| `img_022` | Ảnh trắng đen, mô tô có thùng bên giữa rừng tuyết | Một box `motorcycle` ôm cả thùng bên | Thùng bên gắn cứng |
| `img_011` | Tượng khủng long và các khối sắt hình xe ở hội chợ | Không gán | Không phải xe hoạt động |

Ca tôi **chưa quyết được**: `img_027`, mô hình xe bán tải trên bệ đá. Nó không chạy được,
nhưng là khối cứng kích thước thật nằm ngay cạnh đường. Tôi nghiêng về không gán, nhưng không
tự tin.

## 5. QC và phiên bản

Ngưỡng theo `taxonomy.json`. Thêm một điều tôi muốn nhóm thống nhất: **người kiểm không được
là người đã gán ảnh đó**. Nhóm ba người thì luôn xếp được người thứ ba đi kiểm.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản giảng viên | Khởi đầu |
| v0.1 (bản này) | Tách `tuktuk` và `cyclo`; thêm `bus` và `bicycle`; điều kiện đầu và thân cho box người; quy tắc ảnh trong nhà | v0 không nói gì về xe ba bánh, ảnh trong nhà và vật không thật |
