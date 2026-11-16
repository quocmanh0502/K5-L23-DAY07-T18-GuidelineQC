# Guideline gán nhãn — v1.0 (thống nhất) — nhóm T18

- Phiên bản: **v1.0** — khóa cho lượt gán độc lập (Pha 3)
- Ngày: 2026-11-16
- Người viết: 2A202603219 (điều phối) · Người duyệt: 2A202603197 (review)
- Nguồn: `individual/guideline_2A202603219.md`, `individual/guideline_2A202603197.md`
- Taxonomy đi kèm: `submission/taxonomy.json` version `v1.0` (mục `rules` giữ nguyên)
- Phạm vi: 40 ảnh `img_001` … `img_040`, bounding box 2D trên CVAT, xuất COCO 1.0

> Phép thử chung khi phân vân: **"Mô hình có phải đoán ra điều này không?"** Phải đoán → không vẽ box lớp;
> nếu đã vẽ mà còn nghi ngờ → tick `can_review`; không vẽ được → ghi vấn đề cấp ảnh (ảnh, vùng, lý do).

---

## 1. Mục tiêu

- **Mô hình:** phát hiện vật thể 2D cho camera giao thông / xe tự lái, nhận ra **người và phương tiện cơ giới thật** trong 5 lớp `car`, `truck`, `van`, `motorcycle`, `person`.
- **Sai ở đâu thì nguy hiểm:**
  1. **Bỏ sót người** (bị che, ngồi trên xe máy, ở mép ảnh) — nguy hiểm nhất, xe không phanh.
  2. **Gán vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) — mô hình báo động giả, đếm sai lưu lượng.
  3. **Sai lớp xe** (`car` / `van` / `truck`) — sai ước lượng kích thước, quãng phanh, thống kê.

## 2. Lớp và thuộc tính

### 2.1 Lớp

Mỗi vật thuộc **đúng một** lớp. Không tự thêm lớp khi gán.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ, thân liền khối, **không có thùng hàng tách rời**: sedan, hatchback, wagon, SUV, crossover, taxi. | xe đen `img_023`, SUV đỏ `img_028`, Mazda CX-5 `img_012`, taxi `img_040`, xe trên giá `img_002` | Bán tải → `truck`; xe thân cao, khoang sau liền cabin kiểu Hiace → `van`; xe đồ chơi `img_038` → không gán |
| `truck` | Xe có **cabin và thùng/sàn chở hàng tách rời**: xe tải, xe tải nhẹ, **bán tải (pickup) kể cả gắn nắp/khoang trên thùng**, xe chở ô tô. | `img_001`, `img_002`, `img_007` (cả hai xe), `img_012`, `img_028`, `img_036`, `img_037` | Tượng bán tải `img_027` → không gán; thùng kín liền cabin → `van` |
| `van` | Xe **thân cao, khoang sau liền khối với cabin**, không có thùng tách rời, **tối đa 16 chỗ** hoặc chở hàng cỡ nhỏ (kiểu Hiace, Transit, Sprinter). | xe bạc `img_005`, van trắng `img_016`, xe trắng hậu cảnh `img_011` | Minibus / xe buýt > 16 chỗ (`img_009`, `img_031`, `img_037`, `img_040`) → `ignore_region` (R-10) |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, tay ga, mô tô, xe máy điện), kể cả đang đỗ; **gồm cả thùng sidecar / thùng hàng gắn liền**. | `img_006`, `img_008`, `img_010`, sidecar `img_022`, xe kéo thùng `img_025`, xe điện `img_029` | Xe đạp, xích lô → không gán (R-11); tuk-tuk, auto-rickshaw → `ignore_region` (R-10) |
| `person` | **Người thật** nhìn thấy được: đi bộ, đứng, ngồi, lái hoặc ngồi trên xe máy / xe đạp / xích lô. | `img_014`, `img_026`, `img_032`, `img_034` | Tượng `img_017`, ảnh trên biển `img_018`, ma-nơ-canh `img_033`, tượng nhân vật `img_011`, búp bê `img_019`; người sau kính cabin (R-05) |
| `ignore_region` | Vùng chữ nhật **không đánh giá từng vật**; chỉ dùng cho 2 trường hợp ở R-10 và R-13. | tuk-tuk `img_004`, xe buýt `img_009`, khối xe máy xa `img_021` | Không dùng thay cho một lớp còn phân vân; không bao vật đã gán rõ |

**Vật ngoài taxonomy — một cách xử lý duy nhất:**

