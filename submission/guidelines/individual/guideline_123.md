# Guideline gan nhan — v0.1 (ban ca nhan reviewer) — 100

- Ngay: 2026-11-16
- Nguoi viet: 123

---

## 1. Muc tieu

Nhan dung de huan luyen mo hinh phat hien **nguoi va phuong tien** tren anh duong pho (camera giao thong, xe tu lai). Mo hinh can phat hien chinh xac vi tri va lop cua vat the thuc, dang tham gia hoac hien dien tren duong. Sai o dau thi nguy hiem: bo sot nguoi di bo gay tai nan; nham tuong/ma no canh voi nguoi that lam nhieu tin hieu.

## 2. Lop va thuoc tinh

### Lop (moi vat thuoc dung mot lop)

| Lop | Dinh nghia | Vi du | Phan vi du / de nham |
|---|---|---|---|
| `car` | O to con 4 banh, ke ca sedan, SUV, hatchback, taxi. Bao gom ca xe co va xe dien neu dang hinh o to con. | img_012 (Mazda sedan), img_015 (xe do tren bai), img_031 (SUV trang), img_034 (xe den) | Pickup co thung mo (xet `truck` neu thung cho hang lon) — xem R-02. Xe dieu hanh / novelty car (img_019) — xem R-08. |
| `truck` | Xe tai co thung cho hang (thung kin, thung mo, xe ben, dau keo ro-mooc). Bao gom pickup co thung mo khi thung > 1/3 chieu dai xe. | img_001 (Suzuki pickup thung mo), img_002 (dau keo cho xe), img_007 (Isuzu thung kin), img_036 (xe tai nho), img_037 (pickup voi camper top) | Van (thung kin nhung cabin va thung lien khoi) — xem R-03. Xe tai nho dang van — xem R-03. |
| `van` | Xe thung kin co nho, cabin va khoang hang lien khoi (khong tach roi). Minibus chua duoc 16 cho tro xuong. | img_005 (xe van phia sau), img_009 (minibus), img_016 (van trang) | Bus lon (> 16 cho) — xem R-04. Pickup co canopy/camper — van R-02. |
| `motorcycle` | Xe may, mo to, scooter (xe ga) 2 banh co dong co. Bao gom ca xe dien 2 banh dang xe may. | img_005 (xe may voi nhieu nguoi), img_006 (nhieu xe may), img_008 (xe may do), img_026 (xe may dem), img_040 (nguoi di xe may) | Xe dap (khong co dong co) — KHONG gan, bo qua — xem R-05. Xe may co sidecar (img_022) — van la `motorcycle`. Tuk-tuk/auto-rickshaw 3 banh — xem R-06. Xe dap dien nho — xem R-05. |
| `person` | Nguoi that, dang song, nhin thay tren anh. Bao gom nguoi di bo, nguoi ngoi, nguoi tren xe. | img_001 (nguoi dung, nguoi ngoi), img_003 (nguoi di bo), img_014 (2 nguoi qua duong), img_032 (dam dong cho bus) | Tuong nguoi (img_017) — KHONG gan — xem R-07. Ma-no-canh (img_033) — KHONG gan — xem R-07. Mascot/bup be (img_019) — KHONG gan — xem R-07. Hinh nguoi tren billboard (img_018) — KHONG gan — xem R-09. |
| `ignore_region` | Vung khong can gan tung vat rieng le: dam dong qua day dac khong tach duoc tung ca the, hoac vung vat qua nho (cao < 20 px) tap trung. | img_021 (vung xa dam dong xe may), img_030 (bai do xe may day dac) | Chi dung khi KHONG THE tach tung vat. Neu tach duoc thi gan tung box. |

**Vat khong thuoc lop nao**: xe dap, xich lo, tuk-tuk/auto-rickshaw, bus lon, xe lua, thuyen — KHONG gan box, KHONG dung `ignore_region` cho tung chiec. Ghi vao `image_issues.csv` de nhom thao luan. Xem chi tiet tung loai o muc Ca kho.

### Thuoc tinh

