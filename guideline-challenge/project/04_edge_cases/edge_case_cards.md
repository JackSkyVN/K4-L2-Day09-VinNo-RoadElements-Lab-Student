# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: EC01
Sample: BDD16 (blind)
Scene: Dưới cầu vượt, đường cong sang trái, chiều tối
Observation: Bên trái làn xe chủ là vạch vàng liền, bên kia vạch là vùng kẻ sọc vàng (gore); xa hơn bên trái có làn khác với xe trắng. Bên phải là vạch trắng liền dọc tường cầu
Decision: LABEL (ego_left, ego_right) + IGNORE (sọc trong gore)
Expected: ego_left trên vạch vàng liền gần xe nhất, topology=merge_split, điểm cuối ở cạnh trái ảnh; không polyline nào trên sọc chéo vàng; ego_right trên vạch trắng liền, điểm đầu ở cạnh phải ảnh (gold BDD16 d1–d4)
Rationale: Downstream contract mục 3 — đi theo gore/nhánh là lỗi nặng nhất vì LKA sẽ học làn xe chủ lệch vào vùng cấm
Common mistake: Đặt điểm lên mép ngoài của vùng kẻ sọc hoặc lên sọc chéo; chọn topology=normal vì không để ý vùng sọc
Diversity: critical, conflict

---

CASE ID: EC02
Sample: BDD15 (blind)
Scene: Phố đô thị trời nắng, bên phải có bó vỉa sơn đỏ-trắng và xe đỗ
Observation: Xe nằm sát vạch trắng đứt bên phải, hướng xe chếch sang phải như đang chuyển làn. Kéo vạch xuống mép nắp capo được x≈730, điểm giữa đáy ảnh x≈640 vẫn ở bên trái vạch
Decision: ESCALATE
Expected: Vạch đứt gần điểm giữa đáy ảnh gán ego_right, needs_review=true; không polyline trên bó vỉa (gold BDD15 d1–d2)
Rationale: Không chắc xe thuộc làn nào — downstream cần biết mẫu này không chắc chắn để lọc hoặc review (contract mục 4)
Common mistake: Coi làn bên phải là làn xe chủ và gán vạch đứt thành ego_left; vẽ bó vỉa đỏ-trắng thành ego_right; quên bật needs_review
Diversity: escalation, ambiguity

---

CASE ID: EC03
Sample: BDD24 (blind)
Scene: Phố có tuyết, đường ướt, nắp capo phản chiếu mạnh
Observation: Một vạch trắng liền dài ~115 px ở bên trái điểm giữa đáy ảnh; hình phản chiếu của vạch hiện trên nắp capo; bên phải chỉ có đống tuyết và bó vỉa
Decision: LABEL (ego_left) + UNKNOWN (right)
Expected: ego_left trên vạch trắng x≈490–515, y≈460–575, không có điểm dưới y≈580; tag ego_boundary_unknown side=right (gold BDD24 d1–d3)
Rationale: Contract mục 2 — mỗi bên phải có polyline hoặc tag để downstream biết bên nào thiếu ranh giới thật
Common mistake: Kéo polyline xuống vệt phản chiếu trên nắp capo; coi mép đống tuyết/bó vỉa là ego_right; bỏ trống bên phải không có tag
Diversity: low_visibility, truncation, negative

---

CASE ID: EC04
Sample: BDD09 (blind)
Scene: Đường dẫn cao tốc cong sang phải, có hai xe SUV phía trước
Observation: Vạch vàng liền bên trái chạy tới chỗ SUV xám thì bị che tới cuối; vạch trắng đứt bên phải cong theo đường; có biển báo vàng phía trước nhưng không có vạch sơn gore
Decision: LABEL
Expected: ego_left solid, dừng ở chỗ xe che; ego_right dashed, ≥ 4 điểm bám đoạn cong; topology=normal hai bên (gold BDD09 d1–d4)
Rationale: Model cần hình dạng cong đúng để LKA bám cua; đoán tiếp sau xe hoặc nối thẳng qua đoạn cong làm sai hướng làn
Common mistake: Nối thẳng hai điểm xa nhau ở đoạn cong; nối sang vạch liền mép phải; chọn merge_split vì biển báo
Diversity: occlusion, small_far

---

CASE ID: EC05
Sample: BDD05 (example)
Scene: Đường có vạch vàng liền bên trái, bên phải là vạch trắng liền giáp vùng kẻ sọc trắng
Observation: Vùng kẻ sọc nằm ngay bên kia vạch phải của làn xe chủ
Decision: LABEL + IGNORE (sọc)
Expected: ego_right solid, visible, topology=merge_split, trên vạch trắng liền giáp làn; không vẽ sọc
Rationale: Đánh dấu merge_split cho downstream biết đoạn ranh giới có rủi ro nhầm nhánh
Common mistake: Chọn topology=normal; vẽ lên sọc
Diversity: critical, conflict — đã chép thành ví dụ BDD05 ở mục 9 guideline

