# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

---

CASE ID: EC01
Sample: BDD01
Scene: Cao tốc nhiều làn, bên phải có vùng gạch chéo (gore) rồi nhánh ra
Observation: Nhánh ra cùng chiều nằm sau vùng gore, ngăn với làn ego bằng vạch trắng liền
Decision: LABEL
Expected: `alternative` polygon riêng cho nhánh ra; vùng gore không vẽ
Rationale: Downstream cần biết mọi làn cùng chiều đi được, kể cả làn không chuyển sang được ngay
Common mistake: Bỏ sót nhánh ra, hoặc tô luôn vùng gore
Diversity: conflict

---

CASE ID: EC02
Sample: BDD10
Scene: Phố 2 chiều, tim đường vạch vàng đứt, bên phải làn xe đạp
Observation: Vạch vàng đứt cho phép vượt, nên dễ nghĩ làn bên kia là alternative
Decision: IGNORE
Expected: Không có drivable_area bên trái vạch vàng; `lane_marking` vàng `dashed`
Rationale: Làn ngược chiều là failure critical của downstream
Common mistake: Gán làn ngược chiều là `alternative` vì vạch đứt
Diversity: critical, ambiguity

---

CASE ID: EC03
Sample: BDD16
Scene: Đường chui tối, chevron vàng bên trái, xe đen phía trước
Observation: Xe đen chiếm giữa làn ego; chevron sát biên trái
Decision: LABEL
Expected: `direct` cắt xe đen, phía sau xe không vẽ; biên trái ở mép vạch vàng giáp chevron
Rationale: Vùng phủ lên xe là vùng không đi được thật
Common mistake: Vẽ xuyên qua xe (rule v1), hoặc tô chevron
Diversity: occlusion, low_visibility

---

CASE ID: EC04
Sample: BDD18
Scene: Đêm, có vạch sang đường, xe đỗ hai bên
Observation: Phần xa tối, không thấy biên làn
Decision: UNKNOWN
Expected: `direct` và `crosswalk` có `boundary = estimated`; polygon dừng ở chỗ còn thấy mặt đường
Rationale: Planner cần biết biên nào là suy đoán
Common mistake: Kéo polygon vào vùng tối tới chân trời
Diversity: low_visibility

---

CASE ID: EC05
Sample: BDD02
Scene: Dừng trước vạch sang đường ở giao lộ, taxi làn trái
Observation: Vạch sang đường nằm trong làn ego; vạch dừng ngay trước
Decision: LABEL
Expected: drivable không bị cắt ở vạch; 1 `crosswalk` bắt đầu từ mép phải taxi; vạch dừng không thuộc crosswalk
Rationale: Crosswalk là vùng giảm tốc, không phải vùng cấm đi
Common mistake: Cắt drivable ở vạch sang đường, hoặc gộp vạch dừng vào crosswalk
Diversity: conflict

---

CASE ID: EC06
Sample: BDD17
Scene: Mưa, giá điện thoại che góc trái, chữ sơn trên làn trái không đọc được
Observation: Không biết làn trái là làn bus hay làn thường
Decision: ESCALATE
Expected: `frame_review decision = escalate`, reason nêu làn trái + nghi ngờ; làn trái không vẽ
Rationale: Gán nhầm làn bus là alternative dạy model đi vào làn cấm
Common mistake: Cắt giá điện thoại ra khỏi polygon, hoặc đoán làn trái là alternative
Diversity: escalation, occlusion

---

CASE ID: EC07
Sample: BDD14
Scene: Cao tốc ban ngày, sedan bạc trong làn ego
Observation: Ca chuẩn; chỉ cần cắt xe và đặt biên đúng tim vạch
Decision: LABEL
Expected: Xem gold BDD14 d1–d4
Rationale: Đo độ chuyển giao cho ca normal
Common mistake: Gộp alternative hai bên ego; tô shoulder bên phải
Diversity: normal

---

CASE ID: EC08
Sample: BDD24
Scene: Phố tuyết, đống tuyết chắn giữa làn ego, xe trắng phía sau đống tuyết
Observation: Phần làn phía sau đống tuyết vẫn nhìn thấy
Decision: LABEL
Expected: `direct` dừng ở mép dưới đống tuyết, không có `direct` thứ hai; `boundary = estimated`
Rationale: Tránh hai polygon direct mâu thuẫn định nghĩa "làn ego"
Common mistake: Vẽ thêm direct phía sau đống tuyết, hoặc tô cả đống tuyết
Diversity: occlusion, conflict

---

CASE ID: EC09
Sample: BDD07
Scene: Phố 2 chiều, vạch vàng đôi sát bên trái làn ego, xe SUV chạy ngược chiều, vạch sang đường nhỏ ở xa
Observation: Mặt đường bên kia vạch vàng liền mạch, cùng màu, dễ tô luôn; vạch sang đường xa chỉ cao ~15 px
Decision: LABEL + IGNORE
Expected: Không có drivable bên trái vạch vàng đôi; direct = làn SUV trắng, cắt xe; crosswalk ở xa vẫn vẽ
Rationale: Critical: lấn làn ngược chiều; bỏ sót crosswalk làm planner không giảm tốc
Common mistake: Tô cả mặt đường bên trái; bỏ sót vạch sang đường nhỏ ở xa
Diversity: critical, small_far

---

CASE ID: EC10
Sample: BDD23
Scene: Phố dân cư không vạch làn, xe đỗ kín hai bên, người đạp xe phía xa
Observation: Không có vạch tim đường; xe đỗ hai bên đều quay đuôi về camera nên nhiều khả năng là đường 1 chiều, nhưng không có biển xác nhận
Decision: LABEL hoặc ESCALATE
Expected: 1 direct phủ lòng đường giữa hai dãy xe đỗ, không alternative; hoặc frame_review=escalate với reason 1 chiều/2 chiều
Rationale: Không đủ bằng chứng chắc chắn về chiều đi; guideline cho phép escalate
Common mistake: Chia đôi lòng đường, tô nửa trái thành làn ngược chiều hoặc alternative
Diversity: ambiguity, escalation, occlusion

---
