# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Không có nhóm peer chính thức trong buổi lab (không nhóm nào ghép cặp với VinNo). Blind test do một
  người **ngoài nhóm**, chưa đọc guideline và chưa thấy gold, thực hiện theo đúng quy trình: nhận `blind-pack.zip`
  (guideline v2), tự tạo task CVAT, label 5 ảnh blind, owner không giải thích rule (2 câu hỏi được ghi trong
  `clarification_log.csv`). Export: `peer_output/peer_blind.zip` (job CVAT tạo 14:21, sửa lần cuối 14:29 ngày 2026-09-26).
- **Người label blind:** Hồ Minh Hậu (ngoài nhóm VinNo)

## 1. Peer trả lời

Chép nguyên ý câu trả lời peer gửi kèm export.

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?** Định nghĩa làn xe chủ ở mục 1 (làn chứa điểm giữa đáy ảnh,
   x ≈ 640) — nhìn giữa đáy ảnh là biết. Ngưỡng UNKNOWN ở mục 6 và 7 (dưới 2 đoạn sơn tách nhau và không có đoạn liền
   ≥ 50 px) — cho phép đếm thay vì cảm tính. Mục 2 (tối đa một `ego_left` + một `ego_right`) — không vẽ thừa. Checklist
   tự kiểm trước export ở cuối mục 10.
2. **Rule nào mơ hồ hoặc phải tự suy diễn?** (a) Ngưỡng `faded` "từ 1/3 chiều dài": 1/3 của polyline đã vẽ hay của
   vạch thật trên đường, phải ước lượng bằng mắt. (b) Mục 5 "xe đã ở sát vạch qua đường": không có tiêu chí "sát" là
   bao nhiêu px, không có ví dụ. (c) Phân biệt `solid` / `dashed` ở mục 4: "đoạn vạch đứt bình thường ở cùng khoảng
   cách" không được định nghĩa. (d) `merge_split`: vùng kẻ sọc có nhiều dạng (vạch chéo ngắn, chữ V, viền ngoài), không
   rõ cái nào tính là vạch sơn của gore.
3. **Sample nào khiến guideline "vỡ"?** Peer nêu 4 tình huống (không gắn sample_id): (1) xe đang đổi làn, điểm giữa đáy
   ảnh nằm trên vạch, mũi xe hướng chéo nên không rõ "làn đang hướng tới"; (2) giao lộ phức tạp nhiều vạch qua đường
   chồng nhau (ngã 5, ngã 6), không rõ mép gần/xa của vạch nào; (3) vạch vừa bị che vừa mờ trong cùng một đoạn, một
   attribute `visibility` không đủ diễn đạt; (4) đường không có vạch, một bên chỉ có bó vỉa — bó vỉa là ranh giới vật lý
   nhưng rule không cho vẽ.
4. **Attribute / default nào trong CVAT dễ gây thao tác sai?** `marking`, `visibility`, `topology` mặc định
   `__undefined__` dễ quên chọn và CVAT không chặn export; `needs_review` là checkbox mặc định tắt, dễ quên bật và không
   có dấu hiệu rõ trên canvas; `side` của tag dễ bị bỏ quên; tag `ego_boundary_unknown` phải thêm ở phần tag của ảnh,
   không vẽ trên canvas nên dễ thêm nhầm chỗ; không có nhắc Ctrl+S trước export.
5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?** Thêm cây quyết định ở đầu guideline, cho mỗi bên trái/phải:
   có vạch sơn không → đếm đoạn sơn (UNKNOWN hay vẽ) → bao nhiêu phần khó thấy (`faded` / `occluded` / `visible`) →
   kiểu vạch (`dashed` / `solid` / `double`) → bên kia vạch có vạch sơn gore/nhánh không (`merge_split` / `normal`).

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

