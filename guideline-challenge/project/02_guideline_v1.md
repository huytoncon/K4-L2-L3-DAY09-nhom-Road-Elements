# Annotation guideline — Drivable area (vùng ego được phép đi) bằng polygon

**Version:** v1

> Bản lưu v1 (bản nháp đầu, trước khi gán nhãn). Bản đang dùng là v2 trong `02_guideline.md`; khác biệt ghi ở
> `08_revision_log.md`. Schema CVAT của v1 chỉ có `drivable_area` và `frame_review`.

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

**Mục đích:** dữ liệu huấn luyện model segmentation cho module planning, để trả lời câu hỏi *"xe ego được phép chạy
vào đâu, ngay bây giờ, theo luật?"*. Polygon là **vùng chức năng theo luật giao thông**, không phải "tô hết mặt nhựa".

**Trong scope** (phải vẽ):

- Làn ego đang chạy (`direct`).
- **Mọi làn cùng chiều khác mà xe đi được** (`alternative`), gồm làn kề, làn xa hơn, làn rẽ, làn nhập/tách, nhánh ra
  cùng chiều. Không cần ego chuyển sang được ngay bằng một lần chuyển làn.
- Phần giao lộ, vạch dừng và vạch sang đường nằm trên đường đi thẳng của các làn trên.

**Ngoài scope** (không vẽ):

- Làn ngược chiều ngăn với ego bằng **vạch vàng** (đơn, đôi, liền hay đứt) hoặc dải phân cách.
- Lề/vai đường nằm ngoài vạch biên liền (shoulder), vùng gạch chéo (gore, chevron), đảo giao thông.
- Làn đỗ xe, làn xe đạp, vỉa hè, bãi cỏ, bãi đất.
- Đường cắt ngang ở giao lộ.
- Nắp capô và mọi thứ bên trong xe ego.

## 2. Annotation unit

- **Đơn vị:** một ảnh tĩnh.
- **Instance:** mỗi **vùng liền nhau** là một polygon.
  - Có đúng **1 polygon `direct`** cho mỗi ảnh, trừ khi ảnh bị escalate (mục 7).
  - Mỗi dải `alternative` **tách rời** là một polygon riêng. Ví dụ: làn trái và làn phải của ego là **hai** polygon.
  - Các làn `alternative` nằm sát nhau cùng một phía (ví dụ 2 làn bên trái ego) gộp chung **1** polygon.
- Polygon `direct` và `alternative` **chung cạnh** tại vạch làn, không chồng lên nhau, không hở khe.

## 3. Geometry rule

- **Công cụ:** Polygon. Không dùng mask/brush.
- **Loại biên:** biên **visible** (theo phần mặt đường nhìn thấy), với 2 ngoại lệ:
  - **Xe và người trên làn:** vẽ **xuyên qua**, không cắt vật thể ra khỏi polygon. Lý do: xe là vật thể tạm thời, còn
    model object detection lo phần xe. Vùng làn phía sau xe vẫn tính nếu còn nhìn thấy hai bên hoặc phía trên xe.
  - **Vật che gắn trên xe ego** (giá điện thoại, sticker, giọt mưa, phản chiếu trên kính): nội suy biên đi qua như thể
    vật che không có. Nếu vật che làm mất một đoạn biên thì đặt `boundary = estimated`.
- **Đặt biên:**
  - Giữa `direct` và `alternative`: **tim vạch sơn phân làn**.
  - Tại vạch biên liền ở mép đường, vạch vàng tim đường, vạch làn xe đạp: **mép trong** của vạch, tức phía làn được vẽ.
    Vạch không thuộc vùng drivable.
  - Tại curb/vỉa hè/guardrail/tường: **chân** curb/guardrail, là chỗ mặt đường gặp vật đó.
  - Tại dãy xe đỗ không có vạch: đường nối **mép bánh xe phía lòng đường** của các xe đỗ.
