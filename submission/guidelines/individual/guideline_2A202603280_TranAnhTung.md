# Guideline gán nhãn — bản cá nhân nháp — Người 3

> Trạng thái: **guideline cá nhân đã chốt**. Đổi tên file thành
> `guideline_<mã-ba-chữ-số>.md` trước khi nộp. Đây là quyết định cá nhân trước
> buổi họp thống nhất guideline v1.0.

- Phiên bản: v0-individual
- Ngày: 2026-11-16
- Người viết: Người 3 (chưa có mã thành viên)
- Phạm vi: bounding box trên ảnh đường phố, xuất CVAT/COCO 1.0.

## 1. Mục tiêu

Tạo nhãn nhất quán cho mô hình phát hiện người và phương tiện trong ảnh 2D. Một
vật chỉ được gán khi thuộc taxonomy, nhận diện được như một thực thể riêng, và
quy tắc bên dưới cho phép gán. Mục tiêu không phải là đoán tình trạng chuyển động
của vật từ một ảnh tĩnh: taxonomy hiện hành không có thuộc tính `moving`/`parked`.

Sai có rủi ro cao: gán thiếu vật hợp lệ, nhầm lớp phương tiện, gộp nhiều vật thành
một box, hoặc dùng box quá rộng do lấy cả vật che/phụ kiện/hàng hoá của vật khác.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa theo taxonomy | Ví dụ | Không phải lớp này / cần xử lý riêng |
|---|---|---|---|
| `car` | Ô tô con, SUV, pickup. | sedan, hatchback, SUV, pickup | xe tải, van, xe máy. |
| `truck` | Xe tải, xe có thùng hàng riêng. | xe tải có cabin và thùng hàng tách biệt | car, van; xe ba bánh/chở hàng lạ là ca R-08. |
| `van` | Xe thùng kín cỡ nhỏ. | xe giao hàng thân kín | xe tải thùng hở, xe ba bánh, car/SUV. |
| `motorcycle` | Xe máy, mô tô. | xe máy có/không người lái | xe đạp, xích lô, xe ba bánh chở hàng nếu taxonomy không mở rộng. |
| `person` | Người đi bộ hoặc người ngồi trên xe. | người đi bộ, người lái/ngồi sau xe máy | tượng, mannequin, người trên banner, phản chiếu nếu không được R-06/R-07 cho phép. |
| `ignore_region` | Vùng không gán từng vật; bỏ sót trong vùng không tính lỗi. | đám đông quá dày, vùng xe quá sít không thể tách biên từng vật | Không dùng thay cho box của vật tách được. |

**Ngoài taxonomy.** Taxonomy hiện không có lớp cho xe đạp, xích lô, xe ba bánh chở
hàng, hàng rong hay lớp “phương tiện khác”. Không tự đổi tên lớp, tự thêm lớp hay
ép chúng vào `car`/`truck`/`van`/`motorcycle`. Với một phương tiện ngoài taxonomy
tách được: không gán box. Với một cụm ngoài taxonomy hoặc đám đông mà không thể
tách biên từng vật: vẽ `ignore_region` đúng vùng đó. Đề xuất bổ sung lớp “phương
tiện khác” chỉ được đưa ra ở buổi thống nhất taxonomy, không được tự áp dụng khi
gán nhãn v1.0.

### Thuộc tính

| Thuộc tính | Giá trị | Quy tắc dùng trong nháp |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Đánh giá phần của **chính vật** bị vật khác che, không tính phần bị cắt bởi mép ảnh: `0` = 0–10% bị che; `1` = >10–30%; `2` = >30–60%; `3` = >60%. Nếu vẫn chắc chắn nhận diện được lớp thì vẫn gán box, kể cả mức `3`. |
| `truncated` | `no`, `yes` | `yes` khi vật kéo dài ra ngoài khung ảnh; `no` khi chỉ bị vật khác che trong khung. |
| `can_review` | tick / không | Tick khi không xác định chắc lớp, ranh giới, hay cách áp dụng rule; không dùng thay cho quyết định sau calibration. |

Không có thuộc tính về chuyển động, tốc độ, hướng đi hay tình trạng đỗ xe. Vì vậy
không suy luận và không ghi các thuộc tính đó cho car trên xe tải hay car bị mờ.

## 3. Quy tắc vẽ box

### Nguyên tắc chung

- Mỗi box ứng với **một thực thể vật lý** thuộc một lớp trong taxonomy; nhiều vật
  tách được không gộp thành một box chỉ vì chúng đứng sát nhau.
