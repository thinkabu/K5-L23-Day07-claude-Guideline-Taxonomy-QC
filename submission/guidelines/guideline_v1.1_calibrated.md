# Guideline gán nhãn — v1.1 — nhóm claude (sau calibration)

- Phiên bản: **v1.1**, thay cho v1.0 từ 16/11/2026
- Loại thay đổi: **semantic** (xem `submission/guideline_change.json`) — đổi cách chọn vật, nên
  **phải rà lại cả lô 40 ảnh**, không chỉ 7 ảnh bị cờ
- Người chốt: 209 (điều phối), đồng thuận với 123, 234, 345 và 777 tại buổi calibration Pha 5
- Căn cứ: `submission/qc/iaa_summary.md` và 24 ca trong `calibration_candidates.csv`

> v1.1 **giữ nguyên toàn bộ** mục 1, 2, 3 và 5 của v1.0, trừ ba chỗ ghi rõ dưới đây. Taxonomy
> không đổi: vẫn 8 lớp và `ignore_region`, vẫn 4 thuộc tính, `submission/taxonomy.json` phiên
> bản `v1.0` tiếp tục có hiệu lực vì không có lớp hay thuộc tính nào được thêm hoặc bỏ.

---

## 1. Số đo dẫn tới bản này

| Chỉ số | Giá trị đo được | Đọc thế nào |
|---|---|---|
| Lượt gán | 120/120 (5 người, mỗi ảnh 3 người) | Đủ, có thể đo IAA |
| Cặp box ghép được | 1066 | Mẫu số của hai chỉ số dưới |
| IoU trung bình | 1.000 | ĐẠT ngưỡng 0.85 |
| Tỉ lệ cùng lớp | 100% | ĐẠT ngưỡng 95% |
| Box chưa ghép được | 44 (36 của A, 8 của B) | **Không nằm trong hai chỉ số trên** |
| Ca cần xem | 77, trên 8 ảnh | Nguồn của các quyết định bên dưới |

**Điều quan trọng nhất học được ở Pha 4:** cả mười cặp người gán đều ĐẠT cả hai chỉ số, trong khi
lô vẫn còn 44 box chỉ một người vẽ và 77 ca bất đồng. Lý do là cả hai chỉ số chỉ tính trên **cặp box ghép được**: chỗ
một người vẽ còn người kia không vẽ gì thì không có cặp nào để tính, nên bất đồng lớn nhất lại vô
hình với số liệu. Mọi báo cáo QC từ nay phải đọc IoU và tỉ lệ cùng lớp **kèm số box chưa ghép**.

## 2. H-A — Xe nằm trên một xe khác

**Quyết định (R-07 viết lại).** Hàng hoá gắn liền nằm trong box xe chở, trừ một trường hợp:

> Nếu khối hàng **bản thân nó là một phương tiện** thuộc danh sách 8 lớp và cao từ 20 px trở lên,
> thì nó được **một box riêng theo đúng lớp của nó**, và box của xe chở **không** nới ra để bao nó.
> Khi đó `load_attached = false` cho xe chở, vì phần hàng đã được gán riêng. Hai box được phép
> chồng nhau theo R-13.
>
> Hàng hoá **không phải phương tiện** (thùng các-tông, bao tải, thùng camper, tượng) vẫn nằm trong
> box xe chở, `load_attached = true`.

**Vì sao.** Mô hình phải nhận ra một chiếc ô tô là ô tô ở mọi vị trí trong khung hình. Nếu sáu
chiếc xe trên giá của `img_002` bị nuốt vào box đầu kéo, mô hình học rằng xe ở độ cao đó không
phải xe. Ngược lại, một thùng camper hay một tượng ngựa không thuộc lớp nào nên không có cách nào
gán riêng, và nó vẫn chiếm chỗ thật trên đường, nên phải nằm trong box xe chở.

**Ảnh phải sửa:** `img_002` (6 xe trên giá, đầu kéo bỏ cờ `load_attached`), `img_024` (xe vàng
trên nóc xe đen). `img_028` và `img_037` **không** đổi: tượng ngựa và thùng camper không phải
phương tiện.

## 3. H-B — Cụm xe dày đặc nhưng từng chiếc cao hơn 20 px

**Quyết định (R-09 viết lại).** Chỉ được dùng `ignore_region` trong đúng hai trường hợp:

> **(a) Vật nhỏ:** chiều cao box dưới 20 px ở độ phân giải gốc.
> **(b) Cụm không tách được biên:** các vật cùng lớp chồng lên nhau tới mức không chỉ ra được đâu
> là mép của từng chiếc — cụ thể là **mỗi vật bị các vật cùng lớp khác che từ 70% trở lên**, hoặc
> cụm có **trên 15 vật** mà không đếm tách ra được.
>
> **Dày đặc nhưng vẫn tách được biên thì phải gán từng chiếc.** Mỏi tay không phải là lý do.
> Vật tách được biên nằm ở rìa cụm thì gán riêng, và vùng `ignore_region` vẽ lùi lại để không
> chồng lên box đã gán.

