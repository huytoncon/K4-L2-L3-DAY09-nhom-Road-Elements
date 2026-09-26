# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:**
  - QA owner review mỗi batch; spec owner review lại mọi ảnh có defect critical.
  - Calibration và blind: review 100% ảnh.
  - Production: review 20% ảnh mỗi batch, cộng 100% ảnh có tag `low_visibility` hoặc có `frame_review = escalate`.
- **Chọn sample theo rule nào:**
  - Ưu tiên ảnh có nguy cơ cao: giao lộ, có vạch vàng, đêm/mưa/tuyết, ảnh annotator mới làm.
  - Phần còn lại chọn ngẫu nhiên theo `sample_id`.
- **Issue được ghi ở đâu, đóng thế nào:**
  - Mỗi defect là một comment trên shape trong CVAT, ghi rule vi phạm (mục guideline) và severity.
  - Annotator sửa rồi reply. Reviewer mở lại ảnh, xác nhận rồi resolve.
  - Batch chỉ đóng khi không còn issue critical/major nào mở.
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Ghi dòng mới vào `08_revision_log.md` kèm `sample_id` làm bằng chứng.
  - Spec owner sửa `02_guideline.md` và tăng version (v2 → v3), rồi dán lại vào Guide của task CVAT.
  - Ảnh đã làm theo rule cũ chỉ làm lại nếu defect ở mức major trở lên.

## Defect severity

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Làm planner cho xe đi vào nơi cấm hoặc chọn sai làn ego | drivable bên kia vạch vàng; sai `direct`; drivable phủ lên người/xe; bỏ sót crosswalk có người | Rework ngay cả ảnh; spec owner review lại |
| Major | Sai vùng/label nhưng không trực tiếp gây đi sai | bỏ sót làn alternative; tô shoulder/gore; thiếu `crosswalk`; `__undefined__`; polygon `direct` thứ hai | Rework shape đó trong batch |
| Minor | Sai hình học trong vùng đúng, hoặc attribute phụ | biên lệch 5–15 px; `boundary` sai; `style` vạch sai | Sửa nếu > 3 minor/ảnh; còn lại ghi nhận để coaching |
| Question | Guideline chưa trả lời được | vùng tối không rõ chiều đi; chữ sơn không đọc được | Escalate cho spec owner, có thể thành rule mới |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Critical escape rate | số defect critical reviewer tìm thấy / số ảnh được review | Đo đúng failure nguy hiểm nhất trong downstream contract |
| IoU `direct` | IoU polygon `direct` của annotator với reviewer, theo từng ảnh | Làn ego là output chính của planner |
| Recall `alternative` | số làn cùng chiều được vẽ / số làn cùng chiều reviewer thấy | Bỏ sót làn làm planner không biết lựa chọn chuyển làn |
| Attribute accuracy | số shape có `area_type`/`color`/`style` đúng / tổng shape | Sai thuộc tính vẫn là nhãn sai dù hình đúng |
| Escalation precision | số escalate hợp lệ / tổng escalate | Tránh lạm dụng escalate để né ca khó |

Metric high-risk tách riêng: **critical escape rate** được báo riêng, không gộp vào điểm trung bình.

## Quality gate

```text
PASS if:
  critical escape rate = 0
  AND IoU direct trung bình ≥ 0.85 và không ảnh nào < 0.70
  AND recall alternative ≥ 0.90
  AND attribute accuracy ≥ 0.95
REWORK if: có ≤ 2 defect critical trong batch, hoặc một chỉ số non-critical dưới ngưỡng
REJECT / ESCALATE if: > 2 defect critical, hoặc IoU direct trung bình < 0.70, hoặc > 20% ảnh bị escalate
```

**Trade-off:**
- IoU 0.85 chứ không cao hơn vì biên xa và biên ở ảnh đêm vốn lệch nhiều giữa người vẽ. Đòi 0.95 sẽ bắt rework ảnh
  đúng về ngữ nghĩa, tốn công mà không giảm rủi ro.
- Critical đặt 0 vì chỉ một lỗi lấn làn ngược chiều nhân lên 100 000 ảnh đã dạy model hành vi nguy hiểm.
- Review 20% production là đủ để phát hiện lỗi hệ thống sớm mà vẫn giữ chi phí. Các ảnh rủi ro cao thì review 100%.