| Loại vật | Xử lý | Quy tắc |
|---|---|---|
| Không phải vật thật: tượng, mô hình, đồ chơi, ma-nơ-canh, hình in, poster, tranh tường, phản chiếu | **Không gán gì** | R-06 |
| Phương tiện **cơ giới** thật ngoài taxonomy: xe buýt, minibus > 16 chỗ, tuk-tuk, auto-rickshaw, xe lam | **`ignore_region`** bao trọn xe | R-10 |
| Phương tiện **không động cơ**: xe đạp, xích lô, xe đẩy, gánh hàng | **Không gán xe**, người trên đó vẫn gán `person` | R-11 |
| Vật khác: động vật, cọc tiêu, biển báo, đồ đạc, hàng hóa | Không gán | — |

### 2.2 Thuộc tính

Gán trên mọi box của 5 lớp. **`ignore_region` không có thuộc tính** (Project CVAT sinh từ taxonomy không cấu hình thuộc tính cho lớp này). Chỉ dùng trường thuộc tính của Project; **không dùng cờ occluded sẵn của CVAT (phím `Q`)** — script bỏ qua cờ đó.

**`occluded_level`** — % hình dáng **của vật** bị **vật khác** che, ước lượng như thể thấy toàn bộ vật. Phần ngoài mép ảnh **không** tính ở đây (tính vào `truncated`). Ước lượng bằng mắt, làm tròn về mức gần nhất.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che **< 10%** | người đi bộ `img_014`, xe tải `img_036` |
| `1` | **10% ≤ che < 40%** | xe sau hàng rào song thưa `img_018`, SUV đỏ sau bán tải `img_028` |
| `2` | **40% ≤ che < 80%** | người ngồi sau bàn quán `img_001`, xe máy giữa dòng xe `img_021` |
| `3` | **che ≥ 80%**, hoặc **không đủ căn cứ ước lượng** (tối, nhòe) nhưng vẫn chắc chắn lớp | người khuất gần hết sau người khác `img_005` |

- Hàng rào song mảnh / lưới: tính theo phần thực sự bị song che (thường mức `1`).
- Người trên xe máy: chân người che thân xe và xe che người **đều tính** là che.
- Box chồng nhau ≠ bị che: chỉ tính khi vật phía trước thực sự lấp vật phía sau.

**`truncated`**

| Giá trị | Khi nào |
|---|---|
| `yes` | Vật **thực sự bị cắt bởi biên ảnh**: một phần vật nằm ngoài khung, box dừng tại biên (`img_015` hai xe, `img_001` xe tải, `img_016` người áo đỏ, `img_040` người và taxi). |
| `no` | Vật nằm trọn trong ảnh. **Chạm biên nhưng không mất phần nào → `no`.** |

Che khuất và cắt mép **độc lập**: một vật có thể vừa `occluded_level ≥ 1` vừa `truncated=yes`.