| Thuoc tinh | Gia tri | Nghia | Nguong cu the |
|---|---|---|---|
| `occluded_level` | `0` | Thay ro toan bo vat, khong bi che | 0% bi che |
| | `1` | Bi che it | 1–30% dien tich vat bi che boi vat khac hoac canh vat |
| | `2` | Bi che nhieu | 31–70% dien tich vat bi che |
| | `3` | Khong ro / gan nhu bi che het | > 70% bi che, chi thay mot phan nho |
| `truncated` | `no` | Vat nam tron trong khung anh | Khong bi cat boi mep anh |
| | `yes` | Vat bi cat boi mep anh (tren/duoi/trai/phai) | Bat ky phan nao cua vat bi cat boi mep anh |
| `can_review` | tick | Tick khi chua chac ve lop, box hoac thuoc tinh | Dung cho moi ca nghi ngo |

## 3. Quy tac ve box

### 3.1 Nguyen tac chung
- **R-00**: Gan box cho **moi** nguoi va phuong tien (thuoc cac lop tren) nhin thay trong anh, tru cac ngoai le duoi day.
- Box om sat **phan nhin thay** cua vat. Khong ve box bao gom ca phan bi che (phan khong thay).
- Moi vat the mot box duy nhat.

### 3.2 Nguoi tren xe may (R-01)
- **Nguoi va xe may la 2 box rieng biet.**
- Xe may: 1 box bao toan bo xe (bao gom ca phan banh, than, guidon nhin thay).
- Moi nguoi tren xe (nguoi lai + nguoi ngoi sau): moi nguoi 1 box `person` rieng. Box chi bao phan co the cua nguoi do, khong bao ca xe.
- Vi du: img_005 — xe may co 2 nguoi → 1 box `motorcycle` + 2 box `person`.
- Vi du: img_006 — xe may co 2 nguoi ngoi → 1 box `motorcycle` + 2 box `person` (nguoi phia sau `occluded_level=1` hoac `2` tuy muc che).

### 3.3 Pickup truck — car hay truck? (R-02)
- Pickup co **thung mo** (flatbed, thung phu bao tai, thung cho hang nhin thay) va thung chiem > 1/3 chieu dai xe → `truck`. Vi du: img_001 (Suzuki thung mo), img_007 (pickup trang giua 2 xe tai).
- Pickup co **canopy kin / camper top** phu kin thung → `truck`. Vi du: img_037 (pickup trang voi thung camper).
- Pickup doi thuong khong co thung cho hang ro rang, nhin nhu SUV → `car`.
- **Test**: co nhin thay khu vuc cho hang rieng biet khong? Co → `truck`. Khong → `car`.

### 3.4 Van hay truck? (R-03)
- Cabin va khoang hang **lien khoi** (khong co moi noi giua dau xe va thung) → `van`. Vi du: img_009 (minibus), img_036 (xe tai nho VeryCA — cabin tach roi thung → `truck`).
- Cabin va thung hang **tach roi** (co moi noi, co the thao thung) → `truck`.
- Xe tai nho nhung cabin-thung tach → `truck` (img_036).

### 3.5 Bus lon (R-04)
- Bus lon (> 16 cho) **khong co lop** trong taxonomy hien tai (v0). 
- Quyet dinh tam: **KHONG gan box**. Ghi vao `image_issues.csv`.
- Anh lien quan: img_037 (bus GO Transit xanh), img_040 (bus vang phia sau).
- **Can hoi reviewer / nhom**: co nen them lop `bus` hoac gan `truck` cho bus?

### 3.6 Xe dap va xe thu cong khong dong co (R-05)
- Xe dap, xich lo, xe day — **KHONG gan box** vi khong co dong co, khong thuoc lop nao.
- Nguoi dap xe: van gan box `person`.
- Vi du: img_003 (xe dap ben le — khong gan xe dap, chi gan `person` cho nguoi), img_023 (xich lo — khong gan xich lo, chi gan `person` cho nguoi dap), img_029 (nguoi dap xe dap — gan `person`, khong gan xe dap).

### 3.7 Tuk-tuk / Auto-rickshaw 3 banh (R-06)
- Tuk-tuk la xe 3 banh co dong co, KHONG phai motorcycle (2 banh) va KHONG phai car/van (4 banh).
- Quyet dinh tam: **KHONG gan box** vi khong co lop phu hop. Ghi vao `image_issues.csv`.
- Anh lien quan: img_004 (2 tuk-tuk), img_016 (tuk-tuk xanh va xanh la), img_035 (auto-rickshaw vang An Do), img_040 (tuk-tuk phia sau).
- **Can hoi reviewer / nhom**: co nen gan `motorcycle` (vi co dong co, goc xe may) hay them lop moi?

