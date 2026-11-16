# Guideline gán nhãn — v1.0 — Bản thống nhất nhóm

> Bản thống nhất sau khi đối chiếu các bản cá nhân Pha 1.
> Mọi thành viên tuân thủ đúng guideline này khi gán nhãn trên CVAT (Pha 3).

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết / duyệt: Cả nhóm thống nhất

---

## 1. Mục tiêu

Nhãn trong bộ dữ liệu này dùng để huấn luyện **mô hình phát hiện vật thể 2D (2D object detection)** trên ảnh đường phố, phục vụ ứng dụng camera giao thông hoặc hệ thống hỗ trợ lái xe. Mô hình cần nhận diện **người thật** và **phương tiện giao thông thật đang tham gia hoặc hiện diện trên đường** bằng bounding box hình chữ nhật.

**Sai ở đâu thì nguy hiểm:**
- Bỏ sót người hoặc phương tiện trên đường → mô hình không cảnh báo va chạm.
- Gán nhầm vật không thật (tượng, phản chiếu, xe đồ chơi) thành vật thật → mô hình phanh/né giả.
- Sai lớp (ví dụ `car` thành `truck`) → ảnh hưởng logic xử lý theo kích thước phương tiện.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan, SUV, hatchback — xe 4 bánh chở người có chiều cao thùng xe thấp, không có khoang hàng riêng biệt phía sau. | Sedan (`img_003.jpg`), SUV (`img_007.jpg`), ô tô con trên sàn xe chở xe (`img_002.jpg`) | Xe van kín thùng → `van`; xe bán tải → `truck`; xe mô hình đồ chơi (`img_038.jpg`) → **KHÔNG gán** |
| `truck` | Xe tải có khoang hàng/thùng hàng tách biệt khỏi cabin, hoặc xe bán tải (pickup) có thùng mở phía sau. | Xe tải nhỏ (`img_001.jpg`), xe chở ô tô (`img_002.jpg` — gán truck cho xe đầu kéo + sàn), xe bán tải (`img_037.jpg`) | Xe van kín → `van`; tượng xe bán tải (`img_027.jpg`) → **KHÔNG gán** |
| `van` | Xe thùng kín cỡ nhỏ-vừa, hình hộp, không tách rõ cabin và thùng hàng, thường dùng chở hàng hoặc chở khách nhỏ (≤ 16 chỗ). | Xe van trắng (`img_016.jpg` — xe van kín phía sau tuk-tuk) | Xe buýt lớn (> 16 chỗ) → `bus`; bán tải gắn nắp thùng kín cao → xem R-09 |
| `bus` | Xe buýt, xe khách cỡ lớn (thường > 16 chỗ), thân dài, nhiều cửa sổ hành khách. | Xe buýt trắng (`img_009.jpg`), xe buýt xanh GO Transit (`img_037.jpg`), xe buýt trên đường (`img_031.jpg`, `img_040.jpg`) | Xe van nhỏ ≤ 16 chỗ → `van` |
| `tuk_tuk` | Xe ba bánh có động cơ, thường dùng chở khách hoặc hàng, phổ biến ở Đông Nam Á và Nam Á. | Tuk-tuk xanh (`img_004.jpg`), tuk-tuk vàng chở hàng (`img_035.jpg`), tuk-tuk xanh/đen (`img_016.jpg`) | Xích lô đạp (không có động cơ) → **KHÔNG gán** (xem R-05) |
| `motorcycle` | Xe máy, mô tô, xe scooter — xe hai bánh có động cơ. | Xe máy (`img_006.jpg`), scooter xanh (`img_016.jpg`), xe máy chở ống (`img_029.jpg`) | Xe đạp (không động cơ) → **KHÔNG gán** (xem R-05); xe mô tô điêu khắc kim loại (`img_011.jpg`) → **KHÔNG gán** (xem R-04) |
| `person` | Người thật, đang sống, nhìn thấy trong ảnh — bao gồm người đi bộ, người ngồi/đứng trên phương tiện, người bán hàng. | Người đi bộ (`img_006.jpg`), người ngồi trên xe máy (`img_006.jpg`, `img_029.jpg`), người đứng cạnh xe tải (`img_002.jpg`) | Tượng người (`img_017.jpg`) → **KHÔNG gán**; ma-nơ-canh (`img_033.jpg`) → **KHÔNG gán**; người trên biển quảng cáo (`img_018.jpg`) → **KHÔNG gán**; phản chiếu người trong gương (`img_020.jpg`) → **KHÔNG gán** |
| `ignore_region` | Vùng bỏ qua — đánh dấu khu vực không gán từng vật riêng lẻ: đám đông quá đông không tách được cá thể, hoặc cụm phương tiện/người quá nhỏ (cao < 20 px) tập trung dày đặc. | Đám đông phía xa (`img_006.jpg` — nhóm người + xe ở xa), cụm xe máy nhỏ phía xa (`img_011.jpg`) | Không dùng ignore_region để che vật gán được riêng |

