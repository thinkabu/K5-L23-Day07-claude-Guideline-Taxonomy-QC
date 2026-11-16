# Guideline gán nhãn cá nhân — v0 (bản độc lập) — ảnh đường phố

- **Phiên bản:** v0
- **Ngày:** 16/11/2026
- **Người viết:** Đoàn Thị Ánh Hậu_345
- **Phạm vi:** 40 ảnh `data/images/img_001.jpg`–`img_040.jpg`; box 2D trong CVAT, xuất COCO 1.0
- **Nguồn xây dựng lớp:** rà soát nội dung 40 ảnh trong `data/images/` và mô tả tại `data/ATTRIBUTION.csv`; taxonomy v0 trong `submission/taxonomy.json` chỉ là taxonomy mẫu ban đầu, chưa phản ánh các lớp đề xuất cá nhân dưới đây.


## 1. Mục tiêu

Gán hộp chữ nhật 2D cho người thật và phương tiện đường phố có thể nhận diện trong bộ ảnh: ô tô, xe tải/pickup, van, xe máy, xe buýt, xe ba bánh có động cơ, xe đạp và cyclo/xe kéo đạp. Các lớp được đề xuất từ các đối tượng quan sát thấy trong ảnh, không sao chép danh sách lớp của bản v0. `ignore_region` chỉ đánh dấu vùng có đối tượng thuộc phạm vi nhưng không thể tách đáng tin cậy theo mục 3.

Lỗi nguy hiểm nhất là bỏ sót hoặc gán nhầm người và xe máy trong cảnh đông/che khuất; gán tượng, mannequin, hình in, xe đồ chơi hay ảnh phản chiếu thành vật thật; và xử lý không nhất quán xe chở xe, xe ba bánh hoặc xe không có lớp tương ứng. Không suy diễn đối tượng nằm ngoài khung hình. Chỉ gán theo bằng chứng nhìn thấy trong ảnh.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Một người thật có thể nhận diện được, dù đi bộ, ngồi trên xe, đứng xa, bị che một phần hoặc chỉ thấy một phần cơ thể. | Người bán hàng/khách `img_001`, người trên xe `img_005`, người đi bộ `img_025`, người ở `img_026`. | Tượng (`img_017`), mannequin (`img_033`, `img_038`), hình billboard (`img_018`), hình in/đồ chơi. |
| `car` | Ô tô chở người có thân xe kín, gồm sedan, hatchback, wagon và SUV; không gồm pickup có thùng hàng hở. | Ô tô trong `img_015`, `img_024`, `img_031`, `img_040`; từng ô tô nhìn rõ trên xe chở ở `img_002`. | Pickup thùng hở là `truck`; đồ chơi/mô hình (`img_019`, `img_038`) và xe tượng (`img_027`) không gán. |
| `truck` | Phương tiện cơ giới chở hàng có thùng hoặc kết cấu xe tải rõ ràng; gồm pickup thùng hở, xe tải nhẹ và xe tải chở ô tô. | Pickup `img_001`, `img_012`, `img_028`; xe tải `img_007`, `img_036`; xe chở ô tô `img_002`. | Van khoang kín liền thân là `van`; xe buýt là `bus`; tuk-tuk/auto-rickshaw là `three_wheeler`. |
| `van` | Xe cơ giới có khoang kín, thân liền khối, kiểu van hoặc xe tải van nhỏ; không có thùng hàng hở tách biệt. | Xe van `img_037`; van nhìn rõ trong cảnh `img_009`. | Pickup là `truck` nếu nhận ra kết cấu thùng hở; xe tải lớn là `truck`. Nếu thiếu bằng chứng, bật `can_review`. |
| `motorcycle` | Xe hai bánh có động cơ, gồm xe máy và mô tô; xe máy kéo sidecar vẫn là một `motorcycle` nếu nhận diện được xe máy. | `img_005`, `img_008`, `img_021`, `img_026`, `img_030`, `img_040`; sidecar `img_022`. | Xe đạp là `bicycle`; cyclo là `cycle_rickshaw`; tuk-tuk/xe ba bánh máy là `three_wheeler`. Không gộp người lái vào box xe. |
| `bus` | Phương tiện cơ giới cỡ lớn có khoang chở nhiều hành khách, thân xe dài và nhiều hàng ghế/cửa sổ. | Xe buýt `img_009`. | Xe tải chở hàng là `truck`; van nhỏ là `van`. Nếu chỉ thấy một phần và không đủ nhận diện, bật `can_review`. |
| `three_wheeler` | Phương tiện chở người/hàng có động cơ và ba bánh, gồm tuk-tuk, auto-rickshaw và xe ba bánh máy. | Tuk-tuk `img_004`; xe ba bánh chở hàng/khách `img_010`, `img_016`, `img_035`, `img_040`. | Cyclo/xe kéo đạp là `cycle_rickshaw`; xe máy hai bánh là `motorcycle`. Không suy đoán bánh bị khuất nếu kiểu xe không rõ. |
| `bicycle` | Xe đạp hai bánh không có động cơ nhìn thấy được, dù đang được đẩy, đỗ hoặc có người cưỡi. | Xe đạp trong cảnh giao thông `img_006`, `img_023` nếu nhận diện rõ. | Xe máy là `motorcycle`; xe đạp ba bánh chở khách là `cycle_rickshaw`. Người đi xe là box `person` riêng. |
| `cycle_rickshaw` | Phương tiện chở người/hàng bằng sức đạp, gồm cyclo/pedicab hoặc xe kéo đạp có khoang/ngồi chở riêng. | Cyclo `img_006`, `img_009`, `img_023`, `img_034`, `img_039`. | Xe đạp thường là `bicycle`; tuk-tuk có động cơ là `three_wheeler`. Người đạp/khách là `person` riêng nếu nhìn thấy. |
| `ignore_region` | Hộp vùng có đối tượng thuộc các lớp trên nhưng không thể tách/định vị đáng tin cậy ở độ phân giải ảnh; đây không phải vật thể. | Cụm người hoặc xe máy dày đặc mà từng cá thể không thể khoanh riêng sau khi phóng ảnh. | Không dùng để thay lớp, cho vùng đông nhưng vẫn tách được, vật giả, hoặc che quyết định chưa chắc chắn. |