**`can_review`** — tick khi box **đã vẽ** nhưng còn nghi ngờ lớp, ranh giới, thuộc tính hoặc thật/giả; mỗi lần tick ghi câu hỏi ngắn (ảnh, vật, nghi ngờ gì). **Không** tick chỉ vì vật nhỏ hay bị che — đã có thuộc tính riêng.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Box ôm sát phần nhìn thấy**, mỗi cạnh cách mép vật **≤ 3 px**. Không kéo box tới phần bị khuất, không suy đoán phần ngoài ảnh. Không bao bóng đổ, phản chiếu. Chỉ dùng Rectangle. |
| **R-02** | **Một vật = một box.** Các phần nhìn thấy bị vật khác ngăn cách vẫn dùng **một** box nếu chắc chắn cùng một vật (box được phép chứa vật che ở giữa). |
| **R-03** | **Cắt mép:** box dừng tại biên ảnh, `truncated=yes` (mục 2.2). |
| **R-04** | **Người và xe 2 bánh là đối tượng riêng:** một `motorcycle` cho xe, một `person` cho **từng** người (lái, ngồi sau, trẻ em). Box xe bao thân, bánh, gương, giỏ, yên, đồ gắn cố định; box người bao cơ thể, quần áo, mũ bảo hiểm. Box người và box xe **được phép chồng nhau**. Hai người trên một xe = **3 box** (`img_026`). |
| **R-05** | **Người trong cabin kín** (ô tô, xe tải, van — nhìn qua kính) **không gán `person`** (`img_019` tài xế xe vàng, `img_002` tài xế xe chở xe). |
| **R-06** | **Không gán vật không thật:** tượng (`img_017`, `img_027`, ngựa `img_028`, rồng `img_013`, nhân vật `img_011`), ma-nơ-canh (`img_033`), mô hình / đồ chơi (`img_038`, xe vàng nhỏ trên nóc `img_024`), búp bê / nhân vật trang trí (`img_019`), hình in / poster / biển quảng cáo (`img_001`, `img_018`, `img_036`, `img_040`), tranh tường (`img_026`), phản chiếu trên kính, gương, mặt đường ướt (`img_015`, `img_020`, `img_031`, `img_033`). Vật thật cùng ảnh vẫn gán. |
| **R-07** | **Xe gắn camera (ego)** và mọi thứ thấy qua gương của nó → không gán (`img_020`). |
| **R-08** | **Phân biệt lớp xe:** thùng/sàn hàng tách cabin → `truck`; thân cao liền khối ≤ 16 chỗ → `van`; xe chở người cỡ nhỏ → `car`. **Bán tải luôn là `truck`**, kể cả gắn khoang cao trên thùng (`img_037`) hoặc thùng kín (`img_007`). |
| **R-09** | **Xe chở xe:** xe chuyên chở = **một** box `truck` bao **đầu xe + thân + rơ-moóc nối liền** (`img_002`). Mỗi ô tô thật trên giá = box `car` riêng, `occluded_level` theo phần bị khung sắt / xe khác che. Box được phép chồng nhau. Vật mô phỏng đặt trên xe (`img_024`, `img_028`) → R-06. |
| **R-10** | **Phương tiện cơ giới ngoài taxonomy → `ignore_region`** bao trọn xe kể cả hàng chở trên xe: xe buýt / minibus > 16 chỗ (`img_009`, `img_031`, `img_037`, `img_040`), tuk-tuk (`img_004`, `img_016`, `img_040`), auto-rickshaw (`img_035`). Người ngồi **trong** các xe này nằm trong vùng ignore → không gán. |
| **R-11** | **Xe không động cơ** (xe đạp `img_003`, `img_006`, `img_029`, `img_034`; xích lô `img_023`, `img_039`; xe đạp bán hàng `img_010`) → **không gán, không đặt ignore**. Người trên / cạnh xe **vẫn gán `person`**. |
| **R-12** | **Vật nhỏ:** chiều cao box phần nhìn thấy **< 20 px trên ảnh gốc** → **bỏ**, không kéo box cho đủ, không đặt ignore riêng cho vật nhỏ lẻ. Đo trên ảnh gốc, phóng màn hình không đổi ngưỡng. |
| **R-13** | **Đám đông / cụm dày:** nhóm **≥ 5 vật** cùng lớp chồng lấn đến mức **không đặt được box đáng tin cho từng vật** → một `ignore_region` bao **chỉ phần không tách được**. Vật ở tiền cảnh / rìa cụm còn tách được thì gán riêng. Không đặt `ignore_region` lên vật đã gán; chia nhiều vùng nhỏ nếu cần (`img_019`, `img_021`, `img_025`, `img_030`, `img_032`). |
| **R-14** | **Hàng chở, đồ mang theo** (ống thép `img_029`, bao hàng, hành lý rời) **không tính** vào box xe hay box người. Hàng hóa trên thùng xe tải không mở rộng box ra ngoài thân xe. |
| **R-15** | **Vật nhòe chuyển động:** nhận ra được lớp → gán, box ôm phần thân **đậm nhất** (không bao vệt nhòe), tick `can_review`. Không nhận ra lớp → không gán, ghi vấn đề cấp ảnh (`img_014`, `img_015`). |
| **R-16** | **Thấy quá ít:** chỉ gán khi phần nhìn thấy đủ nhận ra lớp **không cần đoán từ ngữ cảnh**. Chỉ thấy gương / tay lái xe máy ở mép ảnh → không gán (`img_040`). |
| **R-17** | **Xe thật kiểu dáng lạ** (xe độ, xe diễu hành, xe trưng bày): có bánh, kích thước thật, biển số hoặc rõ ràng chạy được → gán theo hình dạng / chức năng gần nhất (`img_019` → `car`, xe trong bảo tàng `img_024` → `car`). Không chắc là xe thật hay tác phẩm (`img_011` dãy xe máy độ) → gán `motorcycle` + `can_review`. |
| **R-18** | **Xe đỗ, xe đang được vận chuyển** vẫn gán nếu là xe thật. |
| **R-19** | **Ảnh không có vật cần gán** (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) vẫn nằm trong Task và trong COCO xuất ra, không có annotation. |
| **R-20** | **Chưa đặt được box** (không chắc vật / vùng) → ghi ảnh, vùng, lý do; nhóm tập hợp vào `submission/qc/image_issues.csv` (cột `image,annotator,region,note`) **sau khi khóa nhãn độc lập**. |

