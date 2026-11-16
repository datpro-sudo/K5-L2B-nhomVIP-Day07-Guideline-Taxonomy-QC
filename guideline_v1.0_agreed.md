# Guideline gán nhãn chung — v1.0

- Phiên bản: `v1.0` — bản tổng hợp từ 4 guideline cá nhân.
- Loại nhãn: bounding box trên ảnh đường phố (40 ảnh `img_001`–`img_040`).
- Công cụ: CVAT, xuất COCO 1.0.
- Nguồn tổng hợp: guideline của Nguyễn Văn Nhất (2A202603278), Nguyễn Tiến Đạt (2A202603289), Lê Minh Thắng (2A202603292) và Trần Anh Tùng (2A202603280).
- Cách gộp: quy tắc nào ít nhất 3/4 bản đồng ý thì đưa vào thẳng; quy tắc các bản khác nhau thì ghi ở **mục 6** kèm lựa chọn mặc định đề xuất. Trong lúc chưa họp xong, các mục có dấu ⚠️ là mặc định tạm thời.
- Lưu ý khi dùng: các ca theo ảnh ở mục 4 là đề xuất dựa trên mô tả của từng người, **phải mở ảnh gốc kiểm lại** trước khi gán, nhất là vật nhỏ ở xa.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố (camera giao thông, xe tự lái).

Các lỗi cần tránh nhất:

1. Bỏ sót người hoặc xe (kể cả ở xa, ở mép ảnh, trong đám đông).
2. Gộp người với xe, hoặc gộp nhiều vật vào một box.
3. Gán nhầm vật không thật (tượng, ma-nơ-canh, ảnh in, đồ chơi, phản chiếu).
4. Nhầm loại xe (car / van / truck).
5. Box quá rộng vì bao cả hàng hóa hoặc vật che.

Một vật chỉ được gán khi: (a) thuộc taxonomy, (b) là vật thật, (c) nhận diện được là một cá thể riêng, (d) phần nhìn thấy cao tối thiểu 20 px trên ảnh gốc. Trạng thái đang chạy / đang đỗ / đang được chở **không** ảnh hưởng việc có gán hay không và không quyết định lớp.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Giữ đúng 6 lớp trong `submission/taxonomy.json`. Không tự thêm hay đổi tên lớp khi gán (xem mục 6, điểm 1).

| Lớp | Định nghĩa | Ví dụ | Dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, wagon, SUV, MPV thân xe con, taxi. | img_015, img_040, wagon đen img_024 | Pickup có thùng riêng → `truck` ⚠️; xe buýt → ngoài taxonomy. |
| `truck` | Xe có cabin và khu vực chở hàng **tách biệt về cấu trúc**: thùng hở, thùng kín gắn sau cabin, pickup ⚠️, xe chở xe. | img_002, img_012, img_036 | Van thân liền khối; không phân loại theo kích thước. |
| `van` | Xe thân hộp, khoang sau liền thân, chở hàng hoặc ít người. | img_037, xe phía sau img_005 | MPV kiểu ô tô con → `car`; xe tải thùng gắn riêng → `truck`; xe buýt lớn → ngoài taxonomy. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô. Box chỉ bao phần xe. | img_006, img_008, img_026 | Xe đạp, xích lô, tuk-tuk/xe ba bánh → ngoài taxonomy ⚠️. |
| `person` | Người thật ở mọi tư thế: đi bộ, đứng, ngồi, người lái, hành khách, người đi xe đạp. Mỗi người một box. | img_014, img_034, img_006 | Tượng, ma-nơ-canh, người trên ảnh in/banner, người chỉ thấy qua phản chiếu. |
| `ignore_region` | Vùng hẹp có đối tượng đích nhưng **không thể gán từng vật đáng tin cậy** (quá nhỏ, quá che, không tách được cá thể). | Phần đám đông xa ở img_021 | Không phải lớp chứa mọi vật ngoài taxonomy; không bao cả ảnh hoặc cả đoàn xe. |

**Phân lớp xe (thứ tự xét):** (1) cấu trúc cabin/thùng → (2) kiến trúc thân xe → (3) chức năng thể hiện được trong ảnh. Màu sơn, biển quảng cáo, nhãn hiệu, việc đang chở hàng đều **không** đủ để quyết định lớp. Chỗ giáp ranh car/van hoặc van/bus chưa đủ bằng chứng thì tick `can_review`, không tự suy ra số ghế hay tải trọng.