**Vật không thuộc lớp nào** (xe đạp, xích lô đạp, xe ngựa, động vật, cây cối, cột đèn, biển báo...): **KHÔNG gán box**, trừ khi chúng tạo thành đám đông cùng các vật thuộc lớp → dùng `ignore_region`.

### 2.2. Thuộc tính

Mỗi box (trừ `ignore_region`) bắt buộc có 3 thuộc tính sau:

| Thuộc tính | Kiểu | Các mức | Ngưỡng cụ thể | Mặc định |
|---|---|---|---|---|
| `occluded_level` | select | `0`, `1`, `2`, `3` | **0**: Vật thấy rõ hoàn toàn (0% bị che). **1**: Bị che 1–30% diện tích nhìn thấy (ví dụ: xe bị cột đèn che một phần nhỏ). **2**: Bị che 31–70% diện tích (ví dụ: người bị xe che nửa thân). **3**: Bị che > 70% nhưng vẫn nhận dạng chắc chắn là đối tượng thuộc lớp đó (ví dụ: chỉ thấy đầu xe nhưng chắc chắn là car). | `0` |
| `truncated` | select | `no`, `yes` | **no**: Vật nằm hoàn toàn trong khung ảnh. **yes**: Vật bị cắt bởi mép ảnh (trên, dưới, trái, phải) — nghĩa là nếu ảnh rộng hơn thì sẽ thấy thêm phần của vật. | `no` |
| `can_review` | checkbox | tick/không | Tick khi bạn **không chắc chắn** về lớp, ranh giới box, hoặc thuộc tính và muốn nhóm xem lại ở Pha 5. | `false` |

---

## 3. Quy tắc vẽ box

### R-01 — Box ôm sát phần nhìn thấy được

- Vẽ box hình chữ nhật bao **chặt nhất có thể** quanh phần **nhìn thấy** của vật thể.
- Nếu vật bị che bởi vật khác: box chỉ ôm phần nhìn thấy, **KHÔNG** kéo box ra bao phần bị che.
- Nếu vật bị cắt bởi mép ảnh: box kéo đến **sát mép ảnh** tại chỗ bị cắt, đặt `truncated = yes`.

**Ví dụ:** `img_001.jpg` — xe tải bị cắt mép phải ảnh → box kéo đến mép phải, `truncated = yes`.

### R-02 — Người đi trên xe máy / xe đạp

Khi có **người ngồi/đứng trên xe máy**:
- Vẽ **1 box `motorcycle`** ôm sát phần nhìn thấy của xe máy (bao gồm bánh xe, thân xe, tay lái).
- Vẽ **1 box `person`** cho **mỗi người** trên xe, ôm sát phần thấy được của từng người (từ đỉnh đầu/mũ bảo hiểm đến phần thấp nhất nhìn thấy — thường là đầu gối hoặc bàn chân).
- Các box `person` và `motorcycle` sẽ **chồng lấn** nhau — điều này là bình thường và đúng quy tắc.
- Nếu xe có 2 người (người lái + người ngồi sau): vẽ **2 box `person`** riêng biệt + **1 box `motorcycle`**.

**Ví dụ:** `img_006.jpg` — cặp đôi trên xe máy (người mặc áo chấm bi vàng + người mặc áo đỏ): 1 box `motorcycle` + 2 box `person`.

**Ví dụ:** `img_029.jpg` — người ngồi trên xe máy chở ống sắt: 1 box `motorcycle` + 1 box `person`.

> **Lưu ý:** Quy tắc tương tự áp dụng cho người ngồi trên tuk-tuk nếu **nhìn thấy rõ cá thể**. Nếu người ngồi trong tuk-tuk bị thùng xe che gần hết (occluded > 70% và không nhận dạng chắc chắn) → KHÔNG gán box person.