## 4. Ca khó

### 4.1 Giải quyết bất đồng giữa hai bản cá nhân

| # | Vấn đề | 2A202603219 | 2A202603197 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|
| D1 | Xe buýt, tuk-tuk, auto-rickshaw | `ignore_region` | Không gán | **`ignore_region`** (R-10) | Vật thật, lớn, nằm trên đường. Để nền sẽ dạy mô hình đây là "không có gì" và phạt mọi dự đoán trên đó; ignore giữ vùng trung tính. Xe có hình dạng rõ nên hai người đặt vùng dễ trùng nhau (ghép IoU ≥ 0.5). |
| D2 | Người ngồi trong tuk-tuk | Không gán | Gán `person` | **Không gán** (R-10) | Nằm trong vùng ignore; tránh box chồng vùng ignore gây bất đồng `bat_dong_ignore`. |
| D3 | Người trong cabin ô tô | Không gán | Gán `person` | **Không gán** (R-05) | Chỉ thấy qua kính, mô hình phải đoán; vật bao ngoài đã có box xe. |
| D4 | Ngưỡng `occluded_level` | 4 mức <10 / 10–40 / 40–80 / ≥80 | 0 = không che, 1 ≤ 30%, 2 > 30%, 3 = không ước lượng | **4 mức <10 / 10–40 / 40–80 / ≥80 + "không đủ căn cứ" vào mức 3** | Mức 2 của bản 197 trải 31–99% quá rộng; "0 = không che chút nào" khó đồng thuận vì hầu như vật nào cũng bị che vài %. Giữ ý "không đủ căn cứ ước lượng" của bản 197 cho mức 3. |
| D5 | Vật < 20 px | Gom vào `ignore_region` | Bỏ | **Bỏ** (R-12); ignore chỉ dùng khi là cụm dày theo R-13 | Giảm số vùng ignore không ghép được giữa hai người; đúng ngưỡng `min_box_height_px`. |
| D6 | Thuộc tính của `ignore_region` | Không có | Để mặc định 0/no/false | **Không có thuộc tính** | `cvat/labels_generated.json` không cấu hình thuộc tính cho `ignore_region`. |
| D7 | Xe vàng nhỏ trên nóc `img_024` | Không gán | `can_review` | **Không gán** (R-06) | Kích thước nhỏ hơn xe thật rõ rệt, đặt trên giá nóc — là mô hình trưng bày; chốt luôn để không thành ca bất đồng. |
| D8 | Xe chở xe `img_002` | 1 box `truck` cả tổ hợp | "Không gộp cả đoàn" (chưa rõ) | **1 box `truck` gồm đầu xe + rơ-moóc nối liền**; mỗi ô tô trên giá 1 box `car` | Bản 197 có ý "không gộp" là không gộp **các ô tô được chở** vào box xe tải — hai bản thực chất trùng; viết rõ để tránh hiểu 2 cách. |
| D9 | Chỉ thấy gương xe máy `img_040` | Không gán | Gán nếu "đủ điều kiện" | **Không gán** (R-16) | Gương không đủ nhận ra xe máy mà không đoán từ ngữ cảnh. |
| D10 | Đám đông | ≥ 5 vật không tách được → ignore | Tách tối đa, ignore phần không tách, chia vùng nhỏ | **Gộp:** ngưỡng ≥ 5 vật + chỉ bao phần không tách được + chia vùng nhỏ, không bao vật đã gán (R-13) | Ngưỡng số giúp hai người cùng quyết định; ý "chia vùng nhỏ" của bản 197 tránh ignore phủ box. |
| D11 | Chạm biên có phải `truncated` | Box chạm mép và mất phần | Chạm biên không tự động là cắt | **Chỉ `yes` khi thực sự mất phần** | Hai bản cùng ý; lấy câu chữ của bản 197 cho rõ. |

### 4.2 Ca khó theo ảnh