Kết quả chấm (`transfer_score.csv`): 13/16 decision đúng, 2/2 critical đúng, 2/2 geometry đúng. Ba decision sai: BDD24 d1,
BDD24 d2, BDD15 d1.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| **BDD24 d1, d2 sai:** vạch trắng giữa đường (bên trái điểm giữa đáy ảnh) bị gán `ego_right` với `merge_split`; `ego_left` đặt trên mép tuyết/lề trái; không có tag unknown bên phải | guideline gap — guideline không nói gì về vật cản trên mặt đường (đống tuyết, cọc, rào): peer coi đống tuyết như vùng gore và như làn xe khác | accept + revise: thêm rule vật cản trên đường không phải vạch sơn, không làm đổi `topology`, bên chỉ có vật cản/bó vỉa là UNKNOWN; thêm bước tự kiểm "vạch bên trái x = 640 phải là `ego_left`" | `transfer_score.csv` BDD24 d1, d2; 2 câu hỏi về BDD24 trong `clarification_log.csv` |
| **BDD15 d1 sai:** không bật `needs_review` dù xe chếch sát vạch đứt | guideline gap — "đang chuyển làn" ở mục 7 không đo được, peer không thấy mình thuộc case ESCALATE | add escalation rule: bật `needs_review` bắt buộc khi kéo ranh giới xuống mép nắp capo cách x = 640 dưới 150 px (BDD15: ≈ 90 px) | `transfer_score.csv` BDD15 d1 |
| Q2a — ngưỡng 1/3 của `faded` không rõ đo theo cái gì | guideline gap | accept + revise: đo theo chiều dài polyline trên ảnh (px), không tính khoảng trống vạch đứt; thêm cách ước lượng nhanh | Peer feedback câu 2; peer chọn `faded` cho BDD09 `ego_left`, BDD15 `ego_right`, BDD24 `ego_left` |
| Q2b — "sát vạch qua đường" không có tiêu chí | guideline gap | accept + revise: định nghĩa "sát" = giữa mép nắp capo và mép gần vạch qua đường không có đoạn sơn làn nào; thêm ví dụ BDD12 (ảnh calibration) | Peer feedback câu 2; calibration report dòng BDD12 |
| Q2c — "vạch đứt bình thường ở cùng khoảng cách" không định nghĩa | guideline gap | accept + revise: bỏ phép so sánh, dùng tiêu chí trực tiếp: thấy khoảng trống trên cùng một đường → `dashed`; một đoạn liền ≥ 150 px không khoảng trống → `solid`; còn lại theo phán đoán + `needs_review` | Peer feedback câu 2 |
| Q2d — dạng nào của vùng kẻ sọc tính là vạch sơn gore | guideline gap | accept + revise: mọi vạch sơn bên trong hoặc viền vùng kẻ sọc (sọc chéo, chữ V, viền ngoài) đều tính | Peer feedback câu 2 |
| Q3(1) — xe đổi làn, mũi xe chéo | data ambiguity | add escalation rule: cùng rule 150 px ở dòng BDD15; bỏ tiêu chí "làn mũi xe hướng tới", thay bằng làn chứa x = 640 + `needs_review` | Peer feedback câu 3; BDD15 |
| Q3(2) — giao lộ nhiều vạch qua đường chồng nhau | data ambiguity | add escalation rule: nhiều vạch qua đường cắt làn xe chủ mà không xác định được làn phía bên kia → UNKNOWN `side = both` + `needs_review` | Peer feedback câu 3 (không có ảnh loại này trong data, rule phòng trước) |
| Q3(3) — vạch vừa che vừa mờ, một attribute không đủ | — | reject with evidence: downstream contract chỉ cần trạng thái xấu nhất để lọc/giảm trọng số; thứ tự ưu tiên `occluded` > `faded` > `visible` đã có ở mục 4 v2. Giữ nguyên, thêm một câu giải thích lý do | `01_problem_statement.md` mục Downstream contract; guideline mục 4 |
| Q3(4) — bó vỉa là ranh giới vật lý nhưng không được vẽ | — | reject with evidence: bài toán là ranh giới **vạch sơn** cho LKA; bó vỉa ngoài scope (mục 5). Thêm câu giải thích vì sao để annotator không thấy rule vô lý | `01_problem_statement.md` mục Scope |
| Q4 — `__undefined__`, `needs_review`, `side`, vị trí thêm tag, Ctrl+S | guideline gap (hướng dẫn thao tác CVAT) | accept + revise: thêm mục thao tác CVAT ngắn (thêm tag ở đâu, chọn `side`, bật `needs_review` ở panel Objects) và mở rộng checklist trước export | Peer feedback câu 4 |
| Q5 — cây quyết định | guideline gap (cấu trúc tài liệu) | accept + revise: thêm cây quyết định vào đầu guideline v3 | Peer feedback câu 5 |

Ghi chú của owner: ở câu 3 peer nêu tình huống giả định, không chỉ ra sample cụ thể trong 5 ảnh; đối chiếu export và
câu hỏi thì sample làm guideline "vỡ" trên thực tế là **BDD24** (2 câu hỏi, 2 decision sai) và **BDD15** (thiếu
escalation). Ngoài phần gold chấm, export còn cho thấy peer kéo polyline qua sau xe ở BDD09 và BDD16 nhưng chọn
`faded`/`visible` thay vì `occluded` — sẽ làm rõ ở v3 cùng rule `occluded`.
