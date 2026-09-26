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
Sample: BDD12
Scene: Phố nhiều làn, vạch vàng đôi bên trái, vạch sang đường ngang đường
Observation: Ego ở làn xe van; SUV đen ở làn cùng chiều bên trái; taxi phía bên kia vạch vàng
Decision: LABEL + IGNORE
Expected: Không có drivable bên trái vạch vàng; direct = làn van; crosswalk x1
Rationale: Critical: lấn làn ngược chiều; chọn sai làn ego
Common mistake: Coi làn SUV là direct vì gần giữa ảnh
Diversity: critical

---

CASE ID: EC10
Sample: BDD26
Scene: Đêm rất tối, bên trái là curb/dải giữa và vùng tối
Observation: Không thấy chiều đi của vùng bên trái
Decision: ESCALATE / IGNORE
Expected: Không có drivable nửa trái ảnh; direct `boundary = estimated`
Rationale: Không đủ bằng chứng; vẽ đoán có thể thành làn ngược chiều
Common mistake: Tô cả vùng tối bên trái là alternative
Diversity: ambiguity, escalation

---