**Vật ngoài taxonomy:** Không tự tạo lớp ngoài danh sách cá nhân trên. Không gán box cho xe kéo tay không có kết cấu xe đạp/xe cơ giới nhận diện được, động vật, biển báo, cọc tiêu, kiến trúc, người/xe trên ảnh in, tượng, mannequin, mô hình đồ chơi, hoặc vật chỉ xuất hiện dưới dạng phản chiếu. Người thật ngồi trên phương tiện khác vẫn được gán `person` nếu nhìn thấy và đạt ngưỡng kích thước. Không tạo `ignore_region` chỉ vì vật không có lớp.

### Thuộc tính

Áp dụng ba thuộc tính dưới đây cho mọi box đối tượng (`person`, `car`, `truck`, `van`, `motorcycle`, `bus`, `three_wheeler`, `bicycle`, `cycle_rickshaw`). `ignore_region` không cần thuộc tính. Không dùng trạng thái occluded mặc định của CVAT thay cho `occluded_level`; script chỉ đọc thuộc tính đã định nghĩa.

| Thuộc tính | Giá trị cố định | Quy tắc chọn |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần diện tích/hình dáng dự kiến của vật bị vật khác che, không tính phần nằm ngoài mép ảnh. `0`: bị che dưới 5% hoặc không bị che; `1`: từ 5% đến dưới 30%; `2`: từ 30% đến 60% (kể cả 60%); `3`: trên 60%, nhưng vẫn đủ dấu hiệu để nhận diện lớp và vị trí vật. Ước lượng nhất quán bằng phần thân/vùng đối tượng thực sự bị che, không lấy phần trống bên trong box. Nếu không thể ước lượng vì ảnh quá mờ/tối, đánh dấu `can_review`. |
| `truncated` | `no`, `yes` | `yes` khi phần vật bị khung ảnh cắt mất tại một cạnh ảnh; `no` nếu toàn bộ vật nằm trong khung, kể cả khi bị vật khác che. Chạm sát cạnh nhưng còn thấy đủ đường biên không tự động là truncated. |
| `can_review` | checkbox `true` / `false` | Bật khi có nghi vấn thực sự về vật có thuộc lớp hay không, phân lớp, số đối tượng, ranh giới box, hoặc thuộc tính. Không bật chỉ vì vật bị che/cắt nhưng quy tắc đã giải quyết được. Nêu vấn đề cấp ảnh riêng nếu không thể đặt box. |

