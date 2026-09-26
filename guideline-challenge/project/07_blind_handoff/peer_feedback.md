# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền.

- **Nhóm peer:** nhóm leductu, nguyenhoangtung, phamanhhuy, vutungdinh (chủ đề Functional Drivable Area)
- **Người label blind:** chưa có. Nhóm peer chưa gửi file export hợp lệ trên blind pack của nhóm mình (xem mục 0)

## 0. Tình trạng blind test (tới lúc nộp)

| Việc | Tình trạng | Bằng chứng |
|---|---|---|
| Gửi blind pack | ✓ `handoff/blind-pack.zip` (guideline v2 đã freeze, 4 label, 5 ảnh BDD07/13/14/23/24) | tag `gold-freeze`, commit freeze |
| Nhận export của peer | ✗ Hai file nhận được không phải bài của peer trên gói nhóm mình | xem dưới |
| Chấm `transfer_score.csv` / GTS | ✗ Không chạy, vì chấm file sai sẽ cho kết luận sai về guideline | — |

File đã nhận và lý do không dùng để chấm:

- `peer (1).zip`: ảnh BDD08/16/23/24/25 (bộ blind **của nhóm peer**), schema của nhóm peer (`areaType`, `visibility`,
  `needs_review`), không có thời điểm export CVAT (`dumped` trống).
- `nhom (1).zip`: đúng 5 ảnh blind của nhóm mình, nhưng owner là tài khoản CVAT của owner nhóm mình, **0 shape**,
  và chỉ có label `drivable_area` (thiếu `lane_marking`, `crosswalk`, `frame_review`).

Clarification log: không có câu hỏi nào được ghi nhận.

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất? Peer nhận xét bằng lời: guideline "khá rõ ràng". Không nêu rule cụ thể.
2. Rule nào mơ hồ hoặc phải tự suy diễn? Chưa có câu trả lời (peer chưa làm bài trên gói).
3. Sample nào khiến guideline "vỡ"? Chưa có câu trả lời.
4. Attribute / default nào trong CVAT dễ gây thao tác sai? Chưa có câu trả lời.
5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn? Chưa có câu trả lời.

## 2. Owner phân loại

Vì chưa có bài peer, v3 chỉ dựa trên bằng chứng nhóm mình **thực sự thu được**: lần nhóm mình làm blind pack của
peer (`peer.zip`, 5 ảnh, 8 polygon), việc đọc guideline của peer, và lỗi phát hiện khi viết gold.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Peer: guideline "khá rõ ràng" | — | no_change (không đủ chi tiết để sửa rule cụ thể) | tin nhắn của peer |
| Khi làm gói của peer, cả 8/8 polygon giữ nguyên default `visibility`/`needs_review` | guideline gap: default không bắt người label chủ động chọn | accept + revise: v3 thêm bước tự kiểm bắt buộc cho `boundary` và `decision` | `peer.zip` (BDD08, 16, 23, 24, 25) |
| Khi làm gói của peer, người label mang thói quen cắt xe từ guideline nhóm mình | execution error do hai guideline khác nhau | accept + revise: v3 nhấn mạnh rule cắt xe ngay đầu mục 3 kèm lý do | `peer.zip` BDD16, BDD25 |
| Guideline peer không có ngưỡng cho "che một đoạn ngắn" | guideline gap (của peer, nhưng v2 nhóm mình cũng chưa có ngưỡng "chắn hết bề ngang") | accept + revise: v3 thêm ngưỡng ≥ 80% bề ngang làn | guideline peer mục 3, 6; gold BDD24 d1 |
| Bản nháp label BDD12 chọn nhầm làn `direct` (làn SUV bên trái thay vì làn xe van chứa x≈640) | guideline gap: chưa có ví dụ ego nằm sau xe ở làn khác | accept + revise: v3 thêm cách xác định `direct` khi phía trước có nhiều xe | BDD12 (bản nháp job 38) |
| Blind set ban đầu có BDD12, BDD26 mà peer đã dùng làm calibration | data/process: trùng ảnh giữa hai nhóm | đổi sang BDD07, BDD23 trước freeze | guideline peer mục 9; commit `07579e8` |