### R-03 — Xe chở xe (car carrier / transporter)

Khi gặp xe tải chở nhiều ô tô trên sàn (như `img_002.jpg`):
- Vẽ **1 box `truck`** cho toàn bộ xe đầu kéo + khung sàn chở.
- Vẽ **1 box `car`** riêng cho **mỗi ô tô con** nhìn thấy trên sàn xe, kể cả khi chúng chồng lên box `truck`.
- Mỗi ô tô con phải đủ nhận dạng (thấy ít nhất phần đầu hoặc đuôi xe). Nếu ô tô bị che > 70% bởi xe khác trên sàn nhưng vẫn nhận dạng chắc chắn → gán `occluded_level = 3`.

**Ví dụ:** `img_002.jpg` — xe tải chở ô tô: 1 box `truck` (bao cả sàn) + 8 box `car` (mỗi xe con trên 2 tầng).

### R-04 — Vật không thật: tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu

**KHÔNG gán box** cho các vật sau (đây là phản ví dụ — trông giống vật thật nhưng không phải):

| Loại | Ảnh minh họa | Lý do loại |
|---|---|---|
| Tượng người / tượng cưỡi ngựa | `img_017.jpg` (tượng đồng cưỡi ngựa) | Không phải người/phương tiện thật |
| Tượng xe / xe điêu khắc | `img_027.jpg` (tượng xe bán tải đá đen), `img_011.jpg` (xe mô tô kim loại điêu khắc) | Không phải phương tiện thật |
| Ma-nơ-canh | `img_033.jpg` (ma-nơ-canh trong tủ kính) | Không phải người thật |
| Xe mô hình, đồ chơi | `img_038.jpg` (xe mô hình trong tủ kính) | Không phải phương tiện thật |
| Hình người/xe trên biển quảng cáo | `img_018.jpg` (chân dung trên biển quảng cáo) | Hình 2D in, không phải vật thật |
| Phản chiếu qua gương/kính | `img_020.jpg` (phản chiếu trong gương chiếu hậu) | Hình ảnh phản chiếu, không phải vật thật |

**Nguyên tắc chung:** Chỉ gán box cho vật **ba chiều, thật, tồn tại vật lý tại hiện trường** (không phải bản sao, hình ảnh, hay mô hình). Ảnh chỉ chứa vật không thật → **0 annotation** (vẫn giữ ảnh trong COCO).

### R-05 — Phương tiện không có động cơ

**KHÔNG gán box** cho:
- Xe đạp (bicycle) — `img_006.jpg` phía xa, `img_029.jpg` (xe đạp bên phải), `img_039.jpg` (xe đạp dựng bên đường)
- Xích lô đạp — `img_039.jpg`
- Xe ngựa, xe kéo tay

**Lý do:** Mô hình phát hiện vật 2D này tập trung vào **phương tiện có động cơ** và **người**. Xe đạp không nằm trong phạm vi phát hiện của bài lab.

> **Ngoại lệ:** Nếu **người** đang đi/cưỡi xe đạp và người đó nhìn thấy rõ → vẫn gán box `person` cho người đó (nhưng KHÔNG gán box cho xe đạp).

### R-06 — Vật quá nhỏ (chiều cao < 20 px)

- Vật có **chiều cao bounding box < 20 pixel** trong ảnh gốc → **KHÔNG gán box riêng**.
- Nếu có **cụm nhiều vật nhỏ dày đặc** (đám đông xa, hàng xe nhỏ phía chân trời) → dùng 1 box `ignore_region` bao vùng đó.
- Nếu vật chỉ hơi nhỏ (20–30 px) nhưng vẫn nhận dạng được → vẫn gán box bình thường, tick `can_review` nếu không chắc.

### R-07 — Hàng hóa quá khổ trên xe

Khi phương tiện chở hàng hóa vượt khỏi thùng xe (như bao tải chất đống trên nóc tuk-tuk ở `img_035.jpg`):
- Box ôm sát **phần thân xe (khung xe, cabin, bánh xe)**, **KHÔNG bao hàng hóa quá khổ** phía trên.
- Lý do: mô hình cần nhận diện phương tiện, không cần nhận diện hàng hóa. Box quá rộng sẽ sai lệch tỉ lệ kích thước.

**Ví dụ:** `img_035.jpg` — tuk-tuk chở bao tải: box chỉ ôm thân tuk-tuk (cabin + sàn chở), KHÔNG kéo box lên bao gồm đống bao tải trên nóc.