## 3. Quy tắc vẽ box

Áp dụng lần lượt các bước sau cho từng ảnh:

1. **Xác định vật thật và thuộc phạm vi** theo bảng lớp. Chỉ gán người thật và phương tiện có lớp được liệt kê. Không suy ra vật bị khuất hoàn toàn từ ngữ cảnh.
2. **Một box cho mỗi cá thể/vật thể có thể tách.** Vẽ box người riêng và box xe riêng, kể cả người đang lái/ngồi trên xe. Người lái nhiều người trên cùng một xe: mỗi người một box nếu phân biệt được. Không gộp nhiều xe máy đỗ cạnh nhau thành một box.
3. **Ôm sát phần nhìn thấy.** Box chữ nhật theo trục ảnh, bao phần pixel nhìn thấy của đối tượng, không bao vùng bị che hoặc phần cơ thể/xe phỏng đoán phía sau vật che. Không kéo box bao cả người và xe, hàng hóa, bóng, phản chiếu hay khoảng trống. Với xe máy có người ngồi, box `motorcycle` ôm phần nhìn thấy của xe; box `person` ôm phần nhìn thấy của từng người.
4. **Che khuất.** Nếu nhận diện được vật và phần thấy được có thể khoanh, vẽ box quanh phần thấy được, gán `occluded_level` theo ngưỡng mục 2. Nếu vật bị che hoàn toàn thì không tạo box. Nếu còn dấu hiệu nhưng ranh giới không đủ chắc để đặt box, bật `can_review` nếu vẫn có thể đặt vùng hợp lý; nếu không thể tách nhiều vật trong một khu vực, cân nhắc `ignore_region` theo điều kiện bên dưới.
5. **Cắt mép.** Với vật đi vào/ra khỏi ảnh, vẽ từ cạnh ảnh tới biên phần vật thấy được, không mở rộng ngoài ảnh; đặt `truncated=yes`. Vật bị che bởi vật khác nhưng không bị cạnh ảnh cắt vẫn `truncated=no`.
6. **Ngưỡng kích thước.** Mỗi box đối tượng phải cao ít nhất **20 px** trên ảnh gốc đã cung cấp. Đối tượng thấp hơn 20 px không tạo box riêng; không phóng ảnh để thay đổi ngưỡng. Nếu cụm đối tượng nhỏ/dày đặc làm mất khả năng gán nhất quán, ghi nhận vấn đề và chỉ dùng `ignore_region` khi đúng định nghĩa (đối tượng trong phạm vi nhưng không thể tách đáng tin cậy). Không dùng ignore để bao trùm các vật nhỏ đơn lẻ có thể đánh giá được.
7. **Đám đông và cụm phương tiện.** Gán từng người/xe riêng nếu từng vật có thể nhận diện, ranh giới thấy được và box đạt 20 px. Không đặt một box chung quanh cả đám. Chỉ đặt `ignore_region` quanh phần cụm không thể tách; giữ box riêng cho vật tách được bên ngoài hoặc ở rìa cụm. Không vẽ box đối tượng nằm trong vùng ignore nếu thực tế không thể xác định biên riêng.
8. **Vật nhiều phần/xe chở xe.** Xe đầu kéo và rơ-moóc chở ô tô ở `img_002`: một `truck` cho toàn bộ xe tải kéo/rơ-moóc theo đường bao phần xe tải nhìn thấy; mỗi ô tô được chở mà nhận diện riêng được là một `car` riêng, box theo phần nhìn thấy của chiếc xe, dù bị tầng/ramp che. Không gán riêng rơ-moóc thành lớp mới. Với xe máy sidecar `img_022`, gán một box `motorcycle` cho cụm phương tiện nếu nhận diện được xe máy; không tạo box riêng cho sidecar. Gán người riêng nếu nhìn thấy.
9. **Xe ba bánh và các nhóm xe đạp.** Tuk-tuk/auto-rickshaw và xe ba bánh có động cơ là `three_wheeler`; cyclo/xe kéo đạp là `cycle_rickshaw`; xe đạp thường là `bicycle`. Không đổi các lớp này thành `motorcycle`. Gán người thật trên xe thành box `person` riêng. Đây là taxonomy cá nhân đề xuất; trước khi dùng CVAT, nhóm cần thống nhất các lớp và cập nhật taxonomy dùng chung.
10. **Tượng, hình ảnh, mannequin, đồ chơi và phản chiếu.** Không gán người/xe giả như tượng ở `img_017`, billboard ở `img_018`, xe đồ chơi ở `img_019`, xe trưng bày/mô hình ở `img_027`, mannequin/đồ chơi trong cửa hàng ở `img_033`/`img_038`. Không gán ảnh phản chiếu trên gương (`img_020`) hoặc mặt đường ướt như một đối tượng thứ hai. Nếu một người thật chỉ được nhìn thấy qua gương/phản chiếu, guideline dự thảo này vẫn không gán ảnh phản chiếu; nhóm cần chốt một cách áp dụng thống nhất cho trường hợp phát sinh.
11. **Xe trong nhà/đang trưng bày.** Vật có vẻ là xe thật nhưng đang ở showroom/garage/trưng bày, không phải ảnh mô hình, vẫn là vật thể thật thuộc lớp (ví dụ xe ở `img_007`, `img_024`); không đòi hỏi vật phải đang lưu thông. Nếu không phân biệt chắc xe thật với mô hình, bật `can_review`.
12. **Kiểm tra box trước khi lưu.** Box nằm trong biên ảnh, có chiều rộng/chiều cao dương, ôm sát vật nhìn thấy, không chồng thành box gộp hai đối tượng. Box dưới 20 px chiều cao sẽ không qua validator. Gán đầy đủ thuộc tính và dùng đúng tên lớp viết thường trong taxonomy.