### 3.8 Tuong, ma-no-canh, mascot, hinh in (R-07)
- **KHONG gan box** cho bat ky vat khong phai thuc the song / phuong tien that:
  - Tuong nguoi hoac dong vat: img_017 (tuong nguoi cuoi ngua), img_011 (tuong Predator), img_013 (tuong con vat vang).
  - Mo hinh xe bang kim loai / da: img_011 (mo hinh xe may kim loai), img_027 (mo hinh xe tai bang da — bia mo).
  - Ma-no-canh: img_033 (ma-no-canh trong tu kinh).
  - Mascot / bup be: img_019 (mascot chuot tren xe dieu hanh).
  - Tuong ngua: img_028 (ngua trang tren thung xe — la tuong, khong gan `person`).
- **Ly do**: mo hinh khong phat hien duoc giup an toan giao thong; false positive lam nhieu tin hieu.

### 3.9 Xe dieu hanh / xe bien dang (R-08)
- Xe dieu hanh co bien so thuc, di chuyen duoc tren duong → **van gan box** theo hinh dang gan nhat.
- Vi du: img_019 (xe Danger Mouse vang, co bien so YR04 NUM) → gan `car` (vi co 4 banh, hinh dang o to).
- Xe mo hinh khong co dong co / khong di duoc → KHONG gan.

### 3.10 Hinh anh trong guong / phan chieu / billboard / poster (R-09)
- **KHONG gan box** cho vat chi xuat hien qua:
  - Guong chieu hau: img_020 (xe chi thay qua guong xe — khong gan).
  - Phan chieu tren kinh / mat nuoc: img_033 (phan chieu tren cua kinh).
  - Hinh in tren billboard / poster: img_018 (hinh nguoi tren bang quang cao), img_026 (hinh cang bien phia sau).
- **Ly do**: day la hinh 2D cua vat, khong phai vat that 3D tren duong.

### 3.11 Xe do choi / mo hinh thu nho (R-10)
- **KHONG gan box** cho xe do choi, mo hinh thu nho trong tu kinh, tren noc xe, v.v.
- Vi du: img_038 (tu trung bay xe mo hinh), img_024 (xe mo hinh vang tren noc xe den trong bao tang).
- Xe that trong bao tang/trien lam: img_024 (cac xe that trong bao tang) → **gan box** `car` vi la xe that kich thuoc that, du khong di chuyen. **Can hoi reviewer**: xe trong bao tang co can phat hien khong?

### 3.12 Vat bi nham (motion blur) (R-11)
- Xe / nguoi bi nhoe do chuyen dong nhung van **nhan dien duoc** lop → gan box, `occluded_level=0` (vi khong bi che boi vat khac, chi bi nhoe).
- Vi du: img_014 (xe may nhoe phia sau nhung van thay hinh dang), img_015 (xe nhoe giua bai do — nhan ra la car → gan).
- Xe / nguoi nhoe **khong the nhan dien** duoc lop → KHONG gan, ghi `image_issues.csv`.

### 3.13 Vat qua nho (R-12)
- Vat co chieu cao < **20 pixel** trong anh → dung `ignore_region` bao ca vung co nhieu vat nho.
- Neu chi 1-2 vat nho le → KHONG gan, ghi `image_issues.csv`.
- Vi du: img_011 (nguoi va xe o rat xa trong festival), img_013 (xe rat xa trong dem), img_025 (nguoi va xe xa tren duong ven bien).
- Phan biet: vat nho nhung > 20 px → van gan box binh thuong.

### 3.14 Dam dong (R-13)
- Khi **co the tach tung ca the** (nguoi, xe) → gan tung box rieng, du mat nhieu thoi gian.
- Khi **khong the tach** (qua day dac, chong len nhau) → dung 1 box `ignore_region` phu vung day.
- Vi du: img_021 (giao thong TP.HCM — phan gan camera co the tach tung xe may + nguoi; vung xa day dac → `ignore_region`). img_030 (bai do xe may — xe day dac nhung tinh → co the tach duoc, gan tung `motorcycle`). img_032 (dam dong nguoi cho bus — co the tach tung `person` gan camera; phia sau → `ignore_region`).

