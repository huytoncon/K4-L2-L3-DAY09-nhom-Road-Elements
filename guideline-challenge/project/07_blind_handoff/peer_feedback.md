# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền.

- **Nhóm peer:** nhóm leductu, nguyenhoangtung, phamanhhuy, vutungdinh (chủ đề Functional Drivable Area)
- **Người label blind:** thành viên nhóm peer, label trên máy CVAT của owner (tài khoản `dangtruonghuy`); file
  `peer_output/peer_export_roadelements.zip`, task tạo 07:21, export 07:37 UTC

## 0. Kết quả blind test

- **GTS = 75.0**: D 62.5 (10/16), C 100 (2/2, 0 critical escape), G 75 (3/4), I 100 (0 câu hỏi). Chi tiết:
  `gts_summary.md`, `transfer_score.csv`.
- **Lỗi quy trình:** peer tạo task bằng schema của nhóm họ (`areaType`, `visibility`, `needs_review`), không dán
  `cvat_labels.json` trong gói. Vì vậy 3/6 decision sai (crosswalk BDD13, BDD07; lane_marking BDD23) là do **không
  có label để vẽ**, không phải do đọc sai guideline.
- File nhận trước đó (`peer (1).zip`, bộ ảnh của chính nhóm peer; `nhom (1).zip` bản đầu 0 shape) không dùng để chấm.

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất? Peer: guideline "khá rõ ràng". Bài làm cho thấy rule không vẽ làn
   ngược chiều bên kia vạch vàng và rule cắt xe được làm đúng ở mọi ảnh (2/2 critical đúng).
2. Rule nào mơ hồ hoặc phải tự suy diễn? Peer không nêu thêm.
3. Sample nào khiến guideline "vỡ"? Peer không nêu thêm.
4. Attribute / default nào trong CVAT dễ gây thao tác sai? Peer không nêu thêm.
5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn? Peer không nêu thêm.

Clarification log: peer không hỏi câu nào trong lúc làm.

## 2. Owner phân loại

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| BDD24 d1, d4: đảo `direct`/`alternative` — làn ego bị đống tuyết chắn gán alternative, làn trái (limo) gán direct | guideline gap: chưa nói làn ego vẫn là làn chứa x≈640 khi phía trước bị chắn | accept + revise: v3 mục 4 thêm rule "làn ego không đổi dù bị chắn" | transfer_score BDD24 d1, d4 |
| BDD24 d3: tuyết che biên nhưng để `visible` | guideline gap: default dễ bị giữ nguyên | accept + revise: v3 thêm checklist tự kiểm attribute có default | transfer_score BDD24 d3 |
| BDD13 d2, BDD07 d4, BDD23 d4: thiếu crosswalk / lane_marking | execution error (quy trình): dùng schema của nhóm khác | accept + revise: v3 đầu file thêm bước kiểm đủ 4 label trong Constructor trước khi vẽ | export chỉ có label `drivable_area` |
| BDD14 d3: biên direct cách tim vạch 20–40 px, mép dưới lấn capô | execution error: rule đã có (mục 2: chung cạnh, không hở khe; mục 3a: sát capô) | coaching + v3 nêu rule này thành một dòng trong checklist | overlay BDD14 |
| Peer: "khá rõ ràng", không hỏi câu nào | — | no_change; I = 100 | clarification_log.csv trống |
