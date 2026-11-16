# Guideline gán nhãn — v1.0 — nhóm claude

- Phiên bản: **v1.0** (khóa lúc kết thúc Pha 2, dùng cho toàn bộ lượt gán độc lập)
- Ngày: 16/11/2026
- Người viết: 123, 209, 234, 345, 777 · Người duyệt: 209 (điều phối)
- Loại nhãn: bounding box 2D, công cụ CVAT, xuất COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy.json` phiên bản `v1.0`

> Bản này hợp nhất năm guideline cá nhân `guideline_123.md`, `guideline_209.md`,
> `guideline_234.md`, `guideline_345.md`, `guideline_777.md`. Mục 6 ghi lại chỗ năm bản khác nhau và lý do nhóm chọn phương án
> cuối. Trong lượt gán độc lập, **chỉ bản này có hiệu lực**; bản cá nhân chỉ để đối chiếu.

---

## 1. Mục tiêu

### 1.1 Mô hình nào dùng bộ nhãn này

Nhãn dùng để huấn luyện một mô hình **phát hiện vật thể 2D** chạy trên camera trước của xe
tự lái và trên camera giao thông cố định. Đầu ra của mô hình là lớp và hộp bao của từng
người, từng phương tiện trong khung hình. Mô hình này không ước lượng khoảng cách, không
theo vết, không đọc biển số.

### 1.2 Sai ở đâu thì nguy hiểm

Thứ tự ưu tiên khi phải đánh đổi, đã thống nhất trong Pha 2:

| Ưu tiên | Loại lỗi | Vì sao |
|---|---|---|
| 1 | **Bỏ sót** một người hoặc một xe đang ở trên phần đường xe chạy | Mô hình học rằng chỗ đó trống. Đây là lỗi dẫn tới va chạm. |
| 2 | **Gán nhầm sang lớp khác hẳn** (`person` thành `motorcycle`) | Hành vi tránh né của xe khác nhau theo lớp. |
| 3 | **Box lỏng hoặc lệch** | Làm hỏng ước lượng vị trí, nhưng vật vẫn được phát hiện. |
| 4 | **Nhầm trong cùng nhóm** (`van` với `bus`) | Hai lớp cùng là khối lớn bốn bánh, hậu quả điều khiển gần như nhau. |

Hệ quả thực tế: khi lưỡng lự giữa "gán và có thể sai lớp" với "không gán", **luôn gán**, chọn
lớp gần nhất và bật `can_review`.

### 1.3 Vì sao có lớp `person` trong bài nhận diện phương tiện

Người đi bộ và người điều khiển xe hai bánh là vật cản di chuyển khó đoán nhất trên đường
châu Á. Một bộ nhãn chỉ có xe sẽ dạy mô hình bỏ qua đúng đối tượng dễ tổn thương nhất.

### 1.4 Phạm vi lô L23-01

40 ảnh `img_001.jpg` … `img_040.jpg`, cạnh dài tối đa 1024 px, nguồn Wikimedia Commons,
ghi công trong `data/ATTRIBUTION.csv`. Ảnh ban ngày, ban đêm, trời mưa, ảnh trắng đen và ảnh
chụp trong nhà đều nằm trong phạm vi gán.

---

## 2. Lớp và thuộc tính

### 2.1 Phép thử dùng để quyết định lớp hay thuộc tính

> **Mô hình có phải đoán ra điều này từ ảnh không?** Có thì là **lớp**. Không, mà là thông tin
> phục vụ lọc dữ liệu và chấm QC, thì là **thuộc tính**.

Thêm một lớp buộc phải huấn luyện lại; thêm một thuộc tính thì không. Vì vậy danh sách lớp
giữ ở mức nhỏ nhất đủ dùng.

### 2.2 Cây phân loại

```
Đối tượng
├── Bốn bánh trở lên
│   ├── car            khoang khách kín liền khối, không thùng tách rời
│   ├── van            thùng kín một khối, dưới 4 ô cửa sổ bên
│   ├── bus            từ 4 ô cửa sổ bên trở lên, hoặc có cửa khách riêng
│   └── truck          thùng / khung / bệ chở tách khỏi cabin (gồm bán tải)
├── Ba bánh
│   └── three_wheeler  tuk-tuk, auto-rickshaw, xe lam, xe ba gác, xích lô
├── Hai bánh
│   ├── motorcycle     có động cơ
│   └── bicycle        đạp chân
├── person             người thật
└── ignore_region      vùng bỏ qua, không phải vật thể
```

**8 lớp vật thể và 1 vùng `ignore_region`.** Mỗi vật thuộc đúng một lớp.

Hai lớp đã cân nhắc và **không** đưa vào v1.0: `animal` (lô này không có con vật sống nào đi
tự do) và `other_vehicle` (không tìm thấy xe đẩy tay hay xe súc vật kéo trong 40 ảnh). Nếu lô
sau xuất hiện, mở lại ở phiên bản tiếp theo.

### 2.3 Định nghĩa từng lớp

Ranh giới giữa các lớp dựa trên **dấu hiệu đếm được trên ảnh**, không dựa vào số ghế hay
trọng tải, vì hai thứ đó không nhìn thấy được nên hai người gán sẽ đoán khác nhau.

**`car`**

| | |
|---|---|
| Định nghĩa | Xe bốn bánh chở người, khoang hành khách kín và liền khối với thân, không có thùng hàng tách khỏi cabin. |
| Ví dụ | `img_012` xe con bạc bên trái; `img_015` dãy xe trong nhà trưng bày; `img_028` xe đỏ đậu phía sau; `img_040` taxi xanh. |
| Phản ví dụ | `img_001` xe bán tải trắng → `truck`. `img_005` xe thùng bạc → `van`. `img_009` xe khách nhỏ → `bus`. |
| Dễ nhầm với | `van` (xét chiều cao thân và số ô cửa sổ bên), `truck` (xét thùng có tách khỏi cabin không). |

**`van`**

| | |
|---|---|
| Định nghĩa | Xe thùng kín một khối, cao hơn car. Dấu hiệu đếm được: **dưới 4 ô cửa sổ hành khách liên tiếp** một bên và không có cửa lên xuống riêng cho khách. |
| Ví dụ | `img_005` xe thùng bạc biển `51LD-70-65` làm nền cho đám xe máy; `img_016` xe thùng trắng bên phải. |
| Phản ví dụ | `img_009` xe khách nhỏ, đếm được từ 4 ô cửa sổ trở lên → `bus`. `img_007` xe tải thùng Isuzu, thùng tách khỏi cabin → `truck`. |
| Dễ nhầm với | `bus` (đếm ô cửa sổ bên), `truck` (thân sau cabin kín hay hở). |

**`bus`**

| | |
|---|---|
| Định nghĩa | Xe chở khách thân dài: từ **4 ô cửa sổ hành khách liên tiếp** một bên trở lên, hoặc có cửa lên xuống riêng cho khách. Gồm cả xe khách nhỏ loại County. |
| Ví dụ | `img_009` xe khách nhỏ trắng giữa ảnh; `img_037` xe buýt xanh bên trái; `img_040` xe buýt vàng phía xa. |
| Phản ví dụ | Xe thùng kín dưới 4 ô cửa sổ → `van`. |
| Dễ nhầm với | `van`, `truck` thùng kín cỡ lớn. |

**`truck`**

| | |
|---|---|
| Định nghĩa | Xe chở hàng có thùng, khung hoặc bệ chở **tách biệt khỏi cabin lái**. Gồm xe tải thùng, xe ben, đầu kéo, xe chuyên dụng chở xe, và **xe bán tải (pickup)**. |
| Ví dụ | `img_001` bán tải trắng thùng hở; `img_002` đầu kéo Renault chở xe; `img_007` xe tải thùng Isuzu; `img_012` bán tải chở thùng hàng; `img_036` xe tải nhỏ xanh; `img_037` bán tải gắn thùng camper. |
| Phản ví dụ | `img_005` xe thùng kín liền khối → `van`. Xe ba bánh chở hàng → `three_wheeler`. |
| Quyết định đã chốt | Bán tải có cabin giống car nhưng thùng hở tách rời, nên là `truck`. Ranh giới là **thùng tách rời**, không phải kích thước xe. |

**`motorcycle`**

| | |
|---|---|
| Định nghĩa | Xe hai bánh có động cơ: xe số, tay ga, mô tô, xe máy điện. Tính cả xe đang dựng, đang đậu và đang được dắt bộ. |
| Ví dụ | `img_006` dãy xe máy chờ đèn; `img_008` xe máy đậu kín vỉa hè; `img_026` xe máy chở hai người; `img_030` bãi xe máy. |
| Phản ví dụ | `img_003` xe đạp chở hàng → `bicycle`. `img_004` tuk-tuk → `three_wheeler`. |
| Dễ nhầm với | `bicycle` khi ở xa: tìm động cơ, ống xả, bàn đạp. Không đọc được thì gán lớp gần nhất và bật `can_review`. |

**`three_wheeler`**

| | |
|---|---|
| Định nghĩa | Xe **ba bánh** chở người hoặc chở hàng, có động cơ hoặc đạp chân: tuk-tuk, auto-rickshaw, xe lam, xe ba gác, xích lô. |
| Ví dụ | `img_004` hai tuk-tuk xanh; `img_016` tuk-tuk xanh lá và tuk-tuk bạc; `img_023` xích lô chở khách; `img_035` hai auto-rickshaw vàng, một chiếc chất hàng cao quá mui; `img_040` tuk-tuk phía sau taxi. |
| Phản ví dụ | Xe máy kéo rơ-moóc rời → `motorcycle`. Xe đẩy tay ba bánh không chở người → không gán ở v1.0. |
| Dễ nhầm với | `motorcycle`, `car`. **Đếm bánh nhìn thấy và xét cách người ngồi**, không đoán theo kích thước. |

**`bicycle`**

| | |
|---|---|
| Định nghĩa | Xe đạp hai bánh, không động cơ, có bàn đạp và xích. Tính cả xe đạp thồ và xe đang dắt. |
| Ví dụ | `img_003` xe đạp của người bán hàng rong; `img_006` xe đạp bên phải hàng xe máy; `img_029` hai xe đạp. |
| Phản ví dụ | Xe máy điện không bàn đạp → `motorcycle`. Xích lô ba bánh → `three_wheeler`. |

**`person`**

| | |
|---|---|
| Định nghĩa | **Người thật**, đang đi bộ, đứng, ngồi ngoài khoang kín, hoặc điều khiển xe hai và ba bánh. |
| Ví dụ | `img_001` khách ngồi quán; `img_014` hai người qua đường; `img_034` người đi bộ áo trắng; `img_040` người đội mũ bảo hiểm. |
| Phản ví dụ | Người ngồi **trong khoang kín** của car, van, bus, truck: không gán. Người in trên áp phích `img_018`, người vẽ trên tường `img_026`, người trong ảnh in `img_033`: không gán (R-10). |

**`ignore_region`** — không phải vật thể. Xem R-09.

### 2.4 Thuộc tính

| Thuộc tính | Kiểu | Mức | Nghĩa |
|---|---|---|---|
| `occluded_level` | select | `0` | Thấy rõ toàn bộ, không bị vật nào che |
| | | `1` | Bị che **dưới 50%** thân vật |
| | | `2` | Bị che **từ 50%** trở lên |
| | | `3` | Không xác định được mức che (quá tối, quá mờ) |
| `truncated` | select | `no` / `yes` | Vật bị **mép ảnh** cắt |
| `load_attached` | checkbox | true / false | Box có bao cả hàng hoá gắn liền với xe |
| `can_review` | checkbox | true / false | Ca mơ hồ, cần người chốt. Mặc định `false` |

Ba quy ước bắt buộc, vì đây là nguồn lỗi thuộc tính lớn nhất:

1. **Bị che và bị cắt là hai chuyện khác nhau.** Vật bị cột điện che: `occluded_level = 1`,
   `truncated = no`. Vật chạm mép ảnh: `truncated = yes`; **mép ảnh không phải vật che**, nên
   `occluded_level` vẫn tính theo các vật che thật.
2. **Mọi thứ đều có thể là vật che:** xe khác, người, cây, hàng rào, cột, nước, hàng hoá.
3. Mốc 50% là **ước lượng theo diện tích thân vật khi nguyên vẹn**, không cần đo. Lưỡng lự
   giữa hai mức thì chọn mức **cao hơn** và bật `can_review`.

---

## 3. Quy tắc vẽ box

**R-01 — Box ôm sát phần nhìn thấy.** Bốn mép box chạm đúng điểm ngoài cùng của phần vật thể
đang nhìn thấy. Không chừa lề, không bao khoảng trống.

**R-02 — Chỉ bao phần thấy, không đoán phần khuất.** Vật bị che một phần: box ôm phần thấy,
không nới ra để đoán hình dáng phần bị che. Ghi `occluded_level` tương ứng.

**R-03 — Sai số cho phép ±2 px** ở độ phân giải gốc của ảnh.

**R-04 — Một vật, một box, một lớp.** Không gộp hai xe vào một box, không vẽ hai box cho cùng
một vật. Trước khi lưu, soát các box chồng nhau trên **80% diện tích**: nếu là cùng một vật thì
giữ box ôm trọn, xoá box kia. Ngoại lệ duy nhất là vùng `ignore_region` ở R-09.

**R-05 — Bị mép ảnh cắt.** Box dừng tại mép ảnh, `truncated = yes`. Không kéo box ra ngoài khung.

**R-06 — Gương, ăng-ten, dây và cờ không vào box.** Thùng hàng, giá chở hàng và mui xe thì tính.

**R-07 — Hàng hoá gắn liền nằm trong box xe.** Hàng buộc chặt trên xe là một phần vật thể: một
box duy nhất ôm cả xe và hàng, `load_attached = true`. Khối hàng chiếm chỗ thật trên đường nên
xe tự lái phải tránh cả khối đó. Hàng đã dỡ xuống đất, không dính vào xe: không gán.

**R-08 — Người và xe là hai vật riêng.** Mỗi người điều khiển hoặc ngồi ngoài thân xe được một
box `person` riêng, ôm phần cơ thể nhìn thấy, **kể cả khi box người nằm gần trọn trong box xe**.
Người ngồi trong khoang kín: không gán. Mũ bảo hiểm và nón tính vào box người.

**R-09 — Vật quá nhỏ: dùng `ignore_region`.** Vật có **chiều cao box dưới 20 px** ở độ phân giải
gốc thì không gán riêng; vẽ một hình chữ nhật `ignore_region` phủ cả cụm. Bỏ sót bên trong vùng
`ignore_region` không tính là lỗi QC. Vùng `ignore_region` **không được chồng lên một vật đã có
box**.

**R-10 — Không gán vật ảo.** Không gán xe và người trong: ảnh phản chiếu trên kính, gương chiếu
hậu hay mặt nước; hình in trên áp phích, biển quảng cáo, tranh tường, ảnh treo; và phần thân xe
chủ (ego) lọt vào khung. Ví dụ đã chốt: `img_018` người trên biển quảng cáo, `img_020` toàn bộ
cảnh trong gương chiếu hậu, `img_026` tàu và cần cẩu vẽ trên tường, `img_033` người trong ảnh in.

**R-11 — Vật trong nhà.** Xe nằm hẳn bên trong nhà, trong phòng trưng bày hoặc sau cửa cuốn
đóng: **vẫn gán** nếu nhìn thấy rõ, vì lô này có ảnh chụp trong nhà và mô hình cần nhận ra xe ở
mọi nền. Chỉ không gán khi vật không còn nhận ra được hình dáng.

**R-12 — Xe đậu vẫn phải gán.** Xe dựng, xe đậu, xe đang được dắt bộ đều là vật cản thật.

**R-13 — Vật chồng nhau.** Hai xe chồng nhau thì vẽ hai box riêng, mỗi box ôm phần nhìn thấy;
box được phép giao nhau. Không vì chồng lấn mà bỏ vật phía sau.

**R-14 — Không đoán bừa, và không bỏ im lặng.** Không đọc được lớp hoặc gặp tình huống guideline
chưa nói tới: gán lớp phỏng đoán gần nhất, **bật `can_review`**, và ghi một dòng vào
`submission/qc/image_issues.csv` nếu đến mức không vẽ nổi box. Không tự đặt lớp mới, không im
lặng bỏ qua vật.

**Thứ tự thao tác trong CVAT**

1. `N` vẽ box bằng Rectangle, bấm hai góc đối diện.
2. `Ctrl + 1…9` đổi lớp vật đang chọn.
3. Chọn `occluded_level` trong thẻ đối tượng. **Không dùng phím `Q`** — cờ `occluded` sẵn của
   CVAT là thứ khác và script bỏ qua nó.
4. Quét ảnh lần hai từ xa đến gần để bắt vật bỏ sót, rồi `Ctrl + S`.

---

## 4. Ca khó đã chốt trong Pha 2

| Ảnh | Tình huống | Rule | Quyết định | Lý do |
|---|---|---|---|---|
| `img_017` | Tượng đồng người cưỡi ngựa, đội nón giao thông | R-10 | Không gán người, không gán ngựa | Tượng không di chuyển và không phải người thật |
| `img_018` | Người in trên biển quảng cáo phía trên bãi xe | R-10 | Không gán người | Hình in |
| `img_020` | Toàn cảnh đường nhìn qua gương chiếu hậu ướt | R-10 | Không gán vật nào trong gương | Ảnh phản chiếu |
| `img_026` | Tàu, cần cẩu vẽ trên tranh tường sau xe máy | R-10 | Chỉ gán xe máy và hai người thật | Tranh tường là hình in |
| `img_033` | Ảnh in đen trắng chụp người, treo cạnh ma-nơ-canh | R-10 | Không gán người trong ảnh in | Hình in |
| `img_015` | Xe trong phòng trưng bày, sàn ướt phản chiếu rõ | R-10, R-11 | Gán xe thật, **không** gán ảnh phản chiếu dưới sàn | Xe thật vẫn là vật thể; bóng dưới sàn thì không |
| `img_031` | Xe chạy đêm, đèn hắt bóng dài trên mặt đường ướt | R-10 | Gán xe, không gán bóng | Như trên |
| `img_005` | Người đội mũ bảo hiểm che gần kín người phía sau | R-08, R-02 | Mỗi người một box, ôm phần thấy, `occluded_level` 1 hoặc 2 | Người là lớp ưu tiên 1 |
| `img_014` | Xe máy chạy qua, nhòe chuyển động | R-14 | Vẫn gán, `occluded_level = 3` nếu không đọc được lớp, bật `can_review` | Bỏ sót nguy hiểm hơn sai lớp |
| `img_035` | Auto-rickshaw chất hàng cao gấp đôi xe | R-07 | Một box ôm cả xe và hàng, `load_attached = true` | Khối hàng chiếm chỗ thật |
| `img_012` | Bán tải chất thùng các-tông và túi nilon trong thùng xe | R-07 | Một box `truck`, `load_attached = true` | Hàng nằm trong lòng thùng |
| `img_022` | Mô tô kèm thùng bên (sidecar), ảnh trắng đen | R-04 | Một box `motorcycle` ôm cả thùng bên | Thùng bên gắn cứng, không tách rời |
| `img_009`, `img_039` | Xe máy ở cuối phố, cao dưới 20 px | R-09 | Vùng `ignore_region` phủ cụm xa | Dưới ngưỡng 20 px |

### 4.1 Ba câu hỏi Pha 2 chưa trả lời xong

Nhóm hết giờ họp trước khi chốt được ba tình huống dưới đây. Theo R-14, người gán **tự chọn cách
hiểu của mình, bật `can_review`**, và nhóm sẽ chốt ở Pha 5 dựa trên số đo bất đồng thật.

| Mã | Câu hỏi để ngỏ | Ảnh liên quan |
|---|---|---|
| **H-A** | Xe nằm **trên** một xe khác (xe chở xe, xe đặt trên nóc xe) có được box riêng không, hay chỉ là hàng hoá của xe chở theo R-07? | `img_002`, `img_024`, `img_037` |
| **H-B** | Cụm xe **dày đặc nhưng từng chiếc vẫn cao hơn 20 px** có được phủ `ignore_region` cho nhanh không? | `img_008`, `img_021`, `img_030` |
| **H-C** | Vật **ba chiều, kích thước thật, nhưng không phải xe hoạt động hay người sống** (tượng xe, ma-nơ-canh, xe đồ chơi, xe hóa trang) xử lý thế nào? R-10 chỉ nói tới hình in và phản chiếu. | `img_019`, `img_027`, `img_028`, `img_033`, `img_038` |

---

## 5. QC và phiên bản

### 5.1 Năm loại lỗi dùng khi duyệt

| `error_type` | Nghĩa | Ví dụ trong lô |
|---|---|---|
| `bo_sot` | Vật thật không có box | Bỏ xe đạp lẫn trong hàng xe máy |
| `thua` | Box trên vật không được gán | Box trên tượng, trên hình in, trên bóng phản chiếu |
| `sai_lop` | Sai lớp | Bán tải gán `car` thay vì `truck` |
| `sai_vi_tri_kich_thuoc` | Box lỏng, lệch, thiếu một phần vật | Box xe tải không bao hết thùng |
| `sai_thuoc_tinh` | Sai mức thuộc tính | `occluded_level = 0` cho xe bị che nửa thân |

### 5.2 Ngưỡng nghiệm thu (lấy từ `taxonomy.json`, mục `rules`, không được sửa)

| Đại lượng | Ngưỡng |
|---|---|
| IoU giữa hai người gán, vật cao từ 50 px | từ 0.85 |
| IoU giữa hai người gán, vật cao 20 đến 50 px | từ 0.70 |
| IoU tối thiểu để ghép hai box thành một cặp | 0.50 |
| IAA: IoU trung bình trên các cặp đã ghép | từ 0.85 |
| IAA: tỉ lệ cùng lớp trên các cặp đã ghép | từ 95% |
| Tỉ lệ ảnh có lỗi khi duyệt | tối đa 2% |
| Chiều cao tối thiểu của một box riêng | 20 px |

**Cách đọc hai chỉ số IAA.** IoU trung bình và tỉ lệ cùng lớp **chỉ tính trên những cặp box ghép
được**. Vật mà một người vẽ còn người kia bỏ sót **không nằm trong mẫu số đó** — nó được đếm
riêng ở cột "chưa ghép". Vì vậy hai chỉ số này có thể đều ĐẠT trong khi lô vẫn còn bỏ sót nặng.
Luôn đọc kèm số box chưa ghép.

### 5.3 Ai quyết ca mơ hồ

Box có `can_review = true` và mọi ca trong `calibration_candidates.csv` được chốt ở buổi
calibration Pha 5. Mỗi ca **một dòng** trong `calibration_decisions.csv`, ghi rõ người chốt,
quy tắc được thêm hoặc sửa, phạm vi ảnh hay cả lô, và kết quả kiểm lại. Không ai tự sửa nhãn
của người khác ngoài quy trình này.

### 5.4 Quyết định bàn giao

Theo cây `Hold → Rework → Release`: thiếu bằng chứng hoặc guideline còn ca chưa chốt thì
**Hold**; quá 2% ảnh có lỗi hoặc có lỗi lặp cùng kiểu thì **Rework**; đạt ngưỡng và người kiểm
xác nhận thì **Release**.

### 5.5 Việc còn để ngỏ ở v1.0

Ba câu hỏi H-A, H-B, H-C ở mục 4.1. Nhóm biết đây là lỗ hổng khi khóa v1.0 và chấp nhận đánh
đổi: khóa đúng giờ để ba người gán trên **cùng một bản**, rồi dùng chính số đo bất đồng để chốt,
thay vì tranh luận tiếp mà không có dữ liệu.

### 5.6 Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản giảng viên phát: 5 lớp, 3 quy tắc, không có ca khó | Khởi đầu |
| **v1.0** | 8 lớp và `ignore_region`; thêm `three_wheeler`, `bus`, `bicycle`; thêm thuộc tính `load_attached`; R-01 đến R-14; 13 ca khó đã chốt; 3 câu hỏi để ngỏ | Hợp nhất ba bản cá nhân sau Pha 2 |

---

## 6. Năm bản cá nhân khác nhau ở đâu

| Điểm | 209 | 234 | 123 | 345 | 777 | Nhóm chốt |
|---|---|---|---|---|---|---|
| Xe ba bánh | Lớp riêng | Gộp vào `motorcycle` | Tách `tuktuk` và `cyclo` | Lớp riêng | Lớp riêng | **Một lớp `three_wheeler`**: lô có bảy chiếc, chúng chiếm chỗ như ô tô nhỏ; tách hai lớp thì mỗi lớp quá ít mẫu |
| Bán tải | `truck` | `car` | `truck` | `truck` | `truck` | **`truck`**, theo dấu hiệu thùng tách khỏi cabin, đếm được trên ảnh |
| Người lái xe máy | Box `person` riêng | Không gán, coi là bộ phận của xe | Box riêng khi thấy đầu và thân | Box riêng | Box riêng | **Luôn có box `person` riêng** (R-08), vì người là lớp ưu tiên 1 |
| Ranh giới `van` và `bus` | Số chỗ ngồi | Số ô cửa sổ bên | Chiều dài thân | Số ô cửa sổ bên | Số ô cửa sổ bên | **Số ô cửa sổ bên**, vì số chỗ và chiều dài không đo được trên ảnh |
| Thuộc tính cho hàng hoá | Không đề xuất | Không đề xuất | Không đề xuất | Không đề xuất | Thêm `load_attached` | **Nhận**, để lọc riêng các box bị phình to vì hàng |
| Ngưỡng `occluded_level` | Mô tả bằng lời | Theo phần trăm diện tích | Mô tả bằng lời | Theo phần trăm diện tích, viết thành bảng | Mô tả bằng lời | **Theo phần trăm diện tích**, lấy nguyên bảng của 345 |
| Vật không thật | Không gán tất cả | Gán nếu là khối ba chiều kích thước thật | Không gán tượng, phân vân xe hóa trang | Không gán | Không gán | **Để ngỏ (H-C)** — hết giờ họp |
