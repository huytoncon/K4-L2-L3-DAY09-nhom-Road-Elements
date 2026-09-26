# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: 1 label `drivable_area` (`direct`/`alternative`, `boundary`) + tag `frame_review`; vẽ xuyên qua xe | Sau khi xem 26 ảnh BDD | `02_guideline_v1.md` |
| v1 | `alternative` = mọi làn cùng chiều đi được (bỏ điều kiện "tới được bằng một lần chuyển làn"); làn bên kia vạch vàng luôn không vẽ | Nhóm chốt định nghĩa trước khi gán nhãn | BDD01: nhánh ra sau gore |
| v2 | Cắt xe/người ra khỏi polygon và bỏ phần đường phía sau xe; không cắt vật gắn trên xe ego và phản chiếu trên capô | Vẽ xuyên qua xe làm vùng drivable phủ lên vật cản thật; khi gán nhãn, phản chiếu trên capô dễ bị cắt nhầm | Job 38: BDD01, BDD07, BDD09, BDD13, BDD16, BDD25; phản chiếu: BDD02, BDD13 |
| v2 | Thêm label `lane_marking` (polyline, `color`, `style`) và rule: tim vạch, vạch đứt nối liền, vạch đôi = 1 polyline | Cần ranh giới làn tường minh; v1 không có cách ghi vạch | Job 39: 55 vạch / 26 ảnh |
| v2 | Thêm label `crosswalk` (polygon bao cả dải sọc, đè lên drivable, cắt xe) | Câu hỏi khi gán nhãn: "vạch người đi bộ vẽ thế nào" — v1 chỉ gộp vào drivable | BDD02, BDD07, BDD11, BDD12, BDD13, BDD18 |
| v2 | Vật cản chắn hết bề ngang làn ego: `direct` dừng ở mép dưới, không tạo `direct` thứ hai | Mâu thuẫn giữa "cắt vật cản" và "đúng 1 `direct`" | BDD24: đống tuyết giữa làn ego |
| v2 | Vệt đèn phản chiếu trên đường ướt không phải vạch; tuyết phủ thì không vẽ `lane_marking` | Vạch vẽ theo vệt sáng bị lệch | BDD25, BDD17, BDD24 |
| v2 | Ví dụ `reason` escalate cụ thể; vật gắn trên xe ego che mất đoạn cần quyết → escalate | Ca thật khi gán nhãn | BDD17: giá điện thoại + chữ sơn không đọc được |
| v3 | Cách xác định `direct` khi phía trước có nhiều xe: dò từ sát capô theo vạch làn, không theo xe gần giữa ảnh | Bản nháp label chọn nhầm làn SUV bên trái làm `direct` | BDD12 (bản nháp job 38) |
| v3 | Ngưỡng "vật cản chắn hết bề ngang làn" = ≥ 80% bề ngang làn | Guideline v2 và guideline của peer đều thiếu ngưỡng cho vật cản/che khuất | gold BDD24 d1; guideline peer mục 3, 6 |
| v3 | Checklist tự kiểm bắt buộc cho `boundary`, `decision`, `__undefined__`, cắt xe | Khi nhóm mình làm blind pack của peer, 8/8 polygon giữ nguyên default và người label mang thói quen từ guideline khác | `peer.zip` (BDD08, 16, 23, 24, 25) |
| v3 | Ghi nhận: peer chưa gửi export hợp lệ trên blind pack của nhóm mình; chưa có GTS | Hai file nhận được sai bộ ảnh/schema hoặc không có shape | `07_blind_handoff/peer_feedback.md` mục 0 |