| Ảnh | Tình huống | Rule | Quyết định |
|---|---|---|---|
| img_001 | Người ngồi quán bị bàn/người che; xe tải nhỏ cắt mép phải; banner hot dog | R-04, R-08, R-03, R-06 | Mỗi người rõ → `person` (`occluded_level` theo phần bị che); xe → `truck`, `truncated=yes`; banner → không gán |
| img_002 | Xe chở 7 ô tô, rơ-moóc; công nhân đứng đuôi xe | R-09, R-05 | 1 `truck`; mỗi ô tô → `car`; công nhân → `person`; tài xế trong cabin → không gán |
| img_004 | Hai tuk-tuk có tài xế | R-10 | 2 `ignore_region`; tài xế không gán |
| img_005 | Người đi xe máy chen sát, che lẫn nhau; van bạc phía sau; xe cắt mép trái | R-04, R-08, R-03 | Mỗi người 1 `person`; xe máy thấy được → `motorcycle`; xe bạc → `van`; xe trái → `car`, `truncated=yes` |
| img_006 | Xe máy, xe đạp, taxi | R-04, R-11, R-13 | Người và xe máy riêng; xe đạp không gán, người đạp → `person`; taxi → `car`; khối xe xa → R-13 |
| img_007 | Kho tối: bán tải thùng kín, xe tải Isuzu | R-08, R-15 | 2 `truck`; nếu thấy người khuất giữa hai xe → `person` + `can_review` |
| img_009 | Minibus ~29 chỗ; xe máy có người; người ban công | R-10, R-04, R-12 | Minibus → `ignore_region`; `motorcycle` + `person`; người ban công gán nếu cao ≥ 20 px |
| img_011 | Tượng Predator; dãy xe máy độ trưng bày; van trắng; người xa | R-06, R-17, R-13 | Tượng không gán; xe máy độ → `motorcycle` + `can_review`; van → `van`; người tách được → `person` |
| img_012 | Bán tải chở hàng; SUV; xe tải phía trước | R-08, R-14 | Bán tải → `truck` (không bao hàng nhô ra ngoài thân); SUV → `car`; xe tải → `truck` |
| img_014 | Hai người qua đường rõ; xe máy, người nhòe | R-15 | 2 `person`; vật nhòe nhận ra lớp → gán + `can_review`; không nhận ra → ghi vấn đề ảnh |
| img_015 | Hai xe cắt mép; xe hậu cảnh nhòe; đường ướt phản chiếu | R-03, R-15, R-06 | `car` `truncated=yes`; xe hậu cảnh theo R-15; phản chiếu không gán |
| img_016 | Tuk-tuk ×2, Vespa, bán tải, van, xe đen, người mép dưới | R-10, R-04, R-08, R-03 | Tuk-tuk → `ignore_region`; Vespa → `motorcycle` + `person`; bán tải → `truck`; van → `van`; xe đen → `car`; người áo đỏ → `person`, `truncated=yes` |
| img_017 | Tượng người cưỡi ngựa | R-06, R-19 | Ảnh trống |
| img_018 | Biển quảng cáo có người; ô tô sau hàng rào | R-06, mục 2.2 | Người trên biển không gán; ô tô → `car`, thường `occluded_level=1` |
| img_019 | Xe vàng diễu hành có biển số, búp bê chuột, tài xế; đám đông sau rào | R-17, R-06, R-05, R-13 | Xe → `car`; búp bê, tài xế không gán; người tách được → `person`; khối đông → `ignore_region` |
| img_020 | Phản chiếu trong gương xe chủ | R-07, R-19 | Ảnh trống |
| img_021 | Dòng xe máy rất dày | R-04, R-13, R-12 | Tiền cảnh: `motorcycle` + `person` riêng; khối xa không tách được → `ignore_region` |
| img_022 | Mô tô sidecar, ảnh đen trắng | R-04, mục 2.1 | 1 `motorcycle` gồm sidecar; người lái → `person` |
| img_023 | Xích lô có người đạp; ô tô đen; xe máy xa | R-11, R-04 | Xích lô không gán; người đạp → `person`; `car`; `motorcycle` + `person` |
| img_024 | Bảo tàng xe; xe vàng nhỏ trên nóc | R-17, R-06, R-03 | Xe thật → `car` (xe mép ảnh `truncated=yes`); xe vàng nhỏ không gán |
| img_025 | Xe máy kéo thùng; người đi bộ; ghế bãi biển đông người; taxi; bán tải mép phải | R-04, R-13, R-08, R-03 | Xe + thùng → `motorcycle`; người → `person`; khối người ngồi xa → `ignore_region`; taxi → `car`; bán tải → `truck`, `truncated=yes` |
| img_026 | Hai người trên một xe máy; tranh tường | R-04, R-06 | 2 `person` + 1 `motorcycle`; tranh tường không gán |
| img_027 | Tượng đá bán tải | R-06, R-19 | Ảnh trống |
| img_028 | Bán tải chở tượng ngựa; SUV đỏ | R-08, R-06 | `truck`; `car`; ngựa không gán |
| img_029 | Xe máy điện chở ống thép; người đạp xe đạp; người đi bộ | R-14, R-11, R-04 | `motorcycle` (không bao ống) + `person`; xe đạp không gán, người → `person` |
| img_030 | Bãi xe máy đỗ dày; người áo xanh | R-13, R-18 | Xe tiền cảnh tách được → `motorcycle`; phần dày không tách được → `ignore_region`; người → `person` |
| img_031 | Phố đêm; xe buýt; đường ướt | R-10, R-06 | `car`, `motorcycle`, `person`; xe buýt → `ignore_region`; phản chiếu không gán |
| img_032 | Đám đông chờ xe buýt; xe xanh cắt mép trái | R-13, R-03, R-06 | Người tách được → `person`; phần chồng kín → `ignore_region`; xe → `car`, `truncated=yes`; poster nhà chờ không gán |
| img_033 | Ma-nơ-canh, phản chiếu | R-06, R-19 | Ảnh trống |
| img_035 | Auto-rickshaw chở hàng; rickshaw thứ hai; xe máy; ô tô | R-10, R-04 | 2 `ignore_region`; `motorcycle` + `person`; `car` |
| img_036 | Xe tải nhỏ mui bạt; xe máy đỗ; poster | R-08, R-13, R-06 | `truck`; xe máy đỗ tách được → `motorcycle`; poster không gán |
| img_037 | Bán tải gắn khoang cao; xe buýt; người đi bộ | R-08, R-10 | `truck`; xe buýt → `ignore_region`; `person` |
| img_038 | Tủ kính đồ chơi | R-06, R-19 | Ảnh trống |
| img_040 | Người đội mũ cận cảnh; gương xe máy; taxi; tuk-tuk; xe buýt; SUV | R-03, R-16, R-10 | `person` `truncated=yes`; gương không gán; taxi → `car` `truncated=yes`; tuk-tuk, xe buýt → `ignore_region`; SUV → `car` |

