# Problem statement + downstream contract

## Bài toán

Drivable area theo **luật** (không theo màu mặt đường) cho ảnh dashcam BDD100K, kèm vạch làn và vạch sang đường làm
ranh giới — khó ở làn ngược chiều chỉ ngăn bằng vạch vàng, làn cùng chiều nằm sau gore/vạch liền, xe và vật cản
chiếm làn, vạch sang đường, và cảnh đêm/mưa/tuyết.

## Downstream contract

1. **Downstream là ai?** Model segmentation vùng đi được cho module planning (chọn làn, quyết định chuyển làn) của
   xe tự hành; lane_marking và crosswalk là đầu vào phụ cho lane keeping và giảm tốc trước vạch sang đường.
2. **Output cần?** Polygon `drivable_area` với `area_type` (`direct` / `alternative`) và `boundary`
   (`visible` / `estimated`); polyline `lane_marking` với `color`, `style`; polygon `crosswalk`; tag ảnh
   `frame_review` (`ok` / `escalate` + `reason`).
3. **Failure nguy hiểm nhất?** Gán làn **ngược chiều** (bên kia vạch vàng) là drivable → planner cho xe lấn làn
   ngược chiều. Kế đến: vẽ vùng drivable phủ lên xe/người, và bỏ sót vạch sang đường.
4. **Escalation path?** Annotator gắn tag `frame_review = escalate`, ghi `reason` (vùng nào + nghi ngờ gì), không vẽ
   vùng nghi ngờ; spec owner của nhóm xử lý và, nếu thành rule mới, ghi vào `08_revision_log.md`.

## Scope

- **Trong scope:** làn ego (`direct`), mọi làn cùng chiều đi được (`alternative`), vạch sơn dọc, vạch sang đường.
- **Ngoài scope:** làn ngược chiều, shoulder, gore/chevron, làn đỗ xe, làn xe đạp, vỉa hè, đường cắt ngang ở giao lộ,
  vạch ngang (vạch dừng, mũi tên, chữ sơn), capô xe ego.
- **Geometry tolerance:** đỉnh polygon/polyline lệch biên thật ≤ 5 px ở nửa dưới ảnh (y > 450), ≤ 10 px ở nửa trên.

## Output chấm được

- **LABEL:** có polygon/polyline đúng label và attribute.
- **IGNORE:** không có shape ở vùng ngoài scope (ví dụ làn ngược chiều).
- **UNKNOWN:** `boundary = estimated`.
- **ESCALATE:** tag `frame_review` `decision = escalate` + `reason`.
- **Geometry:** biên `direct` so với gold theo tolerance trên.

## Dữ liệu và giới hạn

26 ảnh `bdd100k` (1280×720, chủ yếu ban ngày; 2 đêm, 2 chạng vạng, 2 tuyết, 1 mưa). Dự kiến dùng 3 example,
6 calibration, 5 blind. Giới hạn: ít ảnh đêm/thời tiết xấu; không có cao tốc nhiều nhánh phức tạp; 5 ảnh BDD03, 10,
16, 17, 18 trùng mini lab drivable nên không dùng làm blind.
