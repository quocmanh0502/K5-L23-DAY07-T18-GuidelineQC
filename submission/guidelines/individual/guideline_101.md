# Guideline gán nhãn — v0.1 (bản cá nhân) — 101

- Phiên bản: v0.1 (sửa từ `guideline_v0_starter.md`)
- Ngày: 2026-11-16
- Người viết: 101 · Người duyệt: nhóm T18 (Pha 2)
- Phạm vi: 40 ảnh `img_001` … `img_040`, bounding box 2D, CVAT, xuất COCO 1.0

> Bản này trả lời các câu v0 bỏ ngỏ. Mỗi quy tắc có mã `R-xx` để dẫn lại trong nhật ký QC và bảng calibration.
> Phép thử chung khi phân vân: **"Mô hình có phải đoán ra điều này không?"** Nếu phải đoán, không vẽ box mà
> tick `can_review` hoặc ghi vấn đề cấp ảnh.

---

## 1. Mục tiêu

- **Mô hình:** phát hiện vật thể 2D (object detection) cho camera giao thông / xe tự lái, nhận ra **người và phương tiện cơ giới thật đang tham gia hoặc có thể tham gia giao thông**.
- **Nhãn dùng để:** huấn luyện và đánh giá mô hình phát hiện 5 lớp `car`, `truck`, `van`, `motorcycle`, `person`.
- **Sai ở đâu thì nguy hiểm:**
  - **Bỏ sót người** (nhất là người bị che, người ngồi trên xe máy): xe tự lái không phanh → nguy hiểm nhất.
  - **Gán vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu): mô hình phanh gấp vô cớ, đếm sai lưu lượng.
  - **Sai lớp xe** (`car`/`van`/`truck`): ảnh hưởng ước lượng kích thước, quãng phanh, thống kê giao thông.

## 2. Lớp và thuộc tính

### 2.1 Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ, thân liền khối không có thùng hàng tách rời: sedan, hatchback, wagon, SUV, crossover, taxi. | xe đen `img_023`, Mazda CX-5 `img_012`, taxi xanh `img_040`, xe đỏ/bạc `img_015`, xe vàng diễu hành `img_019` | SUV ≠ `truck`; bán tải (có thùng hở phía sau) là `truck` (R-07); xe đồ chơi `img_038` không gán (R-02) |
| `truck` | Xe chở hàng có **cabin và thùng hàng tách rời**: xe tải, xe tải nhẹ, **xe bán tải (pickup)** kể cả có nắp thùng, xe chở xe. | Isuzu ELF `img_007`, xe chở xe `img_002`, bán tải `img_012`, `img_028`, `img_037`, xe tải nhỏ `img_001`, `img_036` | tượng xe bán tải `img_027` không gán (R-02); van liền khối là `van` |
| `van` | Xe thân kín liền khối, cabin và khoang sau chung một khối, **≤ 16 chỗ** hoặc chở hàng cỡ nhỏ (Hiace, Transit, Sprinter). | xe van bạc `img_005`, van trắng `img_016`, van trắng ở hậu cảnh `img_011` | xe buýt/minibus > 16 chỗ (`img_009`, `img_031`, `img_037`, `img_040`) **không phải** `van` (R-05) |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, tay ga, mô tô, xe máy điện có bàn để chân), kể cả khi đang đỗ; **bao gồm thùng sidecar gắn liền**. | `img_008`, `img_010`, Vespa `img_016`, sidecar `img_022`, xe máy điện `img_029` | xe đạp, xích lô không gán (R-06); tuk-tuk / xe lam 3 bánh là `ignore_region` (R-05) |
| `person` | Người thật nhìn thấy được: đi bộ, đứng, ngồi, **lái hoặc ngồi trên xe 2 bánh / xích lô / xe đạp**. | `img_014`, `img_026`, `img_032`, `img_034` | tượng `img_017`, ma-nơ-canh `img_033`, người trên biển quảng cáo `img_018`, nhân vật hóa trang/tượng `img_011`, búp bê chuột `img_019` không gán (R-02); người trong cabin ô tô không gán (R-04) |
| `ignore_region` | Vùng chữ nhật không gán từng vật; vật nằm trong vùng này bỏ sót không tính lỗi. | tuk-tuk `img_004`, xe buýt `img_009`, đám đông xa `img_021`, cụm vật < 20 px | **không** dùng ignore để "né" vật thật đủ lớn và tách được (R-14, R-15) |