**Ngoài taxonomy (xe đạp, xích lô, tuk-tuk/xe ba bánh, xe buýt lớn, động vật, xe đẩy):**
- Không vẽ box cho chính phương tiện đó.
- Người thật ngồi/lái trên đó vẫn được gán `person` bình thường.
- Chỉ dùng `ignore_region` khi vùng đó có đối tượng đích mà không tách được; **không** tạo ignore chỉ vì vật nằm ngoài lớp.

### 2.2. Thuộc tính (cho 5 lớp đối tượng; `ignore_region` không có thuộc tính)

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| | `1` | Bị che trên 0% đến 30%. ⚠️ |
| | `2` | Bị che trên 30% đến 60%. ⚠️ |
| | `3` | Bị che trên 60%, nhưng **vẫn nhận ra được lớp và cá thể**. ⚠️ |
| `truncated` | `no` / `yes` | `yes` khi một phần vật thực sự nằm ngoài khung ảnh. Chạm sát biên mà vật còn trọn vẹn → `no`. |
| `can_review` | tick / không | Tick khi chưa chắc lớp, ranh giới cá thể, mức che, thật/giả hoặc cách áp dụng quy tắc. Ghi lý do cụ thể, không dùng để né quyết định. |

Lưu ý áp dụng:
- Mức che chỉ tính phần bị **vật khác** che trong phần vật nằm trong ảnh. Phần bị mép ảnh cắt ghi bằng `truncated`, **không** tính vào mức che. Một vật có thể vừa `truncated=yes` vừa `occluded_level>0`.
- Không coi phần vốn không nhìn thấy do góc chụp (mặt sau xe) là bị che.
- Mờ, tối, nhòe **không** đồng nghĩa với che khuất; không dùng mức 3 thay cho "không biết". Không ước lượng được mức che → chọn mức gần nhất và tick `can_review`.
- Cờ `occluded` có sẵn của CVAT không thay cho thuộc tính `occluded_level`.
- Tỉ lệ che chỉ là ước lượng; ca sát mốc thì ghi lý do.

## 3. Quy tắc vẽ box

**R-01 — Phạm vi.** Chỉ gán vật vật lý thật thuộc 5 lớp đích. Đang đi, đang đỗ, trong bãi, trong nhà, đang được vận chuyển đều xét theo cùng tiêu chí.

**R-02 — Một vật, một box.** Mỗi cá thể một box; không gộp nhiều người hoặc nhiều xe vào một box, kể cả khi đứng sát nhau hoặc cùng màu. Không để một xe có hai box.

**R-03 — Box ôm phần nhìn thấy.** Hình chữ nhật song song trục ảnh, nhỏ nhất bao tất cả phần nhìn thấy của **cùng một vật**.
- Không đoán và không kéo box tới phần bị che hoặc nằm ngoài ảnh.
- Các phần nhìn thấy rời nhau nhưng chắc chắn cùng một vật → một box bao chung (khoảng trống ở giữa nằm trong box được).
- Box nằm trong kích thước ảnh gốc, rộng và cao > 0.
- **Xe:** bao thân, bánh, gương và bộ phận gắn cố định nhìn thấy (ghế phụ gắn xe, mũ treo vào xe...). Không bao bóng đổ, phản sáng trên đường, khói, hàng hóa rời.
- **Người:** bao đầu/mũ đang đội, thân, tay chân nhìn thấy; quần áo đang mặc thuộc người. Không mở rộng theo túi xách, quang gánh, cây dài đang mang.

**R-04 — Vật bị che.** Vẫn gán nếu nhận ra được lớp và cá thể; box ôm phần nhìn thấy; điền `occluded_level`. Chỉ thấy mảnh quá mơ hồ → không đoán lớp, ghi vấn đề cấp ảnh (R-12).

**R-05 — Cắt mép ảnh.** Vẽ tới biên trong ảnh, `truncated=yes`, không kéo box ra ngoài ảnh.

**R-06 — Tách người và xe.** Không có lớp "rider". Mỗi người một box `person`, mỗi xe đích một box theo lớp của xe; hai box có thể giao nhau.
- Xe máy + người lái + người ngồi sau → **3 box** (nếu cả ba nhận diện được và đạt 20 px).
- Người đi xe đạp / ngồi xích lô / ngồi tuk-tuk → 1 box `person`, không box phương tiện.
- Người nhìn thẳng qua kính trong suốt vẫn là người thật; hình phản chiếu trên kính thì không.
- Không suy ra số xe từ số mũ bảo hiểm.

