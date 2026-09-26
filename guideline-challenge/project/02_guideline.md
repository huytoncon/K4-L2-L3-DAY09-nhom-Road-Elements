# Annotation guideline — Road surface: drivable area, vạch làn, vạch sang đường

**Version:** v3

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

**Trước khi vẽ:** tạo task bằng **đúng** `03_cvat_labels.json` (dán vào Raw). Mở tab Constructor, kiểm tra đủ 4 label
`drivable_area`, `lane_marking`, `crosswalk`, `frame_review`. Thiếu label nào thì dừng, sửa task rồi mới vẽ.

**Điểm mới ở v3:**

1. Cách xác định `direct` khi phía trước có nhiều xe, hoặc khi làn ego bị vật cản chắn (mục 4).
2. Ngưỡng "vật cản chắn hết bề ngang làn" (mục 5).
3. Bước tự kiểm bắt buộc cho các attribute có default (`boundary`, `decision`) trước khi export (mục 10).

**Điểm mới ở v2 so với v1:**

1. Cắt xe/người ra khỏi polygon, thay cho quy tắc "vẽ xuyên qua" (mục 3, 6).
2. Thêm label `lane_marking` (polyline) cho vạch làn (mục 3b, 4).
3. Thêm label `crosswalk` (polygon) cho vạch sang đường (mục 3c, 4).
4. Thêm rule cho vật cản chia đôi làn, vật gắn trên xe ego, và các ca escalate đã gặp (mục 5–7).

## 1. Objective + scope

**Mục đích:** dữ liệu huấn luyện cho module planning, trả lời ba câu hỏi:

- *Xe ego được phép chạy vào đâu, ngay bây giờ, theo luật?* → `drivable_area`
- *Ranh giới làn nằm ở đâu?* → `lane_marking`
- *Chỗ nào người đi bộ được ưu tiên sang đường?* → `crosswalk`

Polygon drivable là **vùng chức năng theo luật giao thông**, không phải "tô hết mặt nhựa".

**Trong scope** (phải vẽ):

- `drivable_area`:
  - làn ego đang chạy (`direct`);
  - **mọi làn cùng chiều khác mà xe đi được** (`alternative`), gồm làn kề, làn xa hơn, làn rẽ, làn nhập/tách, nhánh ra
    cùng chiều;
  - phần giao lộ, vạch dừng, vạch sang đường nằm trên đường đi thẳng của các làn trên.
- `lane_marking`: mọi vạch sơn **dọc** (chạy theo hướng xe đi), gồm vạch phân làn, vạch tim đường, vạch biên — kể cả
  vạch thuộc phía ngược chiều.
- `crosswalk`: mọi vạch sang đường nhìn thấy trong ảnh, kể cả vạch nằm ngoài vùng drivable.

**Ngoài scope** (không vẽ bằng label nào):

- Làn ngược chiều ngăn với ego bằng **vạch vàng** (đơn, đôi, liền hay đứt) hoặc dải phân cách — không có
  `drivable_area`, nhưng vạch sơn của nó vẫn là `lane_marking`.
- Shoulder ngoài vạch biên liền, vùng gạch chéo (gore, chevron), đảo giao thông, làn đỗ xe, làn xe đạp, vỉa hè, bãi cỏ.
- Đường cắt ngang ở giao lộ (trừ `crosswalk` trên đó).
- Vạch **ngang**: vạch dừng, sọc gạch chéo trong gore/chevron, mũi tên, chữ sơn. Chúng không phải `lane_marking`.
- Nắp capô và mọi thứ bên trong xe ego.

## 2. Annotation unit

- **Đơn vị:** một ảnh tĩnh. Mọi shape là Shape, không dùng Track.
- **`drivable_area`:** mỗi **vùng liền nhau** là một polygon.
  - Có đúng **1 polygon `direct`** mỗi ảnh, trừ ảnh escalate (mục 7).
  - Mỗi dải `alternative` **tách rời** là một polygon riêng. Làn trái và làn phải ego là **hai** polygon. Các làn
    `alternative` sát nhau cùng một phía gộp **1** polygon.
  - `direct` và `alternative` **chung cạnh** tại vạch làn, không chồng lên nhau, không hở khe.
