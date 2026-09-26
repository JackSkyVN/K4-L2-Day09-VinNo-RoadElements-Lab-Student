# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: 10 mục, taxonomy `ego_left`/`ego_right` + tag `ego_boundary_unknown`, rule vạch đôi, vạch bị che, gore/lối ra, 5 ví dụ | Chuyển downstream contract trong `01_problem_statement.md` thành rule; gộp merge/split thành một giá trị `merge_split` vì ảnh tĩnh khó biết làn đang nhập hay tách | `01_problem_statement.md`; ảnh BDD14, BDD20, BDD08, BDD05, BDD01 |
| v2 | `faded` gồm mọi lý do khó thấy trừ bị vật che, thêm ngưỡng 1/3 chiều dài | Calibration: 3 người chọn visible/faded khác nhau ở ảnh mưa, đường vá, ảnh đêm vì v1 chỉ nói "sơn mòn" | `06_calibration_report.csv` dòng BDD17, BDD07 visibility; `06_calibration_measure.csv` |
| v2 | `merge_split` chỉ khi có vạch sơn gore/nhánh/làn nhập; thêm danh sách không tính (bó vỉa, đảo, làn đỗ xe…); thêm ví dụ BDD26, BDD17 | Hai người chọn merge_split vì bó vỉa/đảo và xe đỗ bên trái | Report dòng BDD26, BDD17 topology |
| v2 | Tiêu chí phân biệt liền/đứt khi khó nhìn (≥ 2 đoạn sơn tách nhau mới là dashed), không chắc thì needs_review | BDD26 ảnh đêm: vianh chọn ngược kiểu vạch với hai người còn lại | Report dòng BDD26 marking |
| v2 | Rule vạch qua đường tách hai trường hợp: có vạch làn trước vạch qua đường thì dừng ở mép gần; xe đã sát vạch qua đường thì vẽ làn phía bên kia từ mép xa | BDD12: cả 3 người vẽ phía bên kia vạch qua đường vì rule v1 không áp dụng được khi xe đã ở sát vạch qua đường | Report dòng BDD12 (kiểm toạ độ trong 3 export) |
| v2 | Tiêu chí UNKNOWN đo được: không đủ sơn để dựng đường (dưới 2 đoạn sơn tách nhau và không có đoạn liền nào dài từ 50 px) thì gắn tag unknown; không chắc sơn hay phản chiếu thì luôn bật needs_review | Cả 7 ảnh không ai dùng tag unknown hay needs_review, kể cả ảnh đêm — tiêu chí v1 quá chủ quan | Report dòng BDD26 unknown + needs_review |
| v2 | Thêm lỗi thường gặp 11–13 và checklist tự kiểm trước export | Lần calib đầu một người để `__undefined__` toàn bộ BDD17 | Report dòng BDD17 execution_error |
| v3 | Rule vật cản trên mặt đường (tuyết, cọc, rào, xe đỗ): không phải vạch sơn, không làm đổi topology, bên chỉ có vật cản là UNKNOWN; thêm ví dụ mô tả; lỗi thường gặp 15–16 | Blind test: peer gán vạch giữa đường ở ảnh có đống tuyết thành ego_right + merge_split, không gắn tag unknown, và hỏi 2 câu về vật cản ở ảnh này | `transfer_score.csv` BDD24 d1, d2; `clarification_log.csv` 2 câu hỏi; `peer_feedback.md` phần 2 |
| v3 | Rule 150 px bắt buộc bật needs_review (ranh giới cách x = 640 dưới 150 px ở mép nắp capo); bỏ tiêu chí "mũi xe hướng tới", thay bằng phần lớn nắp capo nằm bên nào của vạch | Peer không bật needs_review ở ảnh xe sát vạch như đang chuyển làn; peer góp ý "mũi xe hướng chéo" không quyết định được | `transfer_score.csv` BDD15 d1; `peer_feedback.md` câu 3(1) |
| v3 | Ngưỡng faded đo theo chiều dài polyline trên ảnh, có cách ước lượng chia 3 đoạn | Peer: không rõ 1/3 của polyline hay của vạch thật | `peer_feedback.md` câu 2a |
| v3 | Định nghĩa "sát vạch qua đường" bằng việc có/không có đoạn sơn làn trước vạch qua đường; thêm ví dụ BDD12; rule giao lộ nhiều vạch qua đường → UNKNOWN both + needs_review | Peer: "sát" không có tiêu chí, không có ví dụ; giao lộ nhiều nhánh làm rule vỡ | `peer_feedback.md` câu 2b, 3(2); calibration report dòng BDD12 |
| v3 | Tiêu chí solid/dashed đo trực tiếp (có khoảng trống → dashed; đoạn liền ≥ 150 px → solid) thay cho so với "vạch đứt bình thường" | Peer: "vạch đứt bình thường ở cùng khoảng cách" không định nghĩa | `peer_feedback.md` câu 2c |
| v3 | Định nghĩa vạch sơn của gore gồm sọc chéo, chữ V, viền ngoài | Peer: không rõ dạng nào của vùng kẻ sọc được tính | `peer_feedback.md` câu 2d |
| v3 | Nối qua sau xe thì bắt buộc occluded; lỗi thường gặp 17 | Export peer: kéo polyline qua sau xe ở hai ảnh nhưng chọn faded/visible | `peer_output/peer_blind.zip`; ghi chú trong `peer_feedback.md` |
| v3 | Thêm cây quyết định ở đầu guideline, mục thao tác CVAT (Setup tag, chọn side, needs_review ở panel Objects), checklist trước export 5 bước; lỗi thường gặp 18 | Peer đề xuất cây quyết định; peer chỉ ra tag/side/needs_review/`__undefined__` dễ thao tác sai | `peer_feedback.md` câu 4, 5 |
| v3 | Giữ một attribute `visibility` và không vẽ bó vỉa, nhưng thêm câu giải thích lý do vào guideline | Peer đề xuất tách visibility và vẽ bó vỉa — owner từ chối kèm bằng chứng từ downstream contract | `peer_feedback.md` câu 3(3), 3(4); `01_problem_statement.md` |