**Vật không thuộc lớp nào thì làm gì?** (một cách duy nhất, theo thứ tự):

1. **Không phải vật thật** (tượng, mô hình, đồ chơi, hình in, poster, tranh tường, phản chiếu): **không gán gì** (R-02).
2. **Phương tiện cơ giới thật ngoài taxonomy** (xe buýt, minibus > 16 chỗ, tuk-tuk, auto-rickshaw, xe lam): **`ignore_region`** bao trọn xe (R-05). Lý do: là vật thật trên đường, nếu để nền thì dạy mô hình rằng nó "không có gì"; nếu ép vào lớp gần nhất thì làm nhiễu lớp đó.
3. **Phương tiện không động cơ** (xe đạp, xích lô, xe đẩy, gánh hàng): **không gán xe**, nhưng **người trên đó vẫn gán `person`** (R-06).
4. Vật khác (cọc tiêu, động vật, biển báo, đồ đạc): không gán.

### 2.2 Thuộc tính

Gán trên **mọi box** trừ `ignore_region` (R-18). Chỉ dùng trường thuộc tính của Project, **không dùng cờ occluded sẵn của CVAT (phím `Q`)**.

**`occluded_level`**: % diện tích **của vật** bị **vật khác** che (ước lượng như thể thấy toàn bộ vật). Phần nằm ngoài mép ảnh **không** tính vào đây mà tính vào `truncated`.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10% | người đi bộ `img_014`, xe tải xanh `img_036` |
| `1` | 10% ≤ che < 40% | xe sau hàng rào song thưa `img_018`; xe SUV đỏ sau xe bán tải `img_028` |
| `2` | 40% ≤ che < 80% | người ngồi sau bàn quán `img_001`; xe máy trong dòng xe `img_021` |
| `3` | che ≥ 80%, **hoặc** không ước lượng được % (tối, nhòe) nhưng vẫn chắc chắn là vật thuộc lớp đó | người khuất gần hết sau người khác `img_005` |

- Hàng rào song/lưới mảnh, vật vẫn thấy rõ hình dạng: tính theo phần thực sự bị song che (thường mức `1`).
- Người ngồi trên xe máy: **xe máy che người và người che xe máy đều tính** (chân người che thân xe).
- Box chồng nhau không phải là che: chỉ tính khi vật phía trước thực sự lấp vật phía sau.

**`truncated`**

| Giá trị | Khi nào |
|---|---|
| `yes` | Vật **kéo dài ra ngoài mép ảnh** (box chạm mép ảnh và một phần vật bị cắt mất), bất kể cắt bao nhiêu. Ví dụ: xe đỏ trái và xe bạc phải `img_015`, xe tải nhỏ cắt mép phải `img_001`, người áo đỏ mép dưới `img_016`. |
| `no` | Toàn bộ vật nằm trong khung ảnh (có thể bị che, đó là `occluded_level`). |