---

CASE ID: EC06
Sample: BDD01 (example)
Scene: Cao tốc nhiều làn, lối ra có gore ở xa bên phải
Observation: Lối ra cách làn xe chủ một làn; vạch đứt bên phải làn xe chủ đoạn gần gần như mòn hết
Decision: LABEL + IGNORE (vạch lối ra)
Expected: ego_right dashed, faded, topology=normal; không vẽ vạch lối ra và gore
Rationale: Chỉ ranh giới của làn xe chủ có ích cho LKA; gore không giáp làn mình thì không phải merge_split
Common mistake: Vẽ theo vạch lối ra; chọn merge_split vì thấy gore ở đâu đó trong ảnh
Diversity: negative, low_visibility — đã chép thành ví dụ BDD01 ở mục 9 guideline

---

CASE ID: EC07
Sample: BDD20 (example)
Scene: Đường khu dân cư, vạch vàng đôi bên trái, hai vạch trắng liền bên phải
Observation: Vạch trắng trong là ranh giới làn; vạch trắng ngoài sát hàng xe đỗ
Decision: LABEL + IGNORE (vạch trắng ngoài)
Expected: ego_left double, điểm đặt giữa hai vạch vàng; ego_right solid trên vạch trắng trong
Rationale: Tâm ranh giới phải nhất quán cho downstream; chọn vạch ngoài làm làn rộng sai
Common mistake: Đặt điểm lên một trong hai vạch vàng; chọn vạch trắng sát xe đỗ
Diversity: conflict — đã chép thành ví dụ BDD20 ở mục 9 guideline

---

CASE ID: EC08
Sample: BDD26 (calibration)
Scene: Ảnh đêm, đèn đường và đèn xe gây loá
Observation: Chỉ thấy vài vệt sơn ở giữa đường; bên trái là bó vỉa và đảo giao thông tại giao lộ. Calibration: 3 người chọn khác nhau marking, topology, visibility; không ai dùng unknown hay needs_review
Decision: LABEL hoặc UNKNOWN tuỳ số đoạn sơn đếm được; ESCALATE khi không chắc vệt là sơn
Expected: topology=normal bên trái (bó vỉa, đảo không phải vạch sơn gore); đủ sơn (≥ 2 đoạn tách nhau hoặc 1 đoạn liền ≥ 50 px) thì vẽ với visibility=faded, không đủ thì tag unknown; không phân biệt được liền/đứt thì needs_review
Rationale: Contract mục 4 — dữ liệu đêm không chắc chắn phải được đánh dấu để downstream lọc
Common mistake: Chọn merge_split vì bó vỉa; chọn visible vì vạch "vẫn nhìn ra"; cố vẽ không bật needs_review
Diversity: low_visibility, ambiguity — rule đã thêm vào v2 (mục 4, 6, 7) và ví dụ BDD26 ở mục 9

---

CASE ID: EC09
Sample: BDD12 (calibration)
Scene: Xe đang ở sát vạch qua đường tại giao lộ
Observation: Giữa nắp capo và vạch qua đường không có vạch làn nào; vạch làn chỉ xuất hiện phía bên kia vạch qua đường. Calibration: cả 3 người đều vẽ từ mép xa vạch qua đường trong khi rule v1 bảo dừng ở mép gần
Decision: LABEL
Expected: Vẽ vạch của làn phía bên kia mà xe đang hướng thẳng vào, điểm đầu ở mép xa vạch qua đường, không băng qua vạch qua đường
Rationale: LKA cần biết làn xe sắp đi vào; rule v1 không phủ trường hợp này
Common mistake: Kéo polyline băng qua vạch qua đường xuống nắp capo; gắn unknown cả hai bên dù phía trước có vạch rõ
Diversity: truncation, conflict — rule đã sửa ở mục 5 v2

---

CASE ID: EC10
Sample: BDD17 (calibration)
Scene: Phố trời mưa, kính có giọt nước, giá hút kính che góc trái
Observation: Đường ướt làm vạch loá và khó thấy; bên trái là xe đỗ sát lề. Calibration: người chọn visible, người chọn faded; một người chọn merge_split bên trái
Decision: LABEL
Expected: topology=normal (xe đỗ, lề phố không phải làn nhập/tách); visibility=faded nếu từ 1/3 chiều dài đoạn vẽ trở lên khó thấy do mưa
Rationale: faded phải có nghĩa thống nhất để downstream dùng làm trọng số
Common mistake: Coi mưa là visible vì vạch vẫn nhìn ra; chọn merge_split vì xe đỗ bên trái
Diversity: low_visibility, occlusion (giá hút kính) — rule đã thêm vào v2 và ví dụ BDD17 ở mục 9

---