- **Mép dưới:** ngay sát mép capô. Không vẽ lên capô.
- **Mép xa:** dừng ở điểm xa nhất còn phân biệt được biên làn. Làn còn hẹp dưới ~8 px thì chốt polygon về 1–2 đỉnh.
  Không kéo tới chân trời.
- **Độ chính xác:**
  - Nửa dưới ảnh (y > 450): đỉnh lệch biên thật ≤ 5 px.
  - Nửa trên: đỉnh lệch biên thật ≤ 10 px.
  - Đặt đỉnh dày ở chỗ cong và chỗ biên đổi hướng.
  - Không để cạnh polygon tự cắt nhau.
- **Mẹo:** giảm opacity polygon xuống ~30% và zoom 200% khi chỉnh biên gần vạch sơn.

## 4. Taxonomy

Nếu bảng này khác `03_ontology_and_cvat_setup.md` thì phải sửa cho khớp. Hai nơi này phải khớp nhau.

| Tên | Loại | Kiểu CVAT | Giá trị | Default | Ghi chú |
|---|---|---|---|---|---|
| `drivable_area` | label | polygon | — | — | label duy nhất để vẽ |
| `area_type` | attribute của `drivable_area` | select | `direct`, `alternative` | `__undefined__` | bắt buộc chọn; export còn `__undefined__` = lỗi |
| `boundary` | attribute của `drivable_area` | select | `visible`, `estimated` | `visible` | `estimated` khi ≥ 1 đoạn biên phải đoán (tối, loá, bị che, tuyết) |
| `frame_review` | tag (ảnh) | tag | — | — | **mỗi ảnh đúng 1 tag** |
| `decision` | attribute của `frame_review` | select | `ok`, `escalate` | `ok` | |
| `reason` | attribute của `frame_review` | text | tự do | rỗng | bắt buộc khi `decision = escalate` |

- **Là class:** chỉ có `drivable_area`, vì `direct` và `alternative` dùng chung geometry rule và chung QA rule.
- **Là attribute:** `direct`/`alternative` là attribute của cùng một loại vùng.
- **`direct`:** làn chứa **điểm giữa mép dưới vùng đường** (x ≈ 640, ngay trên capô). Nếu một vạch làn đi qua trong
  khoảng ±64 px quanh điểm này (ego đang đè vạch hoặc đang chuyển làn), `direct` là làn mà **hướng đầu xe** đang đi
  vào. Không chắc thì escalate (mục 7).
- **`alternative`:** mọi làn **cùng chiều** xe đi được mà **không phải** làn `direct`. Vạch trắng liền, vạch trắng đôi
  hay vùng gạch chéo nằm giữa ego và làn đó **không** làm làn đó mất tư cách `alternative`. Bản thân vùng gạch chéo thì
  vẫn không vẽ.

## 5. Inclusion / exclusion

| Trường hợp | Quyết định |
|---|---|
| Vạch sang đường, vạch dừng, mũi tên, chữ sơn trên làn | **Gộp** vào làn chứa nó |
| Nắp cống, vá đường, vết nứt, ổ gà nhỏ, vũng nước | **Gộp** (vẫn là mặt đường) |
| Giao lộ: đoạn nối thẳng của làn ego và làn cùng chiều | **Gộp**, nối biên làn thẳng qua giao lộ tới vạch/curb bên kia |
| Giao lộ: đường cắt ngang, góc cua | **Không** vẽ |
| Làn rẽ riêng, làn nhập/tách cao tốc cùng chiều | `alternative` |
| Làn cùng chiều nằm sau vạch trắng liền/đôi, vùng gore hay chevron (ví dụ nhánh ra) | `alternative`, polygon riêng. Riêng vùng gore/chevron thì **không** vẽ |
| Làn ngược chiều bên kia vạch vàng (đơn, đôi, liền, đứt) hoặc dải phân cách | **Không** vẽ |
| Làn có xe đi ngược lại nhưng không có vạch vàng ngăn | Escalate |
| Shoulder ngoài vạch biên liền (trắng hoặc vàng) | **Không** vẽ |
| Làn đỗ xe, làn xe đạp (có biểu tượng xe đạp hoặc chữ BIKE) | **Không** vẽ |
| Làn bus ghi rõ "BUS ONLY" hoặc sơn đỏ | **Không** vẽ. Đọc không rõ chữ → escalate |
| Đống tuyết, cọc, rào chắn, cone chiếm chỗ trên làn | **Cắt ra** khỏi polygon (khác với xe: vật cản tĩnh làm làn hẹp lại thật) |
| Đường không có vạch, 1 chiều, xe đỗ hai bên | Toàn bộ lòng đường giữa hai dãy xe đỗ là `direct` |
| Đường không vạch, 2 chiều, không phân biệt được nửa bên nào | Escalate |