**`can_review`** (checkbox): tick khi box **đã vẽ** nhưng còn nghi ngờ về lớp, ranh giới hoặc vật thật/giả. Mỗi lần tick ghi lý do ngắn (ảnh, vật, nghi ngờ gì) để nhóm thu vào nhật ký. Không tick chỉ vì vật nhỏ hay bị che, vì đã có thuộc tính riêng.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Chỉ gán vật thật**: người thật và phương tiện thật kích thước thật, kể cả đang đỗ, đang trưng bày nhưng là xe thật chạy được (bảo tàng xe `img_024`, xe diễu hành có biển số `img_019`). |
| **R-02** | **Không gán vật không thật**: tượng (`img_017`, `img_027`, ngựa trên xe `img_028`, rồng/tượng `img_013`), ma-nơ-canh (`img_033`), xe đồ chơi/mô hình (`img_038`, xe vàng thu nhỏ trên nóc `img_024`), hình in/poster/biển quảng cáo (`img_018`, `img_001`, `img_036`, `img_040`), tranh tường (`img_026`), phản chiếu trên kính, gương, mặt đường ướt (`img_020`, `img_033`, `img_015`, `img_031`), nhân vật hóa trang/búp bê (`img_011`, `img_019`). |
| **R-03** | **Người trên xe 2 bánh: 2 loại box riêng**. `person` cho **mỗi** người (người lái, người ngồi sau, trẻ em ngồi trước), bao toàn bộ phần thấy được của người (mũ bảo hiểm, tay, chân). `motorcycle` bao thân xe (bánh, gương, giỏ, yên, đồ gắn cố định). Box người và box xe **được phép chồng nhau**; không cắt box xe để tránh người. |
| **R-04** | **Người trong cabin kín** (ô tô, xe tải, van, nhìn qua kính) **không gán `person`**: mô hình không cần đoán người sau kính, và vật bao ngoài đã có box xe. Ví dụ tài xế `img_019`, `img_002`. |
| **R-05** | **Phương tiện cơ giới ngoài taxonomy → `ignore_region`** bao trọn xe kể cả hàng chở trên xe: xe buýt / minibus > 16 chỗ (`img_009`, `img_031`, `img_037`, `img_040`), tuk-tuk (`img_004`, `img_016`, `img_040`), auto-rickshaw (`img_035`). Người ngồi trong các xe này nằm trong ignore → không gán (R-18). |
| **R-06** | **Xe không động cơ** (xe đạp `img_003`, `img_006`, `img_029`; xích lô `img_023`, `img_039`; xe đạp bán hàng `img_010`) **không gán**, không đặt ignore. **Người** đi, ngồi, đứng cạnh xe **vẫn gán `person`**. |
| **R-07** | **Phân biệt lớp xe**: có thùng hàng tách rời cabin (kể cả thùng có nắp/mui bạt) → `truck`; thân kín liền khối ≤ 16 chỗ → `van`; xe chở người cỡ nhỏ (sedan, SUV, taxi) → `car`. Bán tải (pickup) **luôn là `truck`** (`img_001`, `img_012`, `img_028`, `img_037`, `img_007`). |
| **R-08** | **Xe chở xe / xe trên xe**: xe chở (đầu xe + thân + rơ-moóc nối liền) → **một** box `truck` bao trọn tổ hợp. Mỗi xe thật đang được chở → box `car` riêng, `occluded_level` tính theo phần bị khung sắt/xe khác che (`img_002`). Mô hình/đồ chơi đặt trên xe → không gán (R-02, `img_024`, ngựa `img_028`). |
| **R-09** | **Sidecar / thùng gắn liền xe máy** → thuộc box `motorcycle` (một box bao cả xe và thùng) (`img_022`, xe máy kéo thùng bán hàng `img_025`). |
| **R-10** | **Hàng chở cồng kềnh nhô ra** (ống thép `img_029`, bao hàng) **không tính** vào box xe hay người. Box chỉ bao thân xe/người. |
| **R-11** | **Box ôm phần nhìn thấy** (không đoán phần bị che); mỗi cạnh box cách mép vật **≤ 3 px**. Không bao bóng đổ, vệt phản chiếu. |
| **R-12** | `occluded_level` theo ngưỡng % ở mục 2.2. |
| **R-13** | `truncated` theo mục 2.2: cắt bởi mép ảnh → `yes`. |
| **R-14** | **Vật nhỏ**: vật cao **< 20 px** (theo ảnh gốc) không vẽ box lớp; nếu muốn đánh dấu (nhất là người/xe ở xa) → bao bằng **một** `ignore_region` cho cả cụm. Vật ≥ 20 px phải gán lớp. |
| **R-15** | **Đám đông / cụm dày**: nhóm ≥ 5 vật cùng lớp chồng lấn đến mức **không tách được ranh giới từng vật** → một `ignore_region` bao cả cụm. Vật ở rìa cụm hoặc tiền cảnh **tách được** thì vẫn gán riêng (`img_021`, `img_030`, `img_032`, `img_019`). |
| **R-16** | **Vật nhòe do chuyển động**: nhận ra được lớp → gán, box ôm phần thân **đậm nhất** (không bao vệt nhòe kéo dài), tick `can_review`. Nhòe đến mức không nhận ra lớp → không gán, ghi vấn đề cấp ảnh (`img_014`, `img_015`). |
| **R-17** | **Xe của người chụp (ego)** và mọi thứ nhìn qua gương chiếu hậu của xe đó → không gán (`img_020`, gương xe máy `img_040`). |
| **R-18** | Vật nằm **trong** `ignore_region` không gán thêm; `ignore_region` không cần thuộc tính. |
| **R-19** | **Thấy quá ít**: chỉ gán khi phần thấy được đủ để nhận ra lớp **mà không cần đoán từ ngữ cảnh**. Chỉ thấy gương/tay lái của xe máy ở mép ảnh (`img_040`) → không gán. |
| **R-20** | **Xe thật nhưng kiểu dáng lạ** (xe độ, xe diễu hành, xe nghệ thuật): có bánh, kích thước thật, biển số hoặc rõ ràng chạy được → gán theo hình dạng/chức năng gần nhất (`img_019` → `car`). Nếu không chắc là xe thật hay tác phẩm trưng bày (`img_011`) → gán và tick `can_review`. |
| **R-21** | **Ảnh không có vật nào** (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) vẫn phải nằm trong Task và trong file COCO xuất ra, không có annotation. |