**Quy tắc `ignore_region`:** chỉ dùng khi (a) vùng có người/phương tiện thuộc taxonomy; (b) do che khuất, mật độ hoặc độ phân giải, không thể khoanh riêng các vật một cách có ý nghĩa; và (c) một reviewer có thể xác định rõ vùng được loại khỏi đánh giá. Vẽ box vùng nhỏ nhất bao trọn phần không thể tách, không bao thêm vùng nền/vật gán được. Không dùng cho vùng không thuộc taxonomy, vật giả, vật mờ đơn lẻ, hoặc để né lựa chọn lớp. Khi không chắc vùng có đủ điều kiện, ghi issue cấp ảnh và bật `can_review` trên các box liên quan nếu có.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định dự thảo | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở nhiều ô tô | R-08 | Một `truck` cho xe tải/rơ-moóc; mỗi ô tô nhìn thấy, phân biệt được và cao ≥20 px là một `car` riêng; box theo phần nhìn thấy. | Vật được chở vẫn là đối tượng riêng; tránh gộp các xe vào xe tải. |
| `img_004`, `img_010`, `img_016`, `img_035`, `img_040` | Tuk-tuk, xe ba bánh/auto-rickshaw | R-09 | Gán phương tiện là `three_wheeler`; người thật là `person` riêng. | Hình dáng có động cơ và ba bánh tạo nhóm phân biệt với xe máy hai bánh. |
| `img_005`, `img_026` | Người ngồi/lái xe máy, có thể bị che | R-02, R-03 | Mỗi người thấy được là `person`, xe là `motorcycle`; hai box độc lập, box chỉ bao phần nhìn thấy; thuộc tính phản ánh mức che. | Người và xe là hai đối tượng, không gộp theo quan hệ lái xe. |
| `img_006`, `img_009`, `img_023`, `img_034`, `img_039` | Cyclo/xe đạp và người điều khiển | R-09 | Cyclo là `cycle_rickshaw`, xe đạp thường là `bicycle`; gán người thật riêng. | Hai loại phương tiện có cấu tạo và cách vận hành khác nhau, cùng xuất hiện trong bộ ảnh. |
| `img_017` | Tượng người cưỡi ngựa | R-10 | Không gán tượng, ngựa hoặc người cưỡi tượng. | Không phải người/phương tiện thật trong cảnh. |
| `img_018` | Hình người trên billboard | R-10 | Không gán hình người trên biển quảng cáo. | Người được mô tả trong ảnh in, không phải người hiện diện trong cảnh. |
| `img_019`, `img_027`, `img_038` | Xe đồ chơi, pickup tượng/trưng bày, mô hình xe trong cửa hàng | R-10 | Không gán đối tượng đồ chơi, mô hình hoặc tượng. | Phân biệt đối tượng vật lý thật dùng trong giao thông với hình/đồ vật mô phỏng. |
| `img_020`; phản chiếu mặt đường tại `img_015`/`img_031` | Vật/xe chỉ hiện trong gương hoặc phản chiếu | R-10 | Không tạo box riêng cho ảnh phản chiếu. Gán xe/người thật trong cảnh nếu nhìn thấy trực tiếp. | Tránh nhân đôi cùng vật thể hoặc coi ảnh phản chiếu là vật thể độc lập. |
| `img_001`, `img_012`, `img_028` | Pickup có thùng hở | R-01 | Gán `truck`, kể cả đang đỗ và có người/hàng hóa gần đó. | Pickup có thùng hàng hở thuộc định nghĩa xe tải của taxonomy. |
| `img_007`, `img_036` | Xe tải/xe tải nhỏ trong khu vực trong nhà hoặc đường phố | R-01, R-11 | Gán `truck` nếu là xe thật và đủ ngưỡng, không phụ thuộc bối cảnh đang chạy hay đỗ. | Phạm vi xét loại đối tượng, không xét xe có đang lưu thông hay không. |
| `img_008`, `img_021`, `img_030` | Nhiều xe máy đỗ/di chuyển gần nhau | R-02, R-07 | Tách từng xe có biên nhận diện và cao ≥20 px; người thật tách riêng. Dùng `ignore_region` chỉ cho phần cụm không thể tách. | Box riêng giữ đơn vị đối tượng; vùng ignore không thay cho toàn cảnh. |
| `img_003`, `img_025`, `img_032` | Người nhỏ/đông người ở xa | R-06, R-07 | Gán từng người cao ≥20 px nếu phân biệt được; dưới 20 px bỏ box. Vùng cụm chỉ được ignore nếu không thể tách các đối tượng thuộc phạm vi. | Ngưỡng pixel cố định giúp các annotator xử lý thống nhất. |
| `img_033`, `img_038` | Mannequin/hình nhân trong cửa hàng | R-10 | Không gán `person`. | Hình người không phải người thật. |
| Bất kỳ ảnh nào | Người hoặc xe bị cắt cạnh ảnh | R-05 | Vẽ phần nhìn thấy trong khung; `truncated=yes`; `occluded_level` chỉ tính che bởi vật khác. | Phân biệt cắt khung và che khuất. |
| Bất kỳ ảnh nào | Không chắc xe là `car`, `truck` hay `van`; không rõ vật thật/giả | R-12 | Chọn lớp chỉ khi có bằng chứng hình dáng đủ rõ; nếu vẫn chưa phân giải được, đặt `can_review=true` và ghi vùng/lý do. | Không đoán lớp theo kích thước đơn thuần hay ngữ cảnh. |