**R-07 — Xe chở xe.** Một box `truck` cho xe vận chuyển (cabin + bệ chở), không mở rộng để ôm hàng. Mỗi ô tô thật được chở là một box `car` riêng nếu nhận diện được; box `car` chỉ ôm chính chiếc xe, không ôm giá đỡ của xe tải. Không đếm cứng số xe khi phần xa chưa rõ.

**R-08 — Vật không thật và phản chiếu.** Không gán: tượng (kể cả tượng người cưỡi ngựa, tượng hình pickup), ma-nơ-canh, đồ chơi/mô hình, ảnh in/biển quảng cáo/poster/màn hình, vật chỉ xuất hiện trong gương/kính/mặt đường ướt. Không tạo `ignore_region` cho chúng chỉ vì giống lớp đích. Xe thật đang trưng bày hoặc đang được chở vẫn gán. Xe có ngoại hình lạ (hoạt hình, xe độ) **không** tự bị loại; nếu chưa phân biệt được xe thật với đạo cụ/mô hình thì tick `can_review` và đưa họp, không kết luận chỉ từ ngoại hình.

**R-09 — Xe chủ camera.** Không gán phần xe đang mang camera (kể cả bộ phận nhìn trực tiếp và ảnh của nó trong gương) khi xác định chắc chắn. Không suy ra "xe chủ" chỉ vì xe nằm sát máy ảnh. Xe khác thấy qua/ngoài gương vẫn xét bình thường.

**R-10 — Hàng hóa, phụ kiện.** Hàng hóa, thùng giấy, ống sắt, túi, vật cầm tay, hàng rong không có box riêng và không làm mở rộng box của người/xe. Chỉ bộ phận **gắn cố định** vào xe mới thuộc box xe.

**R-11 — Vật nhỏ, mờ, đám đông.**
- Đo `h = ymax − ymin` của box phần nhìn thấy **trên ảnh gốc** (không đo trên thumbnail). `h ≥ 20 px` → xét gán cá thể; `h < 20 px` → không gán box lớp đối tượng, khoanh `ignore_region` hẹp nếu có vật đích.
- Mờ/tối/xa: gán khi vẫn chắc chắn nhận ra lớp và biên; không đoán biên theo vệt nhòe hay bóng sáng; không chắc thì không gán và ghi vấn đề (R-12).
- Đám đông / bãi xe: (1) gán mọi cá thể nhận diện được và đạt ngưỡng, kể cả khi sát nhau; (2) chỉ khoanh `ignore_region` phần **còn lại** không tách được; (3) nếu một hình chữ nhật ignore phủ cả vật cần gán thì chia nhỏ vùng; (4) không phủ ignore lên vật rõ, không bao cả ảnh/cả dòng xe, không tạo box giả định trong vùng chỉ có vài nét mờ.

**R-12 — Khi chưa chắc.**
- Đã vẽ được box nhưng nghi lớp/thuộc tính: chọn phương án dự kiến, tick `can_review`, ghi lý do.
- Chưa vẽ được box đáng tin cậy hoặc chưa biết có vật đích hay không: ghi một dòng vấn đề cấp ảnh `image, annotator, region, note` (để thu vào `submission/qc/image_issues.csv` sau khi khóa nhãn). Không tạo nhãn rỗng, không điền mã của người khác.

## 4. Ca khó theo ảnh

Cột "Chốt" cho biết mức đồng thuận giữa các bản: ✅ các bản đồng ý, ⚠️ còn lệch (xem mục 6). Mọi ca cần mở ảnh gốc xác nhận lại.

