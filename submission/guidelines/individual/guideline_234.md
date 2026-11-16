# Guideline gán nhãn — v0 — 234

- Phiên bản: v0
- Ngày: 2026-11-16
- Người viết: Phạm Thị Lương
- Loại nhãn: bounding box trên ảnh đường phố

## 1. Mục tiêu

Gán nhãn các đối tượng thật xuất hiện trong ảnh đường phố để phục vụ phát hiện người và phương tiện trong mô hình 2D. Mục tiêu là đảm bảo mỗi vật được gán đúng lớp, mỗi box phản ánh đúng phần vật thật nhìn thấy trong ảnh, và không nhầm lẫn giữa vật thật với vật phản chiếu, tượng, mô hình đồ chơi, biển quảng cáo, hoặc các vùng quá mờ, quá nhỏ hoặc quá dày không thể tách biệt được.

Trong quá trình gán nhãn, tôi ưu tiên việc phân biệt rõ ràng giữa các đối tượng có hình dạng gần giống nhau nhưng không cùng lớp, ví dụ `car` với `truck`, `car` với `van`, `motorcycle` với `tuk_tuk`, hoặc `person` với tượng/biển quảng cáo. Tôi cũng cần lưu ý rằng một bộ dữ liệu đường phố thực tế thường có nhiều tình huống biên như xe chở xe, xe buýt dài, tuk-tuk ba bánh, vật phản chiếu trên kính, hoặc người đứng trong hình khối quảng cáo. Vì thế, các quy tắc sau đây được viết để làm rõ cách gán nhãn cho những trường hợp dễ bị nhầm.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con, xe du lịch, SUV, taxi, xe hơi cá nhân. | sedan, SUV, hatchback, taxi | `truck`, `bus`, `tuk_tuk`, `motorcycle` |
| `truck` | Xe tải hoặc xe chở hàng có thùng / khung tải lớn tách biệt với cabin. | xe tải, xe chở ô tô | `car`, `van` nếu nhỏ hơn và không có thùng rõ |
| `van` | Xe thương mại cỡ nhỏ hoặc xe thùng kín, lớn hơn ô tô con nhưng nhỏ hơn bus / truck lớn. | van, xe giao hàng nhỏ, minivan | `truck` nếu có thùng rõ ràng, `bus` nếu lớn và dài hơn |
| `bus` | Xe khách lớn, xe buýt, xe khách cỡ lớn. | xe buýt đô thị, xe khách dài | `van` nếu nhỏ hơn và không dài như bus |
| `tuk_tuk` | Xe ba bánh có động cơ, có cabin hoặc mái che cho người và tài xế. | tuk-tuk ba bánh | `motorcycle` |
| `motorcycle` | Xe máy hai bánh có động cơ. | scooter, xe máy | `tuk_tuk`, xe đạp không động cơ |
| `person` | Người thật đang đứng, đi bộ, ngồi hoặc điều khiển xe. | người đi bộ, rider | tượng, biển quảng cáo, phản chiếu |
| `ignore_region` | Vùng quá dày, quá nhỏ, hoặc không thể tách rõ từng vật. | đám đông dày, vật mờ, chi tiết nhỏ | người / xe rõ ràng vẫn phải gán |

Vật không thuộc lớp nào thì không gán. Nếu là tượng, phản chiếu, mô hình, đồ chơi, hoặc biển quảng cáo, bỏ qua. Nếu vật quá nhỏ, quá mờ hoặc không xác định được lớp rõ ràng, dùng `ignore_region` hoặc đặt `can_review=true` để đưa vào rà lại.

### Thuộc tính

#### `occluded_level`
- `0`: không bị che; nhìn rõ
- `1`: che ít (10–30%)
- `2`: che nhiều (31–70%)
- `3`: che rất nhiều (>70%) nhưng vẫn nhận dạng được

Giải thích: mức che được ước lượng theo phần diện tích vật bị phủ, tầm che bởi xe khác, cột, tường, hoặc cây cối. Nếu phần bị che quá lớn làm mất hình dạng chính, tôi ưu tiên đánh dấu `can_review=true` thay vì đoán lớp.

#### `truncated`
- `no`: không bị cắt mép ảnh
- `yes`: vật bị cắt bởi mép ảnh

Khi có phần vật bị cắt, tôi gán `truncated=yes` và chỉ box phần còn thấy trong khung hình, không kéo dài box ra ngoài khung.

#### `can_review`
- `false`: tự tin
- `true`: chưa chắc chắn, cần review lại

Dấu hiệu cần đặt `can_review=true`: không xác định rõ lớp, hình dạng bị mờ, vật nằm ở góc mép ảnh nhưng quá khó phân biệt, hoặc có 2 cách giải thích cùng khả năng.

## 3. Quy tắc vẽ box

