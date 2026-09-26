# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: 10 mục, taxonomy `ego_left`/`ego_right` + tag `ego_boundary_unknown`, rule vạch đôi, vạch bị che, gore/lối ra, 5 ví dụ | Chuyển downstream contract trong `01_problem_statement.md` thành rule; gộp merge/split thành một giá trị `merge_split` vì ảnh tĩnh khó biết làn đang nhập hay tách | `01_problem_statement.md`; ảnh BDD14, BDD20, BDD08, BDD05, BDD01 |