| Ảnh | Tình huống | Rule | Quyết định đề xuất | Chốt |
|---|---|---|---|---|
| img_001 | Pickup, người gần thùng xe, có che/cắt mép. | R-03, R-04, R-05, R-10 | Pickup = `truck` ⚠️; người đủ rõ = `person`; không mở box theo hàng. | ⚠️ (pickup) |
| img_002 | Xe chở xe chở nhiều ô tô. | R-07 | 1 `truck` + mỗi ô tô nhận diện được 1 `car`. | ✅ |
| img_003 | Người gánh hàng, xe đạp, xe máy dựng sát. | R-06, R-10 | Tách người và xe máy; không bao quang gánh; xe đạp không gán. Vùng không tách được → ignore hẹp. | ⚠️ (bicycle) |
| img_004 | Tuk-tuk/xe ba bánh. | R-06, mục 6 | Không gán xe ba bánh; người trong xe nhìn rõ vẫn gán `person`. | ⚠️ |
| img_005 | Nhiều người/xe chồng lấp, người cắt mép dưới, van phía sau. | R-02, R-04, R-05, R-06 | Tách từng người, từng xe nhận diện được; người bị cắt `truncated=yes`; van không gộp với xe máy. | ✅ |
| img_006 | Hai người trên xe máy bên trái; người đi xe đạp bên phải. | R-06 | 2 `person` + 1 `motorcycle`; người xe đạp = `person`, xe đạp không gán. | ⚠️ (bicycle) |
| img_007 | Pickup/xe tải có kết cấu chở hàng, khu vực tối. | R-03, R-11 | Xe có thùng riêng = `truck`; box theo phần kết cấu gắn liền. | ✅ |
| img_008 | Xe máy đỗ sát nhau, ban đêm, nhiều mũ treo. | R-02, R-10, R-11 | Tách từng xe nếu thấy biên; mũ treo gắn vào xe thuộc box xe; ignore chỉ phần không tách được. | ✅ |
| img_009 | Xe chở khách thân dài, khó phân van/bus. | R-12 | **Cần họp.** Thân van → `van`; xe buýt → ngoài taxonomy. | ⚠️ |
| img_010 | Người ngồi xe máy cạnh xe hàng/xe đạp chở hàng. | R-06, R-10 | `person` + `motorcycle` riêng; xe đạp không gán; không bao xe hàng vào box người. | ✅ |
| img_011 | Tượng/mô hình hình sinh vật, người xa. | R-08, R-11 | Loại vật trang trí; người thật ở xa xét theo chiều cao gốc. | ✅ |
| img_012 | Pickup chở thùng/túi hàng, cắt mép; có xe không rõ phía trước. | R-03, R-05, R-10 | Pickup = `truck` ⚠️; box không mở theo kiện hàng; xe không rõ/ngoài taxonomy không gán. | ⚠️ (pickup) |
| img_013 | Đường tối, vật nhỏ ở xa. | R-11 | Gán vật đủ rõ và ≥ 20 px; ignore hẹp phần còn lại. | ✅ |
| img_014 | Người rõ giữa xe máy nhòe chuyển động. | R-11 | Gán người/xe nhận diện chắc; mờ ≠ che; không đoán từ vệt màu. | ✅ |
| img_015 | Ô tô sát hai mép, phản chiếu mặt đường ướt, xe sau nhòe. | R-05, R-08, R-11 | Ô tô thật `truncated=yes`; không gán xe phản chiếu; không tạo box trùng. | ✅ |
| img_016 | Tuk-tuk, van, xe máy, người chồng lấn. | R-02, R-04, R-06 | Tách từng vật; tuk-tuk không gán ⚠️; van, xe máy, người gán riêng. | ⚠️ (tuk-tuk) |
| img_017 | Tượng người cưỡi ngựa. | R-08 | Không gán. | ✅ |
| img_018 | Chân dung in trên biển; ô tô sau hàng rào. | R-04, R-08 | Bỏ hình in; ô tô thật xét riêng với `occluded_level` do hàng rào; không ignore cả bãi xe. | ✅ |
| img_019 | Xe vàng dạng hoạt hình, nhân vật hóa trang, đám đông. | R-08, R-11 | **Cần họp:** xe thật hay đạo cụ? Nếu thật → `car`/lớp theo kết cấu; đầu nhân vật giả không gán; người thật vẫn `person`; ignore vùng đám đông không tách được. | ⚠️ |
| img_020 | Gương chiếu hậu, thân xe mang camera. | R-08, R-09 | Không gán xe chủ và xe trong gương; vẫn kiểm vật đích nhìn trực tiếp ngoài gương; không biến cả mặt gương thành ignore. | ✅ |
| img_021 | Đám đông người đi xe máy, vật xa rất nhỏ. | R-06, R-11 | Gán từng người/xe rõ và ≥ 20 px; ignore hẹp phần xa không tách được; không một ignore bao cả dòng xe. | ✅ |
| img_022 | Xe máy/người rất xa (một bản mô tả có ghế phụ gắn bên xe). | R-03, R-11 | Đo trên ảnh gốc; gán nếu ≥ 20 px và rõ lớp, nếu không ignore hẹp. Ghế phụ **gắn cố định** thuộc box xe máy. | ⚠️ (kiểm lại ảnh) |
| img_023 | Người đạp xích lô. | R-06 | `person` cho người; xích lô không gán; ô tô/xe máy khác xét riêng. | ✅ |
| img_024 | Vật hình xe vàng/khung xe trên nóc wagon đen. | R-03, R-08, R-12 | Wagon = `car`. **Cần họp:** vật trên nóc là xe thật chưa hoàn thiện hay vỏ/mô hình? Thật → gán riêng; mô hình → không gán, không mở box wagon theo vật đó. | ⚠️ |
| img_026 | Hai người trên xe máy, ảnh tối/mờ. | R-06, R-11 | 2 `person` + 1 `motorcycle` khi đủ nhận diện; ảnh kiểm chéo với img_006. | ✅ |
| img_027 | Bia/tượng hình pickup trong nghĩa trang. | R-08 | Không gán `truck`/`car`. | ✅ |
| img_028 | Pickup thật chở tượng ngựa, ô tô đỏ phía sau. | R-03, R-08, R-10 | Pickup = `truck` ⚠️; không bao tượng ngựa; tượng không phải `person`; ô tô sau gán riêng. | ⚠️ (pickup) |
| img_029 | Người/xe chở thanh dài che ngang ảnh. | R-03, R-04, R-10 | Không mở box theo thanh; đánh giá che khuất riêng cho vật bị thanh chắn. | ✅ |
| img_030 | Bãi xe máy đỗ sát, che nhau. | R-01, R-02, R-11 | Gán từng xe tách được (không bỏ vì đang đỗ); không gán 2 box cho 1 xe; ignore chỉ phần không tách được. | ✅ |
| img_032 | Đám đông bên kia đường, ô tô tiền cảnh cắt mép. | R-05, R-11 | Người đủ điều kiện gán riêng; vùng đông không tách được → ignore hẹp; ô tô cắt mép `truncated=yes`. | ✅ |
| img_033 | Ma-nơ-canh ở cửa hàng. | R-08 | Không gán. | ✅ |
| img_035 | Xe ba bánh chở tải cao, người đi xe máy phía trước. | R-06, mục 6 | Xe ba bánh không gán (hàng cao không biến thành `truck`); người + xe máy đủ điều kiện gán riêng. | ⚠️ |
| img_036 | Xe tải rất nhỏ, thùng kín gắn sau cabin. | R-02 | `truck` (xét kết cấu cabin/thùng, không xét kích thước). | ✅ |
| img_037 | Van hàng thân liền; xe buýt bên trái. | R-02, mục 6 | `van` cho xe tiền cảnh; bus không gán; người cạnh bus xét riêng. | ✅ |
| img_038 | Ô tô đồ chơi trong tủ. | R-08 | Không gán; không dùng ignore thay quyết định. | ✅ |
| img_040 | Người đội mũ che mặt, taxi, nhiều xe nền. | R-02, R-06, R-11 | Người vẫn `person` dù không thấy mặt; taxi = `car`; xe nền xét che/kích thước. | ✅ |