## 4. Ca khó

Phân loại: **QT** = xử lý được bằng quy tắc ở mục 3; **HỎI** = cần nhóm/reviewer chốt ở Pha 2.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_002 | Xe tải chở 7 ô tô, có rơ-moóc phía sau; người đứng ở đuôi xe | R-08, R-07 | 1 `truck` bao đầu xe + thân + rơ-moóc; mỗi ô tô thật trên giá → `car` riêng (occluded theo khung sắt); người → `person` | Ô tô trên giá là ô tô thật, để nền sẽ dạy mô hình bỏ qua ô tô | HỎI (gán xe đang được chở hay không) |
| img_024 | Xe vàng thu nhỏ đặt trên nóc AMC Pacer đen trong bảo tàng | R-01, R-02, R-08 | Xe đen và các xe thật xung quanh → `car`; xe vàng trên nóc → không gán | Xe vàng là mô hình/xe trưng bày cỡ nhỏ, không phải phương tiện | QT |
| img_027 | Tượng đá hình xe bán tải trong nghĩa trang | R-02, R-21 | Không gán, ảnh trống | Tượng, không phải xe | QT |
| img_028 | Xe bán tải thật chở tượng ngựa; SUV đỏ phía sau | R-07, R-02 | Bán tải → `truck`; ngựa → không gán; SUV đỏ → `car` (`occluded_level` theo phần bị che) | Bán tải có thùng hở tách cabin | QT |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông | R-02, R-21 | Không gán, ảnh trống | Tượng không phải người | QT |
| img_033 | Ma-nơ-canh trong tủ kính, phản chiếu người và xe van | R-02, R-21 | Không gán, ảnh trống | Ma-nơ-canh và ảnh phản chiếu không phải vật thật trong cảnh | QT |
| img_038 | Tủ kính đầy xe mô hình, đồ chơi | R-02, R-21 | Không gán, ảnh trống | Đồ chơi | QT |
| img_020 | Gương chiếu hậu phản chiếu xe phía sau | R-17, R-02, R-21 | Không gán, ảnh trống | Phản chiếu + xe chủ | QT |
| img_018 | Biển quảng cáo có ảnh người; ô tô sau hàng rào | R-02, R-12 | Người trên biển → không gán; ô tô → `car`, `occluded_level` theo phần bị song che (thường `1`) | Hình in không phải người | QT |
| img_019 | Xe vàng kiểu hoạt hình chạy diễu hành, có biển số, có búp bê chuột; đám đông sau rào | R-20, R-02, R-04, R-15 | Xe → `car`; búp bê → không gán; tài xế → không gán; người phía trước tách được → `person`; khối đông phía sau → `ignore_region` | Xe thật có biển số, chạy trên đường | HỎI (`car` hay `ignore_region`) |
| img_011 | Tượng nhân vật Predator; dãy xe máy độ bằng kim loại trưng bày | R-02, R-20, R-15 | Tượng → không gán; xe máy độ → `motorcycle` + `can_review`; người xa tách được → `person`; cụm người xa → `ignore_region` | Không chắc là xe chạy được hay tác phẩm | HỎI |
| img_004 | Hai tuk-tuk có tài xế ngồi trong | R-05, R-18 | Mỗi tuk-tuk → 1 `ignore_region`; tài xế nằm trong → không gán | Taxonomy không có lớp xe 3 bánh | HỎI (ignore hay `motorcycle`) |
| img_035 | Auto-rickshaw chở bao hàng cao; người lái xe máy; ô tô trắng | R-05, R-03 | Rickshaw (kể cả hàng) → `ignore_region`; xe máy → `motorcycle` + `person`; ô tô → `car` | Như img_004 | QT |
| img_016 | Tuk-tuk, Vespa, bán tải, van, người đi bộ mép ảnh | R-05, R-03, R-07, R-13 | Tuk-tuk → `ignore_region`; Vespa → `motorcycle` + `person`; bán tải → `truck`; van → `van`; người áo đỏ → `person`, `truncated=yes` | Nhiều lớp trong một ảnh | QT |
| img_009 | Minibus ~29 chỗ; xe máy có người; người trên ban công | R-05, R-03, R-14 | Minibus → `ignore_region`; xe máy → `motorcycle` + `person`; người ban công ≥ 20 px → `person` | Minibus > 16 chỗ không phải `van` | HỎI (ngưỡng van/bus) |
| img_005 | Nhiều người trên xe máy chen sát nhau, che lẫn nhau; van phía sau | R-03, R-12, R-07 | Mỗi người → `person` riêng (`occluded_level` 1–3); xe máy thấy được phần → `motorcycle`; xe phía sau → `van` | Không gộp người dù chồng lấn | QT |
| img_026 | Hai người trên một xe máy, nền là tranh tường cảng biển | R-03, R-02 | 2 `person` + 1 `motorcycle`; tranh tường → không gán | Mỗi người một box | QT |
| img_022 | Mô tô có sidecar, ảnh đen trắng cũ | R-09, R-03 | 1 `motorcycle` bao xe + sidecar; người ngồi lái → `person` (che nhiều) | Sidecar gắn liền xe | QT |
| img_025 | Xe máy kéo thùng hàng bên cạnh; người ngồi ghế bãi biển dày đặc | R-09, R-15, R-07 | Xe + thùng → `motorcycle`; người lái → `person`; người đi bộ tách được → `person`; cụm ghế/người xa → `ignore_region`; taxi → `car`; bán tải mép phải → `truck`, `truncated=yes` | Thùng gắn liền xe | QT |
| img_029 | Xe máy điện chở ống thép dài; người dắt xe đạp | R-10, R-06, R-03 | Xe máy điện → `motorcycle` (không bao ống); người → `person`; xe đạp → không gán | Hàng chở không thuộc vật | QT |
| img_023 | Xích lô có người đạp; ô tô; xe máy | R-06, R-03 | Xích lô → không gán; người đạp → `person`; ô tô → `car`; xe máy xa → `motorcycle` + `person` | Xích lô không động cơ | HỎI (xích lô có cần ignore không) |
| img_021 | Dòng xe máy rất dày, đèn đỏ | R-15, R-03, R-14 | Xe/người tiền cảnh tách được → gán riêng; khối xa không tách được → `ignore_region` | Tránh vẽ hàng trăm box không chắc | QT |
| img_030 | Bãi xe máy đỗ chồng lấn dày | R-15 | Xe tiền cảnh tách được → `motorcycle`; phần dày phía sau → `ignore_region`; người áo xanh → `person` | Như img_021 | HỎI (ranh giới "tách được") |
| img_032 | Đám đông chờ xe buýt | R-15, R-02 | Người có đầu + thân tách được → `person`; cụm chồng kín → `ignore_region`; poster nhà chờ → không gán | Đám đông | HỎI |
| img_014 | Người đi bộ rõ, xe máy và người nhòe chuyển động | R-16 | 2 người rõ → `person`; xe/người nhòe còn nhận ra → gán + `can_review`; nhòe quá → không gán | Nhòe | HỎI |
| img_015 | Xe đỏ, xe bạc cắt mép; xe hậu cảnh nhòe; mặt đường ướt phản chiếu | R-13, R-16, R-02 | 2 xe cắt mép → `car`, `truncated=yes`; xe đỗ hậu cảnh → `car`; xe nhòe → R-16; phản chiếu → không gán | | QT |
| img_040 | Người đội mũ bảo hiểm cận cảnh; chỉ thấy gương xe máy; taxi; tuk-tuk; xe buýt | R-19, R-05, R-13 | Người → `person`, `truncated=yes`; gương xe máy → không gán; taxi → `car`, `truncated=yes`; tuk-tuk, xe buýt → `ignore_region`; SUV đen → `car` | Gương không đủ nhận ra xe | QT |
| img_007 | Kho tối, bán tải có thùng kín, xe tải Isuzu; có thể có người khuất giữa hai xe | R-07, R-12 | Bán tải → `truck`; Isuzu → `truck`; nếu thấy người → `person` (`occluded_level` 3) + `can_review` | Thiếu sáng | HỎI |
| img_013 | Đêm, tượng rồng vàng, người đi bộ, xe xa | R-02, R-14 | Tượng → không gán; 2 người đi bộ → `person`; xe máy đỗ → `motorcycle`; xe < 20 px → `ignore_region` | | QT |
| img_001 | Người ngồi quán bị bàn che; xe tải nhỏ cắt mép; banner hot dog | R-12, R-07, R-02 | Mỗi người → `person` (`occluded_level` theo bàn/ghế/người khác); xe → `truck`, `truncated=yes`; banner → không gán | | QT |