## 5. QC và phiên bản

**Ngưỡng QC** — giữ nguyên mục `rules` trong `submission/taxonomy.json`:

| Tham số | Giá trị | Dùng cho |
|---|---:|---|
| `min_box_height_px` | 20 | R-12 |
| `match_iou` | 0.50 | ghép cặp box giữa hai người |
| `iou_thr_large` / `large_px` | 0.85 / 50 | box cao ≥ 50 px |
| `iou_thr_small` | 0.70 | box cao < 50 px |
| `iaa_iou` | 0.85 | IoU trung bình đạt |
| `iaa_class` | 0.95 | đồng thuận lớp đạt |
| `error_rate_img_max` | 0.02 | Release: ≤ 2% ảnh lỗi trên toàn lô (1/40 = 2,5% đã vượt) |

**Quy trình:**
- Mỗi người gán độc lập theo v1.0 đã khóa, xuất COCO, chạy `validate_labels.py`, gửi riêng; chỉ so sánh khi cả hai xong.
- Đọc IAA kèm mẫu số (số cặp ghép được), box chưa ghép, bất đồng vùng ignore. Đồng thuận cao vẫn có thể **cùng sai** → đối chiếu ca tham chiếu của coach.
- Ca mơ hồ chốt ở calibration (Pha 5): mỗi ca một dòng trong `calibration_decisions.csv`, có người chốt, người kiểm lại. Điều phối: 2A202603219; review bản cuối: 2A202603197. Ai sửa ca nào thì người kia kiểm lại, ghi rõ.
- Đổi câu chữ, không đổi nhãn → `editorial`. Đổi cách chọn vật / lớp / box / thuộc tính → `semantic`, tăng phiên bản, rà lại cả lô.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 (×2) | Hai bản cá nhân | xem `individual/` |
| **v1.0** | Hợp nhất hai bản; chốt D1–D11; định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng `occluded_level` 4 mức; quy tắc R-01…R-20 | Xe ngoài taxonomy `img_004`, `img_009`, `img_035`; người trong cabin `img_019`; vật nhỏ và đám đông `img_021`, `img_030`, `img_032`; xe chở xe `img_002`, `img_024`; gương `img_040` |