## 6. Visibility / occlusion

- **Xe/người đang chạy trên làn:** vẽ xuyên qua (mục 3).
- **Xe to che hết làn phía trước** (truck, bus): kéo polygon tới **mép trên phần đường còn thấy hai bên xe**, không
  tưởng tượng thêm phía sau.
- **Vật gắn trên xe ego** (giá điện thoại, sticker, mưa trên kính): nội suy qua, đặt `boundary = estimated` nếu vật che
  mất biên.
- **Bị cắt ở mép ảnh:** polygon chạy dọc mép ảnh. Không kéo ra ngoài khung.
- **Đêm, hầm, bóng tối:**
  - Chỉ vẽ tới chỗ còn **thấy mặt đường** (được đèn chiếu, hoặc thấy vạch/curb).
  - Biên làn tối nhưng suy ra được từ vạch phía gần → nội suy theo hướng vạch, đặt `boundary = estimated`.
  - Không thấy được biên nào của làn ego → escalate.
- **Loá, phản chiếu trên mặt đường ướt:** vẫn là mặt đường. Lấy biên theo vạch và curb, không theo vệt sáng.
- **Tuyết che vạch:** suy biên từ vệt bánh xe và dãy xe đỗ, đặt `boundary = estimated`. Đống tuyết thì cắt ra (mục 5).

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | Rule mục 1–6 đủ để quyết | Polygon `drivable_area` với `area_type` đã chọn, `boundary = visible` |
| **LABEL (UNKNOWN biên)** | Biết chắc là vùng drivable, nhưng một đoạn biên phải đoán | Polygon như trên với `boundary = estimated` |
| **IGNORE** | Vùng ngoài scope (mục 1, 5) | Không vẽ gì |
| **ESCALATE** | Không xác định được làn ego; không biết vùng là cùng chiều hay ngược chiều; đọc không rõ biển/sơn quyết định làn (bus, bike, rẽ bắt buộc); mặt đường không nhìn thấy | Tag `frame_review` với `decision = escalate`, `reason` ghi **vùng nào + nghi ngờ gì**, ví dụ `làn phải: không rõ làn bus hay làn thường`. Vùng đã chắc vẫn vẽ bình thường; vùng đang nghi ngờ **không vẽ** |

- Ảnh nào cũng phải có đúng 1 tag `frame_review`. Ảnh bình thường để `decision = ok`.
- Phân vân giữa hai cách hiểu mà guideline không nói → escalate, **không** tự đoán ý tác giả.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh. Mỗi ảnh quyết độc lập, không suy luận từ ảnh trước/sau.

## 9. Examples