- **`lane_marking`:** mỗi vạch sơn liên tục theo chiều dọc là **1 polyline**.
  - Vạch đứt vẫn là 1 polyline liền, nối qua các khoảng trống.
  - Vạch đôi (hai vạch song song sát nhau) là **1** polyline ở giữa hai vạch, `style = double`.
  - Vạch bị cắt ngang bởi giao lộ hoặc vạch sang đường: kết thúc polyline ở đó, phía bên kia là polyline mới.
- **`crosswalk`:** mỗi vạch sang đường là **1 polygon**. Nếu xe đứng trên vạch cắt nó thành nhiều đoạn thì mỗi đoạn
  là một polygon riêng.

## 3. Geometry rule

### 3a. `drivable_area` (polygon)

- **Biên visible:** chỉ gồm phần mặt đường nhìn thấy.
- **Xe, người, vật cản trên làn → cắt ra:**
  - Biên đi theo **mép dưới** vật (chỗ bánh xe/chân chạm đường) và **hai cạnh bên** của vật.
  - Phần đường phía sau vật, trong bề ngang của vật, bị che nên **không vẽ**.
  - Polygon **không có lỗ**: vùng bị cắt luôn kéo từ mép dưới vật lên tới mép xa của polygon.
- **Không cắt vật gắn trên xe ego** (giá điện thoại, sticker, giọt mưa trên kính) và **phản chiếu trên capô**. Nội suy
  biên đi qua như thể không có. Nếu mất một đoạn biên thì đặt `boundary = estimated`.
- **Đặt biên:**
  - Giữa `direct` và `alternative`: **tim vạch phân làn**, trùng với polyline `lane_marking`.
  - Tại vạch biên liền, vạch vàng tim đường, vạch làn xe đạp: **mép trong** vạch. Vạch không thuộc drivable.
  - Tại curb, guardrail, tường: **chân** vật đó.
  - Tại dãy xe đỗ không có vạch: đường nối **mép bánh xe phía lòng đường**.
- **Mép dưới** sát mép capô. **Mép xa** dừng ở chỗ còn phân biệt được biên làn. Không kéo tới chân trời.
- **Độ chính xác:**
  - Nửa dưới ảnh (y > 450): đỉnh lệch ≤ 5 px.
  - Nửa trên: đỉnh lệch ≤ 10 px.
  - Không để cạnh polygon tự cắt nhau.

### 3b. `lane_marking` (polyline)

- Chạy dọc **tim vạch**, điểm đầu sát capô hoặc mép ảnh, điểm cuối là chỗ xa nhất còn thấy vạch.
- Đặt đỉnh cách nhau ~40–80 px, dày hơn ở chỗ vạch cong. Đỉnh lệch tim vạch ≤ 5 px ở nửa dưới, ≤ 10 px ở nửa trên.
- **Vạch bị xe che một đoạn ngắn**, còn thấy hai phía: nối thẳng qua. Bị che tới hết tầm nhìn: dừng ở mép xe.
- Được vẽ đè lên polygon `drivable_area`. Không cắt, không sửa polygon vì vạch.

### 3c. `crosswalk` (polygon)

- Bao **mép ngoài của cả dải sọc**: cạnh gần là đầu sọc phía xe, cạnh xa là đầu sọc phía bên kia, hai cạnh ngắn là
  mép sọc ngoài cùng. Khoảng trống giữa các sọc nằm trong polygon.
- Vẽ đè lên `drivable_area`. Drivable **không** bị cắt ở vạch sang đường.
- **Xe/người đứng trên vạch → cắt ra** theo cùng cách với 3a.
- Vạch chạy ra ngoài khung ảnh: vẽ tới mép ảnh.
- **Vạch dừng** (vạch ngang liền trước vạch sang đường) không thuộc `crosswalk`.

## 4. Taxonomy

Bảng này phải khớp với `03_ontology_and_cvat_setup.md` và `03_cvat_labels.json`.