### 3.15 Vat bi che (occluded) vs. bi cat mep (truncated)
- **Bi che** (`occluded`): vat bi vat khac trong canh che khuat (xe truoc che xe sau, nguoi che nguoi).
- **Bi cat mep** (`truncated`): vat bi cat boi ranh gioi anh.
- Mot vat co the vua bi che VUA bi cat mep: dat ca `occluded_level > 0` va `truncated=yes`.
- Vi du: img_015 (xe do ben trai bi cat mep → `truncated=yes`, `occluded_level=0`). img_001 (nguoi ngoi phia sau bi che boi ban va nguoi khac → `occluded_level=2`, `truncated=no`).

### 3.16 Xe chu (ego vehicle)
- Neu anh chup tu tren xe → xe chu KHONG xuat hien trong anh, khong can gan.
- Neu nhin thay mot phan xe chu (ca-po, guong) → KHONG gan box cho xe chu.
- Vi du: img_020 (guong chieu hau cua xe chu → khong gan xe chu).

### 3.17 Xe tren xe (R-14)
- Xe that dang duoc van chuyen tren xe tai/ro-mooc:
  - Xe tai cho: gan box `truck` cho **xe tai cho** (ca dau keo + ro-mooc).
  - Cac xe tren ro-mooc: **KHONG gan box rieng** cho xe dang duoc cho. Ly do: chung khong dang tham gia giao thong, khong can phat hien rieng.
- Vi du: img_002 (xe tai cho nhieu o to — chi gan 1 box `truck` cho ca xe tai + ro-mooc, KHONG gan tung car tren ro-mooc).
- Nguoi dung canh xe tai (img_002 — nguoi ben phai) → van gan `person`.

### 3.18 Xe dau / khong hoat dong
- Xe dau ben duong, trong bai do → **van gan box** binh thuong. Mo hinh can phat hien ca xe dung yen.
- Vi du: img_008 (xe may dau san), img_012 (xe do tren pho), img_015 (xe do trong bai).

## 4. Ca kho