Các ảnh chưa có trong bảng (img_025, 031, 034, 039) chưa bản nào mô tả; người gán áp dụng R-01 → R-12 và tick `can_review` khi gặp ca mới, sau đó bổ sung vào bản v1.1.

## 5. Checklist tự kiểm trước khi xuất COCO

- [ ] Đã quét ảnh từ trái sang phải **trước khi bật box**, ghi vị trí mọi người/xe đích, rồi đối chiếu với nhãn.
- [ ] Không bỏ sót người/xe ở mép ảnh, góc ảnh, phía xa, trong đám đông.
- [ ] Mỗi cá thể đúng một box; không box trùng; không gộp nhiều vật.
- [ ] Box ôm phần nhìn thấy, không bao hàng hóa/vật che, không cắt mất đầu/bánh/thân.
- [ ] Người và xe có box riêng; không có box trên tượng, ma-nơ-canh, ảnh in, đồ chơi, phản chiếu.
- [ ] Phân lớp xe theo cấu trúc, không theo kích thước/màu/quảng cáo.
- [ ] `occluded_level`, `truncated` đúng định nghĩa (cắt mép ≠ che khuất; mờ ≠ che khuất).
- [ ] Vật < 20 px (đo trên ảnh gốc) không có box lớp đối tượng; `ignore_region` hẹp và không phủ vật đã gán.
- [ ] Ca chưa chắc đã tick `can_review` và ghi lý do; ca chưa vẽ được box đã ghi vấn đề cấp ảnh.
- [ ] Đã kiểm cả ảnh không có đối tượng; box nằm trong biên ảnh; chạy `python scripts/validate_labels.py <file-coco>`.