| Tên | Loại | Kiểu CVAT | Giá trị | Default | Ghi chú |
|---|---|---|---|---|---|
| `drivable_area` | label | polygon | — | — | |
| `area_type` | attr. của `drivable_area` | select | `direct`, `alternative` | `__undefined__` | bắt buộc chọn |
| `boundary` | attr. của `drivable_area` | select | `visible`, `estimated` | `visible` | `estimated` khi phải đoán ≥ 1 đoạn biên |
| `lane_marking` | label | polyline | — | — | |
| `color` | attr. của `lane_marking` | select | `white`, `yellow` | `__undefined__` | bắt buộc chọn |
| `style` | attr. của `lane_marking` | select | `solid`, `dashed`, `double` | `__undefined__` | bắt buộc chọn |
| `crosswalk` | label | polygon | — | — | |
| `boundary` | attr. của `crosswalk` | select | `visible`, `estimated` | `visible` | |
| `frame_review` | tag (ảnh) | tag | — | — | **mỗi ảnh đúng 1 tag** |
| `decision` | attr. của `frame_review` | select | `ok`, `escalate` | `ok` | |
| `reason` | attr. của `frame_review` | text | tự do | rỗng | bắt buộc khi `escalate` |

- **Là class:** `drivable_area`, `lane_marking`, `crosswalk`. Ba thứ này khác geometry (polygon / polyline / polygon)
  và khác QA rule.
- **Là attribute:** `direct`/`alternative`, màu và kiểu vạch. Đây là thuộc tính của cùng một loại đối tượng.
- **`direct`:** làn chứa **điểm giữa mép dưới vùng đường** (x ≈ 640). Nếu vạch làn nằm trong ±64 px quanh điểm này,
  `direct` là làn mà hướng đầu xe đang đi vào. Không chắc thì escalate.
  - Xác định làn ego **ở sát capô rồi dò theo vạch làn lên xa**, không theo chiếc xe gần giữa ảnh nhất. Xe phía trước
    có thể nằm ở làn bên cạnh.
  - Làn ego **không đổi** khi phía trước bị vật cản chắn (đống tuyết, xe dừng). Phần làn ego trước vật cản vẫn là
    `direct`; làn bên cạnh còn thông vẫn là `alternative`, dù xe đang đi ở làn đó nhìn "thoáng" hơn.
- **`alternative`:** mọi làn **cùng chiều** xe đi được, không phải `direct`. Vạch trắng liền, vạch đôi hay gore nằm
  giữa **không** làm làn đó mất tư cách `alternative`.
- **`style`:** vạch đứt ở gần nhưng liền ở xa (hay ngược lại) thì lấy **kiểu ở đoạn gần xe**.

## 5. Inclusion / exclusion

| Trường hợp | Quyết định |
|---|---|
| Vạch sang đường, vạch dừng, mũi tên, chữ sơn trên làn | Gộp vào `drivable_area` của làn chứa nó. Vạch sang đường **thêm** 1 `crosswalk` |
| Nắp cống, vá đường, vết nứt, vũng nước | Gộp vào `drivable_area` |
| Giao lộ: đoạn nối thẳng của làn ego và làn cùng chiều | Gộp, nối biên làn thẳng qua giao lộ |
| Giao lộ: đường cắt ngang, góc cua | Không vẽ drivable |
| Làn rẽ riêng, làn nhập/tách, nhánh ra cùng chiều (kể cả sau gore) | `alternative`, polygon riêng. Vùng gore thì không vẽ |
| Làn ngược chiều bên kia vạch vàng hoặc dải phân cách | Không vẽ drivable. Vạch vàng vẫn là `lane_marking` |
| Làn có xe đi ngược lại nhưng không có vạch vàng ngăn | Escalate |
| Shoulder, làn đỗ xe, làn xe đạp | Không vẽ drivable. Vạch ngăn chúng vẫn là `lane_marking` |
| Làn bus ghi rõ "BUS ONLY" hoặc sơn đỏ | Không vẽ. Đọc không rõ chữ → escalate |
| Ô tô, xe tải, bus, xe máy, xe đạp, người trên làn (chạy hay dừng) | Cắt ra khỏi `drivable_area` và `crosswalk` (mục 3a) |
| Đống tuyết, cọc, rào chắn, cone trên làn | Cắt ra, giống xe |
| **Vật cản chắn hết bề ngang làn ego** (chiếm ≥ 80% bề ngang làn tại mép dưới vật cản; ví dụ đống tuyết giữa làn) | `direct` dừng ở mép dưới vật cản. Phần làn phía sau vật cản **không** vẽ, không tạo polygon `direct` thứ hai |
| Đường không vạch, 1 chiều, xe đỗ hai bên | Toàn bộ lòng đường giữa hai dãy xe đỗ là `direct` |
| Đường không vạch, 2 chiều, không phân biệt được hai nửa | Escalate |
| Vệt nghi là vạch sang đường nhưng quá tối/mờ để chắc | Không vẽ `crosswalk`; nếu ảnh hưởng quyết định drivable thì escalate |