| Anh | Tinh huong | Rule ID | Quyet dinh | Ly do |
|---|---|---|---|---|
| img_001 | Pickup Suzuki thung mo — truck hay car? | R-02 | `truck` | Thung mo chiem > 1/3 chieu dai, co khung thung |
| img_001 | Nhieu nguoi ngoi ban an bi che | R-00 | Gan `person` voi `occluded_level=2` | Nhin thay dau va mot phan than |
| img_002 | Xe tai cho nhieu o to tren ro-mooc | R-14 | 1 box `truck` cho ca xe tai, KHONG gan xe tren ro-mooc | Xe khong tham gia giao thong |
| img_002 | Nguoi dung ben xe tai | R-00 | Gan `person` | Nguoi that, nhin thay ro |
| img_003 | Xe dap ben le duong | R-05 | Khong gan xe dap | Khong co lop, nguoi dap → gan `person` |
| img_004 | 2 tuk-tuk dau | R-06 | Khong gan box, ghi issue | Khong co lop phu hop. `can_review` |
| img_004 | Nguoi ngoi trong tuk-tuk | R-06 | Gan `person` voi `occluded_level=2-3` | Nguoi that, du ngoi trong xe khong co lop |
| img_005 | Nhieu nguoi tren 1 xe may | R-01 | 1 `motorcycle` + moi nguoi 1 `person` | Tach nguoi va xe |
| img_006 | Dam dong xe may tu phia sau | R-13 | Gan tung box cho xe va nguoi gan camera; vung xa → `ignore_region` | Co the tach phan gan, khong tach duoc phan xa |
| img_007 | Pickup trang giua 2 xe tai | R-02 | `truck` (co thung) hoac `car` tuy goc nhin | Xem co thung cho hang khong. `can_review` |
| img_008 | Nhieu xe may dau day dac | R-13, R-18 | Gan tung `motorcycle` | Xe dau tinh, co the tach |
| img_009 | Minibus | R-03, R-04 | `van` | Minibus <= 16 cho, cabin-thung lien khoi |
| img_010 | Xe dap ganh hang (ban chuoi) | R-05 | Khong gan xe dap, gan `person` | Xe khong dong co |
| img_011 | Tuong Predator + mo hinh xe may kim loai | R-07 | Khong gan | Khong phai thuc the that |
| img_011 | Nguoi va xe o xa (festival) | R-12 | Kiem tra chieu cao px. < 20 px → `ignore_region` | Vat qua nho |
| img_012 | Pickup co thung dung hang (cua sau mo) | R-02 | `truck` | Thung cho hang nhin ro |
| img_013 | Tuong con vat vang truoc nha hang | R-07 | Khong gan | Tuong, khong phai thuc the song |
| img_013 | Xe o xa trong dem, it anh sang | R-12 | Gan neu > 20 px va nhan duoc lop | Xe that nhung nho va toi |
| img_014 | Xe may nhoe do motion blur | R-11 | Gan neu nhan ra lop, `can_review` | Van nhan dien duoc hinh dang xe may |
| img_014 | 2 nguoi qua duong ro rang | R-00 | Gan `person`, `occluded_level=0` | Nhin ro |
| img_015 | Xe bi nhoe giua anh | R-11 | Gan `car`, `can_review` | Van nhan ra la car du nhoe |
| img_015 | 2 xe bi cat mep (trai va phai) | R-00 | Gan `car`, `truncated=yes` | Nhin thay phan dang ke cua xe |
| img_016 | 2 tuk-tuk va 1 scooter Vespa | R-06 | Tuk-tuk: khong gan. Scooter: `motorcycle` | Scooter la xe 2 banh co dong co |
| img_017 | Tuong nguoi cuoi ngua co non giao thong | R-07 | Khong gan `person` hay bat ky lop nao | Tuong dong, khong phai thuc the |
| img_018 | Hinh nguoi tren billboard, xe sau hang rao | R-09, R-12 | Khong gan hinh billboard. Xe sau rao: gan neu > 20 px | Hinh in khong phai vat that |
| img_019 | Xe dieu hanh vang co bien so | R-08 | Gan `car` | Xe that, di duoc, co bien so |
| img_019 | Mascot chuot tren xe | R-07 | Khong gan `person` | Bup be/mascot, khong phai nguoi |
| img_019 | Nguoi lai xe dieu hanh | R-00 | Gan `person` | Nguoi that |
| img_019 | Dam dong khan gia 2 ben | R-13 | Phan gan: gan tung `person`. Phan xa day: `ignore_region` | Tach duoc gan, khong tach duoc xa |
| img_020 | Xe chi thay qua guong chieu hau | R-09 | Khong gan | Phan chieu, khong phai vat that trong canh |
| img_021 | Giao thong cuc ky day dac Sai Gon | R-13 | Phan truoc gan tung box; vung giua + xa → `ignore_region` | Khong the tach het |
| img_022 | Xe may co sidecar (thung phu) | R-01 | Gan `motorcycle` (ca xe + sidecar 1 box) | Van la xe may |
| img_023 | Xich lo (cyclo) 3 banh dap | R-05 | Khong gan xich lo, gan `person` cho nguoi dap | Xe thu cong khong dong co |
| img_024 | Xe that trong bao tang + xe mo hinh nho tren noc | R-10, R-07 | Xe that: `can_review` (co can phat hien?). Mo hinh nho: khong gan | Xe that nhung boi canh khong phai duong pho. `can_review` |
| img_025 | Nguoi va xe xa tren duong ven bien | R-12 | Kiem tra px, gan neu > 20 px | Co the nho |
| img_026 | Poster/billboard phia sau xe may | R-09 | Khong gan hinh tren poster. Gan xe may + 2 nguoi | Chi gan vat that |
| img_027 | Mo hinh xe tai bang da (bia mo) | R-07 | Khong gan | Tuong/dieu khac, khong phai xe that |
| img_028 | Xe tai co voi tuong ngua tren thung | R-07, R-02 | Xe tai: gan `truck`. Tuong ngua: khong gan. Xe do phia sau: gan `car` | Tach vat that va tuong |
| img_029 | Nguoi tren xe may va nguoi tren xe dap vai ong kim loai | R-05, R-01 | Xe may: `motorcycle` + `person`. Xe dap: chi `person`. Ong kim loai → che nguoi → `occluded_level=1-2` | Xe dap khong gan |
| img_030 | Bai do xe may rat day dac | R-13 | Co gang gan tung `motorcycle`. Neu khong tach duoc → `ignore_region` | Xe dau tinh, nhieu xe co the tach |
| img_031 | Pho dem, xe o to, xe may, phan chieu tren mat duong uot | R-00 | Gan xe va nguoi that. Khong gan phan chieu tren duong | Phan chieu khong phai vat |
| img_032 | Dam dong cho bus, xe phia sau | R-13 | Nguoi gan: gan tung `person`. Nguoi xa: `ignore_region` | Tach duoc phan gan |
| img_033 | Ma-no-canh va guong xe tai | R-07 | Khong gan `person` cho ma-no-canh | Khong phai nguoi that |
| img_034 | Nguoi va xe nguoc sang | R-11 | Gan binh thuong, `can_review` neu khong chac lop | Van nhan dien duoc |
| img_035 | Auto-rickshaw 3 banh cho hang day | R-06 | Khong gan box, ghi issue | Khong co lop. `can_review` |
| img_035 | Xe may va nguoi ro rang | R-00, R-01 | `motorcycle` + `person` | Binh thuong |
| img_036 | Xe tai nho VeryCA — van hay truck? | R-03 | `truck` | Cabin va thung tach roi |
| img_037 | Bus GO Transit xanh | R-04 | Khong gan, ghi issue | Bus lon khong co lop |
| img_037 | Pickup trang voi camper top | R-02 | `truck` | Co thung rieng (camper) |
| img_038 | Tu kinh trung bay xe mo hinh / do choi | R-10 | Khong gan | Khong phai xe that |
| img_039 | Pho nho Viet Nam, xe may va nguoi | R-00 | Gan binh thuong | Tinh huong chuan |
| img_040 | Nguoi di xe may can canh, tuk-tuk va taxi phia sau | R-06, R-01 | Xe may + nguoi: gan. Tuk-tuk: khong gan. Taxi: `car`. Bus: khong gan (R-04) | Ap dung nhieu quy tac |