### R-08 — Xe chủ (ego vehicle)

- Nếu ảnh chụp từ camera gắn trên xe chủ và **thấy một phần thân vỏ xe chủ** (như nắp ca-pô, gương chiếu hậu, cửa xe): **KHÔNG gán box** cho phần thuộc xe chủ.
- Lý do: xe chủ không phải đối tượng cần phát hiện — mô hình đặt trên xe chủ không cần phát hiện chính nó.

**Ví dụ:** `img_020.jpg` — gương chiếu hậu là bộ phận xe chủ → không gán box cho gương; ảnh chỉ gồm gương + phản chiếu → **0 annotation**.

### R-09 — Bán tải gắn nắp thùng kín cao (canopy / camper shell)

Xe bán tải (pickup) có **nắp thùng cứng cao** biến thùng xe thành khoang kín (ví dụ: `img_037.jpg` — xe trắng có nắp thùng vuông phía sau):
- Vẫn gán lớp **`truck`** (vì bản chất là xe bán tải có thùng).
- **KHÔNG gán `van`** — vì nhìn rõ cấu trúc cabin + thùng tách biệt đặc trưng của pickup.

**Ví dụ:** `img_037.jpg` — bán tải trắng 4x4 gắn nắp thùng kín cao → `truck`.

### R-10 — Motion blur và điều kiện ảnh kém