- Box ôm sát phần biên của thực thể mà rule đã cho phép gán. Box của các thực thể
  khác lớp có thể chồng/nằm trong nhau, ví dụ người và xe máy, nếu cả hai đều hợp
  lệ theo guideline đã chốt.
- Không dựa vào màu sắc tương tự để gộp xe. Tách từng xe khi có đủ biên hình học
  để xác định nó là một thực thể riêng; nếu không, theo R-05.
- Với vật bị che: luôn gán nếu vẫn chắc chắn nhận diện được lớp; box ôm phần nhìn
  thấy, không tưởng tượng biên của phần đã bị che. Gán `occluded_level` theo bảng
  thuộc tính. Nếu không chắc lớp, không gán box dựa trên phỏng đoán và tick
  `can_review` cho box liên quan (nếu có).
- Với vật mờ do chuyển động, tối hoặc ở xa: gán nếu vẫn chắc chắn nhận diện được
  lớp và biên box; không gán nếu không thể nhận diện chắc chắn. Không suy luận
  vật có đang chạy/đỗ, vì taxonomy không có thuộc tính này.
- Ngưỡng kỹ thuật bất biến trong taxonomy: chiều cao box tối thiểu 20 px. Không
  sửa mục `rules` trong `submission/taxonomy.json`.

### Quy tắc cá nhân đã chốt trước khi gán độc lập

| Rule ID | Vấn đề | Quyết định đã chốt |
|---|---|---|
| R-01 | Vật chở trên phương tiện | Gán từng car được chở nếu nhìn thấy và nhận diện chắc chắn là `car`; box của car chỉ ôm phần nhìn thấy của chính car, không lấy thùng/giá đỡ của xe tải. Gán box `truck` riêng. |
| R-02 | Người và phương tiện | Với người lái hoặc ngồi trên xe máy, gán đồng thời một box `person` và một box `motorcycle` khi mỗi thực thể nhận diện được. Box chồng nhau là hợp lệ. |
| R-03 | Che khuất | Luôn gán nếu vẫn chắc chắn nhận diện được lớp; box ôm phần nhìn thấy; mức che được ghi theo `occluded_level` 0–3 đã định nghĩa. |
| R-04 | Mờ chuyển động/tối/xa | Gán khi vẫn chắc chắn nhận diện được lớp; không gán khi không thể nhận diện chắc chắn. Không có thuộc tính chuyển động. |
| R-05 | Đám đông/vật sát nhau | Chỉ dùng `ignore_region` khi không thể xác định biên của từng thực thể riêng; không đặt ngưỡng số lượng người/xe cố định. |
| R-06 | Vật không thật | Không gán tượng, mannequin, mô hình, hình trên banner/quảng cáo, hoặc hình người cưỡi ngựa là `person` hay phương tiện thật. |
| R-07 | Phản chiếu | Không gán vật chỉ xuất hiện trong gương, kính hay phản chiếu. |
| R-08 | Phương tiện ngoài lớp | Xe đạp, xích lô, xe ba bánh chở hàng và phương tiện ngoài taxonomy không được ép vào lớp hiện có. Không gán nếu tách được; dùng `ignore_region` nếu chúng tạo vùng dày không tách được biên. Đề nghị thêm lớp mới chỉ thuộc pha thống nhất taxonomy. |
| R-09 | Hàng hoá/phụ kiện | Ôm phụ kiện hoặc hàng hoá chỉ khi gắn liền với vật chủ và là một phần cấu trúc/khối hiển thị của nó. Nếu hàng cồng kềnh tràn xa khỏi thân xe, box chỉ ôm phương tiện; tick `can_review` để đề xuất thuộc tính riêng ở pha thống nhất taxonomy, không tự thêm thuộc tính trong v1.0. |
| R-10 | Pickup | Gán pickup là `car`, áp dụng nhất quán toàn lô. |

## 4. Ca khó đã quan sát

