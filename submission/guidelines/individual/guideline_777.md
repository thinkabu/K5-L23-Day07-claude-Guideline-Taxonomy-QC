# Guideline gán nhãn — bản cá nhân — 777

- Phiên bản: v0.1 (bản cá nhân Pha 1)
- Ngày: 16/11/2026
- Người viết: 777

## 1. Mục tiêu

Nhãn cho mô hình phát hiện người và phương tiện 2D. Tôi đọc v0 với một câu hỏi duy nhất: **mô
hình sẽ học được gì từ cái box này?** Một box đúng về mặt hình học nhưng dạy sai khái niệm thì
tệ hơn là không có box.

Lỗi nặng nhất vẫn là bỏ sót người và xe trên phần đường xe chạy. Nhưng tôi xếp ngay sau đó một
lỗi mà v0 không hề nhắc tới: **box dạy mô hình một khái niệm không tồn tại**, ví dụ gán một bức
tượng thành xe.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa một câu | Ví dụ | Phản ví dụ, dễ nhầm |
|---|---|---|---|
| `car` | Xe bốn bánh chở người, khoang khách kín liền thân. | `img_015`, `img_031`, `img_040` taxi | Bán tải thuộc `truck` |
| `van` | Xe thùng kín một khối, dưới 4 ô cửa sổ bên. | `img_005`, `img_016` | `img_009` thuộc `bus` |
| `bus` | Xe khách, từ 4 ô cửa sổ bên trở lên. | `img_009`, `img_037` | `van`, `truck` thùng kín |
| `truck` | Thùng hoặc bệ chở tách khỏi cabin, gồm bán tải. | `img_002`, `img_007`, `img_012`, `img_036` | `van` |
| `motorcycle` | Xe hai bánh có động cơ. | `img_008`, `img_026`, `img_030` | `bicycle` |
| `three_wheeler` | Xe ba bánh chở người hoặc hàng. | `img_004`, `img_016`, `img_035` | `motorcycle` |
| `bicycle` | Xe đạp hai bánh, có bàn đạp. | `img_003`, `img_006`, `img_029` | `motorcycle` ở xa |
| `person` | Người thật. | `img_014`, `img_034` | Tượng, ma-nơ-canh, hình in |
| `ignore_region` | Vùng bỏ qua. | Bãi xe `img_030` | Không phủ lên box đã gán |

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | 0, 1, 2, 3 | 0 không che; 1 dưới 50%; 2 từ 50%; 3 không đọc được |
| `truncated` | no, yes | Chạm mép ảnh |
| `can_review` | tick | Không chắc lớp hoặc không chắc quy tắc |

Tôi đề nghị thêm một thuộc tính `load_attached` cho trường hợp box phải ôm cả hàng hoá gắn trên
xe. Không có nó thì sau này không lọc ra được các box bị phình to vì hàng, mà box phình to lại
là nguồn lỗi IoU lớn nhất khi so hai người gán.

## 3. Quy tắc vẽ box

1. Ôm sát phần nhìn thấy, không đoán phần khuất, sai số ±2 px.
2. Một vật một box; soát các box chồng nhau trên 80% trước khi lưu.
3. Người điều khiển xe hai và ba bánh có box `person` riêng.
4. **Hàng hoá gắn trên xe nằm trong box xe**, kể cả khi hàng cao hơn xe.
5. Vật cao dưới 20 px: không gán riêng, phủ `ignore_region`.
6. **Cụm vật chồng nhau tới mức không chỉ ra được mép từng chiếc: phủ `ignore_region`**, dù từng
   chiếc có cao hơn 20 px. Gán bừa trong đám đông tạo ra box sai, và box sai hại hơn box thiếu.
7. Không gán vật ảo: phản chiếu, hình in, tượng, thân xe chủ.
8. Người kiểm một ảnh không được là người đã gán ảnh đó.

## 4. Ca khó tôi tìm được trong 40 ảnh

| Ảnh | Tình huống | Quyết định ban đầu của tôi | Lý do |
|---|---|---|---|
| `img_002` | Đầu kéo chở sáu ô tô trên hai tầng giá | **Một box duy nhất** cho cả xe chở và khối xe trên giá, bật `load_attached` | Sáu chiếc đó không tự chạy, chúng là hàng. Với bài toán tránh va chạm thì cả cụm là một vật cản duy nhất |
| `img_030` | Bãi xe máy xếp chồng kín khung hình | Phủ `ignore_region` gần hết ảnh, chỉ gán vài chiếc tiền cảnh tách được biên | Hàng sau không chỉ ra được mép từng chiếc. Vẽ đại sẽ tạo hàng chục box sai |
| `img_017` | Tượng đồng người cưỡi ngựa đội nón giao thông | Không gán gì | Tượng không di chuyển. Đây là ảnh tôi nghĩ nhiều nhất: detector chắc chắn sẽ bắn box `person` vào đó |
| `img_038` | Tủ kính bày mấy chục ô tô đồ chơi | Không gán, phủ `ignore_region` cả tủ | Đồ chơi không phải xe thật và không đúng kích thước thật |
| `img_022` | Ảnh trắng đen, mô tô có thùng bên giữa rừng tuyết | Một box `motorcycle` ôm cả thùng bên, bật `can_review` | Thùng bên gắn cứng. Có một vệt tối sau yên mà tôi không đọc được là người hay kính chắn gió |

## 5. QC và phiên bản

Ngưỡng theo `taxonomy.json`. Điều tôi muốn cả nhóm ghi nhớ khi đọc kết quả Pha 4: **IoU trung
bình và tỉ lệ cùng lớp chỉ tính trên các cặp box ghép được.** Chỗ một người vẽ mà người kia
không vẽ gì thì không tạo ra cặp nào, nên nó không hạ một chỉ số nào cả. Vì vậy mỗi lần báo IAA
phải báo kèm số box chưa ghép, nếu không thì con số đẹp sẽ che mất đúng chỗ nhóm bất đồng nhất.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản giảng viên | Khởi đầu |
| v0.1 (bản này) | Thêm `bus`, `three_wheeler`, `bicycle`; thêm `load_attached`; quy tắc cụm không tách được biên; quy tắc người kiểm khác người gán | v0 không có lớp cho xe ba bánh và không nói gì về đám đông hay vật không thật |