## 5. QC và phiên bản

- **Ngưỡng QC** (giữ nguyên mục `rules` của `taxonomy.json`):
  - Box cao < 20 px → R-14.
  - IAA: ghép cặp IoU ≥ 0.5; IoU trung bình ≥ 0.85 (vật cao ≥ 50 px), ≥ 0.7 (vật nhỏ hơn); đồng thuận lớp ≥ 95%.
  - Release khi tỉ lệ ảnh lỗi ≤ 2% trên **toàn lô** đã duyệt; luôn ghi kèm mẫu số.
- **Ai quyết ca mơ hồ:** người gán tick `can_review` / ghi vấn đề cấp ảnh → nhóm chốt ở calibration (Pha 5), người chốt ghi mã; người kiểm lại phải khác người sửa.
- **Đổi phiên bản:** đổi câu chữ, không đổi nhãn → `editorial`; đổi cách chọn vật, lớp, box, thuộc tính → `semantic`, tăng phiên bản và rà lại cả lô.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 | Thêm định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng % `occluded_level`; định nghĩa `truncated`; quy tắc R-01…R-21; bảng ca khó | Người đi xe máy (`img_005`, `img_026`), vật không thật (`img_017`, `img_027`, `img_033`, `img_038`), xe ngoài taxonomy (`img_004`, `img_009`, `img_035`), xe chở xe (`img_002`, `img_024`), đám đông (`img_021`, `img_032`) |