| Ảnh | Tình huống | Rule ID | Quyết định cá nhân cần tự điền | Lý do/ghi chú |
|---|---|---|---|---|
| img_002 | Xe tải dài chở nhiều car. | R-01 | Gán truck và từng car nhận diện được; không có thuộc tính chuyển động. | Box car không ôm giá đỡ/thùng xe tải. |
| img_003 | Người ngồi xe máy; người gánh hàng; xe máy dựng sát; mật độ cao. | R-02, R-05, R-09 | Gán person + motorcycle khi tách/nhận diện được; `ignore_region` khi không tách được. | Box chồng nhau hợp lệ. |
| img_004 | Hai xe chở hàng/ba bánh không khớp mô tả car, truck, van, motorcycle. | R-08 | Không ép vào lớp hiện có. | Không tự thêm class khi gán. |
| img_005 | Người gần camera che nhiều người/xe sau; chỉ thấy mặt hoặc tay lái. | R-03 | Gán nếu còn chắc chắn lớp; box ôm phần thấy và ghi occlusion. | Phân biệt occluded với truncated. |
| img_006 | Nhiều người đi xe đạp; taxonomy không có bicycle. | R-08 | Không gán bicycle; dùng ignore chỉ khi vùng không tách được. | Áp dụng cùng img_023. |
| img_008 | Xe máy treo mũ, sát nhau, màu lẫn nhau. | R-05, R-09 | Tách từng xe nếu xác định được biên; mũ gắn liền được ôm theo xe. | Không dùng màu làm ranh giới. |
| img_010 | Xe đạp chở hàng; người lái/ngồi sau xe máy có box chồng nhau. | R-02, R-08, R-09 | Không gán bicycle; gán person và motorcycle khi nhận diện được. | Box chồng nhau không là lỗi. |
| img_011 | Người xa, đám đông, mô hình hình người. | R-04, R-05, R-06 | Chỉ gán người thật nhận diện chắc; ignore vùng không tách được; không gán mô hình. | Không suy đoán lớp. |
| img_012 | Phương tiện không rõ trước pickup; pickup thuộc car hay truck. | R-08, R-10 | Pickup là car; phương tiện không rõ ngoài taxonomy không gán. | Áp dụng cùng R-08. |
| img_013 | Car xa/tối, khó nhận diện và tách. | R-04 | Gán khi nhận diện chắc chắn; không gán nếu không chắc. | Không suy luận lớp. |
| img_014 | Người/phương tiện mờ do chuyển động. | R-04 | Gán khi nhận diện chắc chắn; không gán nếu không chắc. | Không có thuộc tính chuyển động. |
| img_015 | Car mờ do chuyển động. | R-04 | Gán khi nhận diện chắc chắn; không gán nếu không chắc. | Áp dụng cùng img_014. |
| img_017 | Tượng hình người cưỡi ngựa. | R-06 | Không gán. | Cùng xử lý mannequin/mô hình. |
| img_018 | Người trên banner quảng cáo. | R-06 | Không gán. | Không phải người thật trong cảnh. |
| img_019 | Đám đông chen chúc; phương tiện custom. | R-05, R-08 | Ignore vùng không tách được; không ép phương tiện custom vào lớp có sẵn. | Không tự đặt lớp mới. |
| img_020 | Car xuất hiện qua gương chiếu hậu/phản chiếu. | R-07 | Không gán. | Rule áp dụng mọi phản chiếu. |
| img_021 | Đám đông chờ đèn đỏ quá dày. | R-05 | Dùng ignore_region nếu không tách được biên từng người/xe. | Không dựa vào số lượng. |
| img_022 | Xe máy có chỗ ngồi phụ gắn bên trái. | R-09 | Ôm ghế phụ nếu là bộ phận gắn liền của xe máy. | Không ôm vật rời. |
| img_023 | Xe xích lô. | R-08 | Không gán; dùng ignore_region chỉ khi thuộc vùng không tách được. | Không phải bicycle/motorcycle trong taxonomy hiện tại. |
| img_024 | Mô hình khung xe trên nóc car. | R-06, R-09 | Không gán mô hình/khung như xe riêng; car được gán theo biên car và phần gắn liền được R-09 cho phép. | Phân biệt vật thật chở theo với mô hình. |

## 5. QC và phiên bản

- Trước khi export nhãn cá nhân, chạy `python scripts/validate_labels.py <file-coco>`.
- Khi không chắc, tick `can_review` và ghi ảnh/vùng/lý do; không tham khảo nhãn của
  thành viên khác trong Pha 3.
- Khi hết Pha 3, giữ nguyên file COCO cá nhân. IAA và calibration sẽ được so sánh
  sau khi tất cả đã nộp nhãn độc lập.
- Nếu một rule làm thay đổi cách chọn vật, lớp, biên box hoặc thuộc tính, đó là thay
  đổi ngữ nghĩa và phải được ghi ở guideline v1.1 cùng phạm vi rà lại cả lô.

| Phiên bản | Thay đổi | Lý do / ca dẫn chứng |
|---|---|---|
| v0-individual | Chốt quy tắc cá nhân R-01 đến R-10. | img_002–img_024, các ca ghi ở bảng trên. |
