# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_area` | polygon | class | — | — | — | Vùng ego được phép đi theo luật; đầu vào chính của planner |
| `area_type` | — | attribute của `drivable_area` | `__undefined__`, `direct`, `alternative` | `__undefined__` | no | Cùng một loại vùng, chỉ khác quyền ưu tiên; default undefined buộc annotator chọn |
| `boundary` | — | attribute của `drivable_area` | `visible`, `estimated` | `visible` | no | Ghi UNKNOWN cho biên phải đoán (đêm, tuyết, bị che) mà vẫn giữ polygon |
| `lane_marking` | polyline | class | — | — | — | Geometry khác (đường, không phải vùng), QA rule khác (lệch tim vạch) |
| `color` | — | attribute của `lane_marking` | `__undefined__`, `white`, `yellow` | `__undefined__` | no | Vàng = ranh giới ngược chiều; downstream cần phân biệt |
| `style` | — | attribute của `lane_marking` | `__undefined__`, `solid`, `dashed`, `double` | `__undefined__` | no | Quyết định có được chuyển làn qua vạch hay không |
| `crosswalk` | polygon | class | — | — | — | Vùng ưu tiên người đi bộ; chồng lên drivable nên phải là label riêng |
| `boundary` | — | attribute của `crosswalk` | `visible`, `estimated` | `visible` | no | Như trên, cho vạch đêm/mờ |
| `frame_review` | tag | class (tag ảnh) | — | — | — | Mỗi ảnh đúng 1 tag; nơi ghi quyết định ESCALATE nhìn thấy được trong export |
| `decision` | — | attribute của `frame_review` | `ok`, `escalate` | `ok` | no | |
| `reason` | — | attribute của `frame_review` | text tự do | rỗng | no | Bắt buộc khi `escalate`: vùng nào + nghi ngờ gì |

## Class hay attribute

- **Class** khi geometry hoặc QA rule khác nhau: `drivable_area` (vùng), `lane_marking` (đường), `crosswalk` (vùng
  chồng lên drivable). Tách `crosswalk` thành class vì nó phải nằm đè lên polygon drivable, không thay thế nó.
- **Attribute** khi là thuộc tính của cùng một đối tượng: `direct`/`alternative` chung geometry và QA rule; màu và
  kiểu vạch là thuộc tính của cùng một vạch. Tách thành class sẽ nổ tổ hợp (2 màu × 3 kiểu = 6 class).
- **Default có thể gây bias:**
  - `boundary = visible`: annotator quên đổi sẽ tạo biên "chắc chắn" giả ở ảnh tối. QA lọc ảnh `low_visibility` mà
    không có `estimated` nào.
  - `decision = ok`: ca đáng escalate bị bỏ qua im lặng. QA kiểm các ảnh có vùng bị bỏ trống.
  - `area_type`, `color`, `style` để `__undefined__` nên quên chọn là thấy ngay trong export.

## CVAT

- **Phiên bản CVAT** (`python lab9.py cvat`): 2.76.0 tại http://localhost:8080
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): chưa có task calibration độc lập (xem ghi chú dưới). Task dùng khi viết v2: task 37 `drivable_area` (job 38, 39) trên CVAT của Đặng Trường Huy
- **Guide của task đã dán `02_guideline.md`?** Có trong blind pack gửi peer (`guideline.md` = v2 đã freeze); task 37 chưa xác nhận
- **Nhóm dùng Track hay Shape, vì sao:** Shape — task ảnh tĩnh, mỗi ảnh là một cảnh độc lập, không có đối tượng nào
  cần giữ ID qua nhiều frame.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

Chưa thực hiện do hết thời gian buổi lab. Theo hướng dẫn của Lab Coach, bài nhóm tập trung vào guideline v2,
blind test với peer và guideline v3; setup test và calibration độc lập ghi là việc còn thiếu.