## 6. Visibility / occlusion

- **Xe/người trên làn:** cắt ra, kể cả phần đường phía sau bị che (mục 3a).
- **Xe to che hết bề ngang làn** (truck, bus): polygon làn đó dừng ở mép dưới xe.
- **Xe đỗ lấn một phần vào làn:** cắt theo thân xe. Không coi cả làn là làn đỗ xe.
- **Vật gắn trên xe ego, phản chiếu trên capô:** không cắt, nội suy qua (mục 3a).
- **Bị cắt ở mép ảnh:** shape chạy dọc mép ảnh, không kéo ra ngoài khung.
- **Đêm, hầm, bóng tối:**
  - Chỉ vẽ tới chỗ còn thấy mặt đường hoặc vạch.
  - Biên hoặc vạch suy ra được từ đoạn gần → nội suy theo hướng đó, đặt `boundary = estimated` (drivable,
    crosswalk). Polyline `lane_marking` chỉ vẽ đoạn còn thấy vạch.
  - Không thấy được biên nào của làn ego → escalate.
- **Mặt đường ướt, loá:** vệt đèn phản chiếu **không** phải vạch sơn. Lấy vạch theo sơn thật, biên theo vạch/curb.
- **Tuyết che vạch:** suy biên từ vệt bánh xe và dãy xe đỗ, đặt `boundary = estimated`. Không vẽ `lane_marking` cho
  đoạn vạch bị tuyết phủ.

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | Rule mục 1–6 đủ để quyết | Shape với attribute đã chọn, `boundary = visible` |
| **LABEL (UNKNOWN biên)** | Chắc là vùng đó, nhưng một đoạn biên phải đoán | Shape như trên với `boundary = estimated` |
| **IGNORE** | Ngoài scope (mục 1, 5) | Không vẽ gì |
| **ESCALATE** | Không xác định được làn ego; không biết cùng hay ngược chiều; chữ/sơn quyết định làn không đọc được; vật gắn trên xe ego che mất đoạn cần để quyết | Tag `frame_review`: `decision = escalate`, `reason` ghi **vùng nào + nghi ngờ gì**. Vùng đã chắc vẫn vẽ; vùng nghi ngờ **không** vẽ |

- Ảnh nào cũng phải có đúng 1 tag `frame_review`. Ảnh bình thường để `decision = ok`.
- Ví dụ `reason` tốt: `làn trái: bị giá điện thoại che, có chữ sơn không đọc được (có thể BUS)`.
- Phân vân giữa hai cách hiểu mà guideline không nói → escalate, không tự đoán ý tác giả.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh. Mỗi ảnh quyết độc lập, không suy luận từ ảnh trước/sau.

## 9. Examples

Mô tả theo hướng nhìn từ ghế lái. "Trái/phải" là trái/phải của ảnh.