Khi vật bị nhòe do chuyển động (motion blur) hoặc ảnh tối/nhiễu:
- Nếu **vẫn nhận dạng được** lớp và ranh giới → gán box bình thường.
- Nếu **không nhận dạng được** lớp hoặc ranh giới (không biết đó là car hay motorcycle, không biết vật bắt đầu/kết thúc ở đâu) → **KHÔNG gán box**, tick `can_review` nếu lưỡng lự.

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_001.jpg` | Xe tải bị cắt mép phải ảnh, nhiều người ngồi trong quán | R-01 | Gán `truck` với `truncated=yes`; gán `person` cho từng người thấy rõ, `occluded_level` tùy mức che | Box kéo sát mép ảnh; người bị che bởi bàn ghế gán occluded tương ứng |
| `img_002.jpg` | Xe tải chở 8 ô tô con trên 2 tầng sàn | R-03 | 1 box `truck` + 8 box `car` | Xe đầu kéo + sàn = truck; mỗi ô tô con trên sàn = 1 car riêng |
| `img_004.jpg` | Tuk-tuk 3 bánh, không có trong v0 | — | Thêm lớp `tuk_tuk` vào taxonomy | Xe ba bánh có động cơ, cần phân biệt với car/motorcycle |
| `img_006.jpg` | Đường đông đúc: xe máy, xe đạp, người đi bộ lẫn lộn | R-02, R-05, R-06 | Gán từng motorcycle + person; KHÔNG gán xe đạp; ignore_region cho cụm xa < 20 px | Xe đạp ngoài phạm vi; vật quá nhỏ dùng ignore |
| `img_009.jpg` | Xe buýt cỡ nhỏ-vừa, không có trong v0 | — | Thêm lớp `bus`; xe buýt này gán `bus` | Thân dài, cửa kính hành khách, > 16 chỗ |
| `img_011.jpg` | Tượng/điêu khắc xe mô tô kim loại và tượng đầu người lớn | R-04 | KHÔNG gán box cho tượng/điêu khắc; gán person cho người thật phía xa nếu đủ lớn | Vật không thật |
| `img_016.jpg` | Tuk-tuk + scooter + van + tuk-tuk xanh, đường phố Thái Lan | R-02 | Gán `tuk_tuk` cho 2 tuk-tuk; `motorcycle` cho scooter; `van` cho xe van trắng; `person` cho từng người thấy rõ | Áp dụng đúng định nghĩa lớp mới |
| `img_017.jpg` | Tượng đồng người cưỡi ngựa | R-04 | **0 annotation** — KHÔNG gán box | Tượng đồng, không phải người/phương tiện thật |
| `img_018.jpg` | Chân dung người trên biển quảng cáo, xe đỗ phía sau hàng rào | R-04 | KHÔNG gán person cho hình trên biển; gán car cho xe thật phía sau nếu nhìn rõ | Hình in ≠ người thật |
| `img_020.jpg` | Gương chiếu hậu chụp cận, phản chiếu đường + xe | R-04, R-08 | **0 annotation** — gương = xe chủ, phản chiếu = không thật | Không gán vật trong gương lẫn bộ phận xe chủ |
| `img_027.jpg` | Tượng xe bán tải đá đen trong nghĩa trang | R-04 | **0 annotation** — KHÔNG gán box | Tượng/mô hình, không phải xe thật |
| `img_029.jpg` | Người trên xe máy chở ống sắt dài + xe đạp bên cạnh | R-02, R-05 | 1 `motorcycle` + 1 `person` cho người trên xe máy; KHÔNG gán xe đạp; 1 `person` cho người đi xe đạp | Xe đạp ngoài phạm vi nhưng người đi xe đạp là người thật |
| `img_031.jpg` | Xe buýt trên đường | — | Gán `bus` | Áp dụng lớp mới |
| `img_033.jpg` | Ma-nơ-canh trong tủ kính cửa hàng | R-04 | **0 annotation** — KHÔNG gán box cho ma-nơ-canh | Ma-nơ-canh ≠ người thật |
| `img_035.jpg` | Tuk-tuk chở bao tải chất đống quá khổ | R-07 | Box `tuk_tuk` chỉ ôm thân xe, KHÔNG bao hàng hóa quá khổ | Mô hình cần nhận diện xe, không nhận diện hàng hóa |
| `img_037.jpg` | Bán tải trắng gắn nắp thùng kín cao + xe buýt GO | R-09 | Bán tải → `truck`; xe buýt → `bus` | Pickup + canopy vẫn là truck; xe buýt cỡ lớn = bus |
| `img_038.jpg` | Tủ kính trưng bày xe mô hình đồ chơi | R-04 | **0 annotation** — KHÔNG gán box | Xe mô hình đồ chơi ≠ phương tiện thật |
| `img_039.jpg` | Đường phố Huế: xe máy, xe đạp, xích lô, người đi bộ | R-02, R-05 | Gán motorcycle + person; KHÔNG gán xe đạp/xích lô; gán person cho người đi xe đạp/xích lô | Xe đạp/xích lô ngoài phạm vi; người là người thật |
| `img_040.jpg` | Xe buýt trên đường | — | Gán `bus` | Áp dụng lớp mới |

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC áp dụng (từ `taxonomy.json` — rules)

| Đại lượng | Ngưỡng |
|---|---|
| Chiều cao box tối thiểu | 20 px |
| IoU box lớn (cao ≥ 50 px) | ≥ 0.85 |
| IoU box nhỏ (cao < 50 px) | ≥ 0.70 |
| IAA — IoU trung bình | ≥ 0.85 |
| IAA — Đồng thuận lớp | ≥ 95% |
| Tỉ lệ ảnh lỗi tối đa để Release | ≤ 2% |
| IoU ghép cặp box (match) | ≥ 0.50 |

### 5.2. Ai quyết ca mơ hồ

- **Trong lúc gán (Pha 3):** Cá nhân tự quyết theo guideline v1.0 này. Nếu lưỡng lự → tick `can_review` trên box hoặc ghi vào `image_issues.csv`.
- **Sau khi đo IAA (Pha 5):** Cả nhóm họp calibration, chốt quyết định vào `calibration_decisions.csv` với mã quy tắc `R-xx`.

### 5.3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu do giảng viên viết. 5 lớp (car, truck, van, motorcycle, person) + ignore_region. Thiếu định nghĩa ngưỡng che khuất, thiếu quy tắc cho vật không thật, xe chở xe, người đi xe máy, vật nhỏ. | — |
| v1.0 | **Thêm 2 lớp mới:** `bus` (img_009, img_031, img_037, img_040) và `tuk_tuk` (img_004, img_016, img_035). **Bổ sung 10 quy tắc R-01 → R-10:** ôm sát phần nhìn thấy, người đi xe máy, xe chở xe, vật không thật, phương tiện không động cơ, vật nhỏ, hàng hóa quá khổ, xe chủ, bán tải nắp thùng cao, motion blur. **Định lượng ngưỡng occluded_level:** 0/1–30%/31–70%/>70%. **Thêm bảng ca khó** với 19+ ca trên ảnh cụ thể. | Đối chiếu bản cá nhân Pha 1: bất đồng về tuk-tuk, bus, xe chở xe, tượng, xe đạp, hàng hóa quá khổ. |