## 5. QC va phien ban

### Nguong QC ap dung (tu taxonomy.schema.json — KHONG doi)
- `min_box_height_px`: 20 — vat cao < 20 px khong gan
- `iou_thr_large`: 0.85 — box cao >= 50 px
- `iou_thr_small`: 0.70 — box cao < 50 px
- `iaa_iou`: 0.85 — IoU trung binh giua cac cap
- `iaa_class`: 0.95 — ty le cung lop
- `error_rate_img_max`: 0.02 — toi da 2% anh loi
- `match_iou`: 0.50 — nguong ghep box

### Ai quyet ca mo ho
- Buoc 1: annotator tu quyet dinh theo guideline, tick `can_review` neu chua chac.
- Buoc 2: hop nhom (Pha 2 / Pha 5) chot ca con lai.
- Buoc 3: reviewer (coach) quyet dinh cuoi cung neu nhom khong dong thuan.

### Lich su thay doi

| Phien ban | Thay doi | Ly do (ca nao) |
|---|---|---|
| v0 | Ban goc cua giang vien | Chua du chi tiet |
| v0.1 | Them 18 quy tac (R-00 den R-14), dinh nghia nguong occluded, xu ly 40 anh cu the | Xem 40 anh, phat hien nhieu ca v0 khong bao: tuk-tuk, tuong, xe dieu hanh, dam dong, motion blur, xe cho xe, phan chieu, bus |

---

## Tong hop van de CAN NHOM THAO LUAN

1. **Tuk-tuk / auto-rickshaw** (img_004, img_016, img_035, img_040): them lop moi, gan `motorcycle`, hay bo qua?
2. **Bus lon** (img_037, img_040): them lop `bus`, gan `truck`, hay bo qua?
3. **Xe trong bao tang** (img_024): co gan khong khi khong phai boi canh duong pho?
4. **Xe dap dien** (neu co): phan biet voi xe dap thuong the nao?
5. **Nguoi trong xe kin** (taxi, van): chi thay bong qua kinh — co gan `person` khong?