| sample_id | Thấy gì | Expected output | Rule |
|---|---|---|---|
| BDD01 | Cao tốc 4+ làn, bên phải có gore rồi nhánh ra | `direct` = làn giữa. 1 `alternative` gộp các làn bên trái; 1 cho làn phải tới vạch gore; 1 riêng cho nhánh ra. Gore: không vẽ. SUV phía trước cắt ra. `lane_marking`: 3 vạch đứt trắng + vạch liền trắng mép gore | 3a, 3b, 5 |
| BDD02 | Dừng trước vạch sang đường, phố 1 chiều | `direct` gồm vạch dừng, vạch sang đường và đoạn nối qua giao lộ. Taxi bên trái cắt khỏi `alternative`. 1 `crosswalk` bắt đầu từ mép phải taxi | 3a, 3c, 5 |
| BDD10 | Phố 2 chiều, tim đường vàng đứt, bên phải vạch trắng liền + làn xe đạp | `direct` giữa mép phải vạch vàng và mép trái vạch trắng. `lane_marking`: vàng `dashed` ở tim, trắng `solid` hai bên. Làn ngược chiều, làn xe đạp: không có drivable | 3b, 5 |
| BDD16 | Đường chui, chevron vàng bên trái, xe đen phía trước | `direct` cắt xe đen ra, phía sau xe không vẽ; biên trái = mép vạch vàng giáp chevron. `lane_marking`: vàng `solid` cong + trắng `solid` bên phải | 3a, 3b, 5 |
| BDD18 | Đêm, vạch sang đường, xe đỗ hai bên | `direct` tới chỗ còn thấy mặt đường, `estimated`. 1 `crosswalk` `estimated` | 3c, 6 |
| BDD04 | Phố dốc, vạch vàng đôi, bên phải không vạch, xe đỗ sát curb | `direct` từ mép phải vạch vàng đôi tới đường nối mép bánh xe đỗ. `lane_marking`: vàng `double` | 3a, 3b |
| (negative) | Vỉa hè, bãi cỏ, shoulder ngoài vạch biên liền | Không có shape nào | 1, 5 |

## 10. Common mistakes

1. **Tô hết mặt nhựa:** lấn qua vạch vàng, sang shoulder hoặc làn đỗ xe. Hỏi: *đây có phải làn cùng chiều mà xe đi
   được không?*
2. **Bỏ sót làn cùng chiều ở xa** (sau vạch trắng liền, sau gore, nhánh ra).
3. **Còn `__undefined__`** ở `area_type`, `color` hoặc `style`. Lọc theo attribute trước khi export.
4. **Gộp `alternative` hai bên ego** thành một polygon, hoặc vẽ đè lên `direct`.
5. **Vẽ đè lên xe/người**, hoặc vẽ tiếp phần đường phía sau xe. Ngược lại: **cắt nhầm** phản chiếu trên capô hay giá
   điện thoại.
6. **Tạo polygon `direct` thứ hai** cho phần làn phía sau vật cản chắn ngang.
7. **Kéo shape vào vùng tối**, hoặc tới chân trời.
8. **Tính vạch biên vào drivable:** biên ở **mép trong** vạch biên và vạch tim; chỉ vạch phân làn cùng chiều lấy tim.
9. **Polyline vạch đứt bị vẽ thành nhiều đoạn**, hoặc vạch đôi vẽ thành 2 polyline.
10. **Nhầm vệt đèn phản chiếu trên đường ướt là vạch sơn.**
11. **Cắt drivable ở vạch sang đường**, hoặc gộp vạch dừng vào `crosswalk`.
12. **Quên tag `frame_review`**, hoặc `escalate` mà để trống `reason`.

**Tự kiểm bắt buộc trước khi export** (các attribute có default dễ bị giữ nguyên mà không xét):

- [ ] Mỗi polygon ở ảnh tối/tuyết/mưa hoặc bị che: đã **chủ động** chọn `boundary` (`visible` hay `estimated`).
- [ ] Mỗi ảnh có vùng bỏ trống vì nghi ngờ: tag `frame_review` là `escalate` và có `reason`.
- [ ] Không còn `__undefined__` ở `area_type`, `color`, `style`.
- [ ] Không polygon nào phủ lên xe/người (trừ phản chiếu trên capô, vật gắn trên xe ego).
- [ ] `direct` và `alternative` chung cạnh tại tim vạch phân làn, không để khe hở; mép dưới dừng sát capô.
- [ ] Mỗi vạch sang đường có 1 `crosswalk`; mỗi vạch dọc nhìn thấy có 1 `lane_marking`.