## 6. Điểm chưa thống nhất — cần họp chốt

Đây là những chỗ 4 bản khác nhau. Cột cuối là mặc định tạm thời của bản tổng hợp này; nhóm bỏ phiếu rồi ghi kết quả vào v1.0.

| # | Vấn đề | Các bản nói gì | Mặc định đề xuất |
|---|---|---|---|
| 1 | **Có mở rộng taxonomy** (thêm `bicycle`, `bus`, `three_wheeler`) không? | Đạt: thêm `bus`, `bicycle`, gộp tuk-tuk vào `motorcycle`. Thắng: đề xuất thêm `bicycle`, `bus`, `three_wheeler`. Nhất và Người 3: giữ 6 lớp, xe ngoài taxonomy không gán. | Giữ 6 lớp theo `taxonomy.json`; sửa taxonomy phải làm **cùng lúc** với guideline và trước khi gán. Người trên các xe đó vẫn gán `person`. |
| 2 | **Pickup** thuộc lớp nào? | Nhất, Thắng, Đạt: `truck`. Người 3: `car` (ghi là theo taxonomy). | `truck`. **Việc cần làm:** mở `taxonomy.json` kiểm tra định nghĩa `car`/`truck`; nếu taxonomy ghi pickup thuộc `car` thì phải theo taxonomy. |
| 3 | **Mốc `occluded_level`.** | Nhất, Người 3: 30% / 60%. Thắng, Đạt: 30% / 70%. Người 3 còn tách mức 0 (0–10%). | 0 / ≤30% / ≤60% / >60%; mức 0 = không che. |
| 4 | **Mức 3 khi không ước lượng được.** | Nhất: mức 3 + tick `can_review`. Thắng: không dùng mức 3 thay "không biết". | Mức 3 chỉ khi che > 60% mà vẫn nhận ra; không ước lượng được → tick `can_review`. |
| 5 | **Vật < 20 px.** | Nhất, Thắng: không box, ignore hẹp. Người 3: không gán nếu không chắc. Đạt: vẫn gán nếu nhận ra. | Theo `taxonomy.rules`: `h < 20 px` → không box, ignore hẹp. |
| 6 | **Ranh giới van / bus** (img_009, img_037). | Nhất: van nếu thân van, bus ngoài taxonomy. Thắng: bus. Đạt: có lớp bus. | Theo kết quả điểm 1. Nếu giữ 6 lớp: thân van rõ → `van`, xe khách dài nhiều hàng ghế → không gán. Cần chỉ ra dấu hiệu phân biệt. |
| 7 | **Xe thật hay mô hình** (img_019, img_024). | Cả nhóm chưa chốt. | Nhóm xem ảnh gốc và nguồn ảnh; chưa rõ thì tick `can_review`. |
| 8 | **Người rất nhỏ hoặc chỉ thấy một phần** (Đạt: cân nhắc ignore). | Các bản khác: dùng ngưỡng 20 px, mức che và R-11. | Áp dụng R-04 + R-11, không có ngưỡng "thấy ≥ 30%" riêng. |
| 9 | **Ranh giới `ignore_region`** (img_021, img_030). | Các bản đồng ý ignore phải hẹp, nhưng chưa ai chỉ rõ ranh giới. | Cả nhóm thử khoanh ignore trên cùng ảnh và so lại. |

## 7. QC và phiên bản

### 7.1. Ngưỡng cố định (lấy từ `taxonomy.json`, không sửa mục `rules`)