Nếu tình huống chưa được giải quyết bởi quy tắc hiện hành, không tự tạo lớp hoặc thay taxonomy. Ghi tên ảnh, vị trí tương đối (ví dụ “nửa dưới, bên trái”), vấn đề và cách bạn đang hiểu; bật `can_review` nếu có box liên quan. Reviewer ghi mã Rule ID và quyết định ở nhật ký QC. Chỉ dùng `ignore_region` khi đúng định nghĩa nêu trên.

## 5. QC và phiên bản

### Kiểm tra chất lượng

- **Trước khi nộp nhãn cá nhân:** xem lại cả 40 ảnh được phân công; xác nhận ảnh không có đối tượng vẫn xuất hiện trong COCO; kiểm tra vật bị bỏ sót, box gộp, lớp, chiều cao box ≥20 px, thuộc tính và `can_review`.
- **So sánh giữa annotator:** dùng script chính thức và taxonomy đã khóa. Ngưỡng trong `submission/taxonomy.json` phải giữ nguyên: box cao ≥50 px đạt ngưỡng IoU ghép 0.85; box nhỏ hơn 50 px đạt ngưỡng 0.70; ghép box dùng `match_iou=0.5`; ngưỡng IAA IoU trung bình ≥0.85, đồng thuận lớp ≥0.95; ngưỡng lỗi ảnh QC tối đa 0.02. Đây là ngưỡng công cụ/lab, không thay cho xem ảnh bằng mắt.
- **Duyệt thủ công:** kiểm cả box đã ghép và chưa ghép, bỏ sót, box thừa, sai lớp/ranh giới/thuộc tính, vùng ignore và ca cả nhóm cùng bỏ sót hoặc cùng hiểu sai. Script không xác nhận vật đó có thật hay box có ôm sát vật.
- **Quyết ca chưa rõ:** người được nhóm chỉ định làm reviewer chốt quyết định; ghi `image`, vùng/vật thể, Rule ID, quyết định, lý do, người chốt, phiên bản guideline, phạm vi sửa (`anh` hoặc `lo`) và người kiểm lại. Người gán không tự xóa dấu vết bất đồng; quyết định calibration phải truy lại được đến ảnh và nhãn.
- **Release / Hold / Rework:** chỉ Release khi đủ lượt gán, nhãn và phiên bản nhất quán, QC có bằng chứng và không vượt ngưỡng lỗi. Hold khi còn thiếu dữ liệu/bằng chứng hoặc câu hỏi phạm vi chưa chốt. Rework khi lỗi vượt ngưỡng hoặc lỗi có tính lặp lại; sửa theo phạm vi ảnh hoặc toàn lô tùy thay đổi ngữ nghĩa. Người kiểm xác nhận quyết định cuối.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Taxonomy mẫu ban đầu gồm `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`. | `guideline/guideline_v0_starter.md`; dùng làm điểm xuất phát để đối chiếu, không sao chép nguyên danh sách lớp. |
| `v0-individual` | Taxonomy cá nhân mở rộng theo ảnh: thêm `bus`, `three_wheeler`, `bicycle`, `cycle_rickshaw`; làm rõ định nghĩa lớp, ngưỡng pixel, box, thuộc tính và ca biên. | Các phương tiện nhìn thấy trong bộ ảnh; đề xuất cá nhân cần được nhóm xem xét khi chốt taxonomy. |
| `v1.1` | Chưa phát hành. Ghi từng quyết định calibration đã duyệt tại đây, kèm Rule ID, phạm vi ảnh/lô và người duyệt. | Chỉ cập nhật sau lượt gán độc lập và họp calibration thực tế. |