**Vì sao.** Đây là bất đồng đắt nhất đo được: `img_021` lệch 14 ca và `img_030` lệch 22 ca, cộng
lại là 36 trong tổng số 77 ca của cả lô. Nếu
để mỗi người tự quyết "thế nào là đông", cùng một bãi xe sẽ có người gán 15 box và người gán 0
box, và chênh lệch đó không hiện ra ở IoU trung bình.

**Ảnh phải sửa:** `img_021` (giữ hai vùng `ignore_region` cho phần xa không tách được biên, gán
riêng các xe hàng đầu), `img_030` (giữ hai vùng cho hàng xa, gán riêng ba xe tiền cảnh).

## 4. H-C — Vật ba chiều kích thước thật nhưng không phải xe hoạt động hay người sống

**Quyết định (R-10 mở rộng).** Phép thử một câu:

> **Vật đó có tự di chuyển được trên đường, hoặc có phải một người đang sống, hay không?**
> Không thì **không gán**, dù nó là khối ba chiều đặc và đúng kích thước thật.
>
> Không gán: tượng đài, đài tưởng niệm hình xe, ma-nơ-canh, xe mô hình, xe đồ chơi, tác phẩm điêu
> khắc hình xe, hình in, ảnh phản chiếu, thân xe chủ.
> Có gán: xe hóa trang, xe độ, xe cổ, xe hỏng còn nguyên hình dáng — **chúng là xe chạy được**.
>
> Khi **từ 3 vật không thật trở lên** nằm thành cụm và mô hình dễ nhầm (tủ xe đồ chơi, dãy tác
> phẩm điêu khắc), phủ `ignore_region` lên cụm để đánh dấu vùng không chấm. Vật không thật **đứng
> một mình** thì chỉ cần không gán, không phủ `ignore_region`, vì không có gì để bỏ sót.

**Vì sao.** Lý lẽ của 102 — "khối ba chiều kích thước thật vẫn chắn chỗ, đâm vào vẫn hỏng xe" —
đúng về vật lý nhưng sai về mục tiêu của bộ nhãn này. Nhãn dùng để huấn luyện **phát hiện phương
tiện và người**, không phải phát hiện vật cản nói chung. Dạy mô hình rằng một đài tưởng niệm hình
xe bán tải là `truck` sẽ làm nó bắn box `truck` vào mọi bức tượng ven đường. Việc phát hiện vật
cản tĩnh thuộc về một lớp khác hoặc một cảm biến khác, không nhét vào lớp `truck`.

**Ảnh phải sửa:** `img_027` (bỏ box `truck` trên đài tưởng niệm), `img_033` (bỏ box `person` trên
ma-nơ-canh), `img_019` (**giữ** box `car` cho xe hóa trang Danger Mouse: có biển số, có người lái,
chạy được). `img_017`, `img_020`, `img_038`, `img_011` đã đúng từ v1.0.

## 5. Hai điều nhỏ chốt thêm

**R-15 — Người ngồi trong khoang kín không gán, kể cả khi nhìn rõ qua cửa kính.** Ba box `person`
trong khoang xe khách `img_009` đã bị bỏ đúng theo R-08; ghi lại đây vì đọc R-08 dễ bỏ sót vế này.

**R-16 — Thuộc tính `occluded_level` chấm theo phần thân vật bị che thật, không theo box chồng
nhau.** Một xe nằm lọt trong box bao của một xe lớn hơn chưa chắc bị che: `img_028` chiếc xe đỏ
nằm trong box của xe bán tải nhưng chỉ bị che chừng một phần ba. Khi hai cách đọc cho kết quả
khác nhau, chọn mức thấp hơn và nhìn lại ảnh.

## 6. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên: 5 lớp, 3 quy tắc | Khởi đầu |
| v1.0 | 8 lớp và `ignore_region`, R-01…R-14, 13 ca khó, 3 câu hỏi để ngỏ | Hợp nhất ba bản cá nhân |
| **v1.1** | R-07 viết lại (H-A), R-09 viết lại (H-B), R-10 mở rộng (H-C), thêm R-15 và R-16 | 77 ca trong `calibration_candidates.csv`, trên 8 ảnh: `img_002`, `img_013`, `img_019`, `img_021`, `img_022`, `img_024`, `img_030`, `img_033` |

Thay đổi **semantic**: cách chọn vật đổi, nên toàn bộ 40 ảnh phải rà lại, không chỉ 7 ảnh bị cờ.