| Trường | Giá trị | Cách áp dụng |
|---|---:|---|
| `min_box_height_px` | 20 | Chiều cao box phần nhìn thấy trên ảnh gốc. |
| `match_iou` | 0.5 | Ghép box 1–1 không phân biệt lớp khi so sánh. |
| `large_px` | 50 | Box cao ≥ 50 px thuộc nhóm lớn. |
| `iou_thr_large` | 0.85 | Ngưỡng so box lớn. |
| `iou_thr_small` | 0.70 | Ngưỡng so box nhỏ (20 đến < 50 px). |
| `iaa_iou` | 0.85 | IoU trung bình trên các cặp ghép. |
| `iaa_class` | 0.95 | Tỉ lệ cùng lớp trên các cặp ghép. |
| `error_rate_img_max` | 0.02 | Tỉ lệ ảnh lỗi tối đa cho Release. |

- Luôn báo tử số/mẫu số; IAA ghi N/A khi không có cặp ghép. Box không ghép được và bất đồng ignore phải đọc riêng.
- IAA cao **không** chứng minh nhãn đúng (hai người cùng bỏ sót hoặc cùng sai lớp vẫn có IAA tốt).
- Với 40 ảnh, **1 ảnh lỗi = 1/40 = 2,5%**, đã vượt ngưỡng 2%.

### 7.2. Quy trình

1. **Gán độc lập (Pha 3):** mỗi người dùng đúng guideline đã khóa, tự kiểm theo mục 5, xuất COCO. Không xem nhãn của thành viên khác, không tự sửa quy tắc dựa vào nhãn người khác.
2. **Sau khi khóa nhãn:** cả nhóm tính IAA và calibration. Người 1 cập nhật guideline; Người 3 ghi quyết định và sửa nhãn.
3. **QC (Người 4 tổ chức):** duyệt toàn bộ 40 ảnh trên nhãn cuối; reviewer khác người sửa. Phiếu mỗi vòng tại `submission/qc/reviews/round-NN/qc_review.csv` (do script tạo).
   - Xem ảnh gốc trước (chưa bật box) để tìm vật bị bỏ sót, sau đó mới kiểm từng box về lớp, độ sát, thuộc tính; kiểm lại vùng `ignore_region` và mép ảnh.
   - Ghi lỗi theo thứ tự: **ảnh → vị trí/object_id → lỗi → Rule ID → đề xuất sửa**. Ví dụ lỗi cần mô tả đủ để người sửa tìm đúng vật ("người áo đỏ bên trái xe máy, ngoài cùng"). Còn câu hỏi chưa giải quyết thì ghi câu hỏi, không đổi thành `khong` để hoàn tất phiếu.
   - Báo riêng: ảnh lỗi/ảnh đã duyệt, annotation lỗi/annotation đã duyệt, số vật bỏ sót. Không điền `khong` trước khi xem ảnh, không tự sinh số liệu QC.
4. **Quyết định:** *Hold* khi thiếu bằng chứng, còn ca chưa chốt hoặc mới lấy mẫu; *Rework* khi ảnh lỗi vượt 2% hoặc có lỗi lặp theo script; *Release* khi bằng chứng đầy đủ và reviewer xác nhận.
5. Script kiểm định dạng và số đo; reviewer đọc ảnh để bắt lỗi script không thấy (bỏ sót chung, nhãn giả, box không sát, áp dụng sai phạm vi).

### 7.3. Quản lý phiên bản

- Thay đổi nào làm đổi cách chọn vật, lớp, biên box hoặc thuộc tính đều là thay đổi ngữ nghĩa: tăng phiên bản **và rà lại cả lô**, không chỉ ảnh đang tranh luận.
- Bản cá nhân được giữ nguyên sau họp; quyết định nhóm ghi vào v1.0, điều chỉnh sau calibration ghi vào v1.1.

| Phiên bản | Thay đổi | Cơ sở / trạng thái |
|---|---|---|
| v0 (giảng viên) | Lớp và quy tắc khởi đầu, chưa có ca khó. | File nguồn. |
| v0.1 / v0-individual | 4 bản cá nhân: Nhất (40 ảnh), Thắng (img_001–020), Người 3 (img_002–024), Đạt (bản ngắn). | Chưa nhóm nào duyệt. |
| v1.0-draft (bản này) | Gộp phần đồng thuận, tách 9 điểm cần họp, 12 quy tắc R-01–R-12, checklist, quy trình QC. | Chờ họp nhóm để khóa v1.0. |