Mô tả theo hướng nhìn từ ghế lái. "Trái/phải" là trái/phải của ảnh.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Cao tốc 4+ làn cùng chiều, bên phải có vùng gạch chéo (gore) rồi tới nhánh ra | `direct` = làn giữa chứa x≈640. **1** `alternative` gộp các làn cùng chiều bên trái (dừng ở chân tường bê tông trái). **1** `alternative` cho làn bên phải ego tới vạch liền của gore. **1** `alternative` riêng cho nhánh ra cùng chiều sau gore. Vùng gore: không vẽ | 4, 5 (gore) |
| BDD10 | Phố 2 chiều, tim đường vạch vàng đứt, bên phải ego là vạch trắng liền + làn xe đạp + xe đỗ | `direct` = làn giữa vạch vàng (mép phải vạch) và vạch trắng liền (mép trái vạch). Làn bên trái vạch vàng là ngược chiều: không vẽ. Làn xe đạp và xe đỗ: không vẽ | 3, 5 (ngược chiều, bike) |
| BDD02 | Đang dừng trước vạch sang đường ở giao lộ, phố 1 chiều nhiều làn | `direct` = làn ego, gồm vạch dừng, vạch sang đường và đoạn nối thẳng qua giao lộ. Làn cùng chiều trái/phải: `alternative`, mỗi bên 1 polygon. Đường cắt ngang: không vẽ | 5 (giao lộ, crosswalk) |
| BDD16 | Đường chui dưới cầu, bên trái có vùng chevron vàng, xe phía trước | `direct` = làn ego, vẽ xuyên qua xe đen phía trước, biên trái = mép vạch vàng giáp chevron. Chevron: không vẽ. Phần tối dưới cầu thấy vạch: `boundary = visible` | 3 (xe), 5 (chevron), 6 |
| BDD18 | Đêm, phố có vạch sang đường, hai bên xe đỗ, xa phía trước tối | `direct` vẽ tới chỗ còn thấy mặt đường (~ngay sau crosswalk thứ hai), `boundary = estimated` phần xa. Dãy xe đỗ: không vẽ | 6 (đêm), 3 (xe đỗ) |
| BDD04 | Phố dốc, vạch vàng đôi, bên phải không có vạch, xe máy và xe đỗ sát curb | `direct` = từ mép phải vạch vàng đôi tới đường nối mép bánh xe đỗ bên phải. Bên trái vạch vàng đôi (có biểu tượng xe đạp): không vẽ | 3 (xe đỗ không vạch), 5 |
| (negative) | Vỉa hè, bãi cỏ, shoulder cao tốc ngoài vạch biên liền | Không có polygon ở đó | 1, 5 |

## 10. Common mistakes

1. **Tô hết mặt nhựa:** lấn qua vạch vàng sang làn ngược chiều, sang shoulder hoặc làn đỗ xe, chỉ vì cùng màu. Luôn
   hỏi: *đây có phải làn cùng chiều mà xe đi được không?*
2. **Bỏ sót làn cùng chiều ở xa** (sau vạch trắng liền, sau gore, nhánh ra): làn nào cùng chiều và đi được thì đều
   là `alternative`.
3. **Quên chọn `area_type`:** polygon còn `__undefined__`. Trước khi export, lọc theo attribute để kiểm.
4. **Gộp `alternative` hai bên ego thành một polygon** hoặc vẽ đè lên `direct`. Hai bên là hai polygon, chung cạnh
   với `direct`.
5. **Cắt xe ra khỏi polygon:** guideline này vẽ **xuyên qua** xe đang chạy. Chỉ cắt vật cản tĩnh (tuyết, cone, rào).
6. **Kéo polygon vào vùng tối** mà không thấy mặt đường, hoặc kéo tới chân trời. Dừng ở chỗ còn phân biệt được biên.
7. **Vẽ lên capô** hoặc bỏ hở một dải trên capô. Mép dưới polygon phải sát mép capô.
8. **Tính vạch biên liền vào vùng drivable:** biên nằm ở **mép trong** vạch biên và vạch tim. Chỉ vạch phân làn cùng
   chiều mới lấy ở tim vạch.
9. **Quên tag `frame_review`**, hoặc chọn `escalate` mà để trống `reason`.
10. **Làn ego sai khi xe đang chuyển làn:** dùng điểm x≈640 ở mép dưới, và rule ±64 px ở mục 4.