---

## Phụ lục: quyết định nhanh cho 40 ảnh

| Ảnh | Nội dung chính | Lớp cần gán / xử lý |
|---|---|---|
| img_001 | Quán vỉa hè, người ngồi, xe tải nhỏ Suzuki | `person` (nhiều, bị che), `truck` cắt mép; banner không gán |
| img_002 | Xe chở xe | `truck` (1), `car` cho từng xe trên giá, `person` (công nhân) |
| img_003 | Người bán hàng rong Hà Nội | `person`, `motorcycle` (có người và đỗ); xe đạp không gán; cụm xe xa → R-14/R-15 |
| img_004 | 2 tuk-tuk ban đêm | `ignore_region` ×2 |
| img_005 | Người đi xe máy cận cảnh, van | `person` (nhiều), `motorcycle`, `van`, `car` cắt mép trái |
| img_006 | Dòng xe máy, xe đạp, taxi | `person`, `motorcycle`, `car` (taxi cắt mép); xe đạp không gán; cụm xa → `ignore_region` |
| img_007 | Kho tối, bán tải thùng kín, xe tải | `truck` ×2; người khuất → `can_review` |
| img_008 | Phố đêm, xe máy đỗ, tài xế xe ôm | `motorcycle`, `person`; cụm xa → `ignore_region` |
| img_009 | Phố Huế, minibus | minibus → `ignore_region`; `motorcycle`, `person` |
| img_010 | Hàng hoa quả, người trên xe tay ga | `person`, `motorcycle`; xe đạp bán hàng không gán |
| img_011 | Tượng Predator, xe máy độ | tượng không gán; `motorcycle` + `can_review`; `van`; người xa |
| img_012 | Bán tải chở hàng, SUV | `truck` (bán tải, xe tải phía trước), `car` |
| img_013 | Đêm, tượng rồng, người đi bộ | `person`, `motorcycle`; tượng không gán; vật nhỏ → `ignore_region` |
| img_014 | Người qua đường, xe nhòe | `person`; vật nhòe → R-16 |
| img_015 | Xe dưới mưa, cắt mép, nhòe | `car` (`truncated=yes`), R-16 |
| img_016 | Tuk-tuk, Vespa, bán tải, van | `ignore_region`, `motorcycle`, `person`, `truck`, `van`, `car` |
| img_017 | Tượng người cưỡi ngựa | ảnh trống |
| img_018 | Biển quảng cáo, xe sau hàng rào | `car` (che bởi rào); người trên biển không gán |
| img_019 | Xe diễu hành, đám đông | `car`; đám đông → `ignore_region` + `person` tách được |
| img_020 | Phản chiếu trong gương | ảnh trống |
| img_021 | Dòng xe máy dày đặc | tiền cảnh gán riêng, khối xa → `ignore_region` |
| img_022 | Mô tô sidecar | `motorcycle` (gồm sidecar), `person` |
| img_023 | Xích lô, ô tô | `person`, `car`, `motorcycle`; xích lô không gán |
| img_024 | Bảo tàng xe, xe mô hình trên nóc | `car` (xe thật); xe trên nóc không gán |
| img_025 | Đường ven biển | `person`, `motorcycle` (gồm thùng kéo), `car`, `truck`; cụm xa → `ignore_region` |
| img_026 | Hai người trên xe máy, tranh tường | `person` ×2, `motorcycle` |
| img_027 | Tượng xe bán tải | ảnh trống |
| img_028 | Bán tải chở tượng ngựa | `truck`, `car`; ngựa không gán |
| img_029 | Xe máy điện chở ống thép | `motorcycle`, `person`; xe đạp, ống không gán |
| img_030 | Bãi xe máy dày | `motorcycle` tách được, `ignore_region`, `person` |
| img_031 | Phố đêm Đài Loan | `car`, `motorcycle`, `person`; xe buýt → `ignore_region`; phản chiếu không gán |
| img_032 | Đám đông chờ xe buýt | `person`/`ignore_region` theo R-15; `car` |
| img_033 | Ma-nơ-canh, phản chiếu | ảnh trống |
| img_034 | Người qua đường, xe máy, ô tô | `person`, `motorcycle`, `car`; xe đạp không gán |
| img_035 | Auto-rickshaw quá tải | `ignore_region`, `motorcycle`, `person`, `car` |
| img_036 | Xe tải nhỏ mui bạt | `truck`; cụm xe máy đỗ → R-15; poster không gán |
| img_037 | Bán tải nắp thùng, xe buýt | `truck`, `person`; xe buýt → `ignore_region` |
| img_038 | Tủ kính đồ chơi | ảnh trống |
| img_039 | Phố Huế | `motorcycle`, `person`, `car`; xích lô không gán |
| img_040 | Người đội mũ, taxi, tuk-tuk, xe buýt | `person`, `car`, `ignore_region`; gương xe máy không gán |
