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