### Danh sách quy tắc tra cứu nhanh

| Rule ID | Quy tắc |
|---|---|
| `R-01` | Phân biệt `car`/`truck`/`van` theo cấu trúc thân xe; pickup thùng hở là `truck`. |
| `R-02` | Một box cho mỗi người/xe; người và phương tiện là hai đối tượng riêng. |
| `R-03` | Box chỉ ôm phần nhìn thấy; phân mức che theo tỷ lệ quy định. |
| `R-04` | Phân biệt `occluded_level` với `truncated`; chỉ cắt mép ảnh mới là truncated. |
| `R-05` | Đối tượng bị cắt khung: box trong ảnh và `truncated=yes`. |
| `R-06` | Box đối tượng có chiều cao tối thiểu 20 px. |
| `R-07` | Tách vật trong đám đông/cụm xe; dùng `ignore_region` hạn chế cho vùng không thể tách. |
| `R-08` | Xe chở xe: gán riêng xe tải và từng ô tô nhìn thấy, nhận diện được. |
| `R-09` | Gán tuk-tuk/auto-rickshaw là `three_wheeler`, cyclo là `cycle_rickshaw`, xe đạp là `bicycle`, xe buýt là `bus`; không ép vào lớp gần giống. |
| `R-10` | Không gán tượng, hình in, mannequin, đồ chơi/mô hình hoặc phản chiếu. |
| `R-11` | Xe thật được gán dù đang đỗ, trưng bày hoặc ở trong nhà. |
| `R-12` | Khi chưa đủ bằng chứng, không đoán; bật `can_review`/ghi issue và chuyển reviewer. |





