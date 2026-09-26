# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** Mỗi batch (một job CVAT) do một annotator làm, một thành viên **khác** review
  (xoay vòng Minh Đức → Vĩ Anh → Duy Anh → Minh Đức). Review 100% ảnh có tag rủi ro (mục dưới) + 20% ngẫu nhiên số ảnh
  còn lại, tối thiểu 5 ảnh mỗi batch. Annotator mới hoặc batch đầu sau khi guideline đổi version: review 100%.
- **Chọn sample theo rule nào** (random, theo tag rủi ro, theo annotator mới…): Bắt buộc review mọi ảnh có ít nhất một
  trong các dấu hiệu sau trong export: shape có `topology = merge_split`, `visibility = occluded` hoặc `faded`,
  `needs_review = true`, tag `ego_boundary_unknown`, hoặc ảnh thiếu một bên mà không có tag. Phần còn lại chọn ngẫu nhiên
  20% theo seed ghi trong issue log để có thể lặp lại.
- **Issue được ghi ở đâu, đóng thế nào:** Reviewer ghi issue bằng tính năng Issue của CVAT ngay trên shape (vị trí lỗi
  nhìn thấy được), đồng thời ghi một dòng vào bảng issue của batch: `sample_id · label · severity · mô tả · rule mục số`.
  Annotator sửa rồi trả lời issue; reviewer mở lại ảnh, xác nhận và bấm Resolve. Issue loại Question không đóng cho tới
  khi spec owner (Minh Đức) trả lời bằng rule viết trong guideline.
- **Khi phát hiện guideline gap thì update và version ra sao:** Issue được chẩn đoán là `guideline_gap` (rule thiếu hoặc
  hiểu được hai cách) thì spec owner sửa `02_guideline.md`, tăng version (v2 → v3 …), ghi một dòng vào
  `08_revision_log.md` kèm sample_id làm bằng chứng, dán lại Guide vào mọi task đang mở. Các ảnh đã làm theo version cũ
  có dính rule vừa đổi thì đưa vào review lại 100%.

## Defect severity

Mapping theo downstream contract ở `01_problem_statement.md`: lỗi làm model học sai **hướng** hoặc **vị trí** làn xe
chủ nặng hơn lỗi thuộc tính.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Ranh giới đặt trên vạch không phải của làn xe chủ, làm model học làn xe chủ sai hướng | ego_left/ego_right đi theo vạch lối ra, sọc gore hoặc vạch của làn bên cạnh; đổi nhãn trái ↔ phải | Sửa ngay; review lại 100% batch của annotator đó; đếm vào critical escape rate |
| Major | Đúng vạch nhưng thiếu/thừa quyết định hoặc sai attribute ảnh hưởng tới train | thiếu một bên mà không có tag unknown; vẽ ở bên không có vạch sơn thay vì tag; `topology` sai; `needs_review` thiếu ở case chuyển làn; polyline băng qua vạch qua đường hoặc kéo xuống phản chiếu nắp capo | Sửa trong batch; nếu > 1 lỗi cùng loại ở cùng annotator thì coaching |
| Minor | Sai hình học nhỏ hoặc attribute ít ảnh hưởng | điểm lệch khỏi phần sơn ở vài chỗ; dưới 4 điểm; `marking` hoặc `visibility` lệch khi ảnh thật sự khó; điểm cuối dừng sớm/muộn hơn 20 px | Sửa khi review; không bắt review lại |
| Question | Rule không trả lời được, hai cách làm đều hợp lý | ảnh loại mới chưa có ví dụ; vạch sơn tạm thời màu cam | Ghi issue Question, gắn `needs_review`; spec owner quyết định và cập nhật guideline |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Boundary accuracy | số bên (trái/phải) có quyết định đúng (polyline trên đúng vạch **hoặc** tag unknown đúng) ÷ tổng số bên đã review (= 2 × số ảnh review) | Downstream cần mỗi bên đúng; đo theo từng bên để một ảnh sai một bên không bị tính như sai cả ảnh |
| Attribute accuracy | số attribute (`marking`, `visibility`, `topology`) đúng ÷ tổng attribute trên các polyline đúng vạch | Tách khỏi boundary accuracy để biết lỗi nằm ở chọn vạch hay ở mô tả vạch |
| Geometry compliance | số polyline mà mọi điểm và đoạn nối nằm trên phần sơn, điểm cuối lệch ≤ 20 px ÷ số polyline review | Kiểm tolerance của mục 3 guideline |
| Escalation recall | số case reviewer thấy thuộc danh sách ESCALATE (mục 7) có `needs_review = true` ÷ tổng case như vậy | Calibration cho thấy không ai dùng needs_review; metric này buộc theo dõi đường escalation |
| `__undefined__` rate | số attribute còn `__undefined__` ÷ tổng attribute | Lỗi thực thi đã gặp ở calibration; phải về 0 trước khi nộp batch |

Metric high-risk tách riêng (ví dụ critical defect escape rate): **Critical escape rate** = số lỗi Critical phát hiện
sau khi batch đã PASS (ở QA vòng sau hoặc do downstream báo) ÷ số ảnh `merge_split` + số ảnh có tag rủi ro trong batch.
Mục tiêu 0; một lỗi escape là mở lại toàn bộ batch đó.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  critical defects = 0 trong mẫu review
  AND boundary accuracy >= 95%
  AND attribute accuracy >= 90%
  AND geometry compliance >= 90%
  AND escalation recall >= 80%
  AND __undefined__ rate = 0
REWORK if: không có critical nhưng một trong các ngưỡng còn lại không đạt
  → annotator sửa toàn batch, reviewer review lại theo cùng sampling
REJECT / ESCALATE if: có >= 1 critical defect, hoặc boundary accuracy < 85%, hoặc cùng một rule bị hiểu sai ở >= 3 ảnh
  → không nhận batch; review 100%; nếu nguyên nhân là guideline_gap thì spec owner sửa guideline, tăng version
    trước khi làm lại
```

Trade-off: Ngưỡng critical = 0 và review 100% ảnh rủi ro tốn thời gian review, nhưng ảnh merge/split và ảnh bị che
chỉ là phần nhỏ của dữ liệu mà lại là nơi lỗi làm LKA lái sai làn, nên dồn công review vào đó rẻ hơn để lỗi lọt vào
model. Attribute và geometry được nới (90%) vì ảnh đêm/mưa thật sự khó và calibration đã cho thấy đồng thuận attribute
chỉ ~84%; ép cao hơn sẽ làm rework nhiều mà không cải thiện downstream tương xứng. Escalation recall 80% chấp nhận một
ít case bị bỏ sót cờ vì các case đó vẫn nằm trong 100% ảnh rủi ro được review.