1. Box phải ôm sát phần vật nhìn thấy nhất, không để khoảng trống quá lớn.
2. Nếu vật có thể tách thành nhiều phần riêng biệt, mỗi vật phải có box riêng.
3. Nếu vật bị che một phần, box chỉ ôm phần nhìn thấy; không kéo dài sang phần bị che không rõ.
4. Nếu vật bị cắt mép ảnh, gán `truncated=yes` và box chỉ bao phần còn thấy.
5. Không gán phản chiếu, hình vẽ, tượng, mô hình, đồ chơi, biển quảng cáo, hoặc vật không có thật trong cảnh.
6. Nếu vật quá nhỏ, mờ hoặc đám đông dày quá không tách được, dùng `ignore_region`.
7. Với xe chở xe, nếu xe trên thùng là xe con rõ và tách được, gán cả `truck` và từng `car`; nếu không tách được rõ thì chỉ gán `truck` và đặt `can_review=true`.
8. Nếu một người ngồi trên xe máy / rider rõ ràng và tách được, có thể gán cả `person` và `motorcycle`; nếu không tách rõ, gán theo vật chính.
9. Vật có hình dạng rất đặc thù nhưng không phải phương tiện thật (ví dụ tượng, xe mô hình, cảnh phản chiếu) phải bị loại bỏ ngay, dù nhìn rất giống xe hoặc người.
10. Với các vật sáng/đậm bất thường do bóng tối hoặc ánh sáng mạnh, chỉ gán khi nhận dạng được hình khối chính; nếu không chắc, gán `can_review=true` và review lại.

### Một số quy tắc loại trừ rõ ràng

- Không gán `person` cho tượng người, ma-nơ-canh, hoặc hình vẽ trên biển quảng cáo.
- Không gán `motorcycle` cho tuk-tuk ba bánh, xe đạp hoặc xích lô không động cơ.
- Không gán `bus` cho van nhỏ hoặc xe khách cỡ trung nếu thân xe không dài và nhiều cửa sổ rõ.
- Không gán xe trong gương, kính, vũng nước hoặc màn hình LED vì đó không phải vật trong không gian thực của ảnh.
- Không gán `ignore_region` cho vật rõ ràng chỉ vì nó nằm trong đám đông; `ignore_region` chỉ cho đám đông quá dày hoặc vật quá nhỏ không phân tách được.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe chở ô tô trên thùng | R-01 | Gán `truck` cho xe chính; gán thêm `car` nếu các xe con tách rõ | Thùng xe và các xe con là vật riêng |
| `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe ba bánh tuk-tuk | R-02 | Gán `tuk_tuk`, không gán `motorcycle` | Đây là xe ba bánh, không phải mô tô hai bánh |
| `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe buýt / xe khách lớn | R-03 | Gán `bus` | Có thân dài và nhiều cửa sổ như xe khách lớn |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng đồng / tượng đá / tượng ngựa | R-04 | Không gán | Đây là vật trang trí, không phải người thật |
| `img_018.jpg` | Người trên biển quảng cáo | R-04 | Không gán | Hình in trên quảng cáo không phải vật thật |
| `img_020.jpg` | Phản chiếu qua gương/kính | R-04 | Không gán | Không phải vật thật trong cảnh |
| `img_019.jpg`, `img_038.jpg` | Mô hình toy / đồ chơi | R-04 | Không gán | Mô hình / đồ trưng bày không phải phương tiện thực |
| `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp / xích lô không động cơ | R-05 | Không gán `motorcycle` | Không phải xe có động cơ |
| `img_037.jpg` | Bán tải gắn nắp thùng cao / xe tải lạ | R-06 | Chốt theo hình dạng thực tế: nếu rõ là `truck` thì gán `truck`; nếu còn mơ hồ thì `can_review=true` | Đây là ca biên cần nhóm thống nhất |
| `img_033.jpg` | Ma-nơ-canh / trang trí trong tủ kính | R-04 | Không gán | Đây không phải vật thật trong cảnh, mà là sản phẩm trưng bày |

## 5. QC và phiên bản

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản đầu tiên | Dùng để rà các ca mơ hồ và định nghĩa lớp riêng trước khi nhóm thống nhất |

### Tiêu chí QC cá nhân

- Nếu không chắc chắn: đặt `can_review=true` thay vì đoán.
- Nếu một ca lặp lại ở nhiều ảnh hoặc nhiều người hiểu khác nhau, khi họp nhóm sẽ cập nhật guideline và tăng phiên bản.
- Điểm quan trọng: không gán theo cảm giác; chỉ gán khi có bằng chứng trực quan rõ trên ảnh.
- Trước khi submit, nên tự kiểm tra tại 5 ca quan trọng: `img_002`, `img_004`, `img_009`, `img_017`, `img_020` để tránh sai lớp và sai loại trừ.
- Với những ca đang phân vân, ưu tiên đặt `can_review=true` và ghi ảnh để nhóm xem lại ở buổi họp, thay vì cố gắng đoán nhầm.

### Lịch sử cập nhật

- v0: Khởi tạo guideline cá nhân với các lớp chính, quy tắc loại trừ, ca biên, và phương án review khi mơ hồ.
