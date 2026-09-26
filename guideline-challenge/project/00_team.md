# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** VinNo
- **Nhóm peer test bài của mình:** Không có nhóm peer chính thức trong buổi lab. Blind test do một người ngoài nhóm
  (chưa đọc guideline, chưa thấy gold) thực hiện — chi tiết ở `07_blind_handoff/peer_feedback.md`
- **Nhóm mình test bài của:** Không có — không nhóm nào gửi blind pack cho VinNo
- **Problem family:** Lane boundary — ranh giới trái/phải của làn xe chủ (ego lane) tại merge/split, vạch mờ/đứt và vạch bị xe che
- **Nguồn ảnh:** `bdd100k`

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Nguyễn Trọng Minh Đức | emsiCUD | Spec owner | `01_problem_statement.md`, `02_guideline.md`, `08_revision_log.md` |
| Nguyễn Đăng Vĩ Anh | NDViANh | Gold owner + QA plan | `04_edge_cases/edge_case_cards.md`, `04_edge_cases/gold_decisions.csv`, `05_qa_plan.md` |
| Đào Duy Anh | JackSkyVN | CVAT owner + calibration/handoff | `03_cvat_labels.json`, `03_ontology_and_cvat_setup.md`, `sample_pack.csv`, `06_calibration_report.csv`, `07_blind_handoff/`, `09_cvat_export_or_task_reference.txt` |

**Gán nhãn: cả 3 người đều label.** Vai trò ở bảng trên chỉ là người sửa chính file; phần vẽ trên CVAT thì mỗi
người tự làm task của mình: bộ calibration (mỗi người một zip `minhduc.zip`, `vianh.zip`, `duyanh.zip`) và bài blind
của nhóm peer.

### Phân công theo bước

| Bước | Việc | Người làm chính | Cả nhóm |
|---|---|---|---|
| 01 | Chốt bài toán, điền team + problem statement | Minh Đức | Đọc và đồng ý scope |
| 02 | Guideline v1; thẻ edge case bắt đầu ghi | Minh Đức (`02`); Vĩ Anh (thẻ edge case) | Góp trường hợp khó |
| 03 | Chia ảnh `sample_pack.csv`, viết nhãn CVAT, `pack calibration`, tạo task | Duy Anh | Một người chưa dựng task thử mở task |
| 04 | Gán nhãn calibration: mỗi người một task riêng, export zip, rồi chạy `calib` | **Cả 3 người label**; Duy Anh gom zip, chạy `calib`, viết `06_calibration_report.csv` | Không nhìn màn hình nhau khi vẽ |
| 04 | Sửa guideline lên v2, ghi revision log | Minh Đức | Chốt cách sửa các chỗ lệch |
| 05 | Viết `gold_decisions.csv`, chạy `freeze` (chỉ một người), push tag | Vĩ Anh | Mở từng ảnh blind cỡ gốc để soát đáp án |
| 05 | Viết `05_qa_plan.md` | Vĩ Anh | — |
| 06 | Chạy `handoff`, gửi gói cho nhóm peer; ghi câu hỏi vào `clarification_log.csv` | Duy Anh | Không giải thích miệng cho nhóm peer |
| 06 | Gán nhãn bài blind của nhóm peer (15 phút), gửi lại zip + 5 câu nhận xét | **Cả 3 người label** | Vẽ đúng chữ trong guideline của họ, không đoán ý |
| 07 | `score` + điền `transfer_score.csv`, `gts`, chép `peer_feedback.md` | Duy Anh (chạy lệnh); Vĩ Anh (chấm 0/1 theo gold) | — |
| 08 | Guideline v3 + dòng v3 revision log | Minh Đức | Góp chỗ nhóm peer vấp |
| 08 | Đủ ≥ 8 thẻ edge case | Vĩ Anh | — |
| 08 | Điền `09_...txt`, chạy `check`, push cuối | Duy Anh | Xoá hết placeholder còn sót trong file mình phụ trách |
| — | Trình bày 2 phút cuối buổi | Minh Đức (quy tắc không dùng được, đã sửa gì) | Người vẽ bài nhóm khác nói chỗ khó nhất |

Gợi ý chia vai (nhóm 2–3 người thì gộp): **spec owner** (`01`, `02`), **CVAT owner** (`03_*`, `sample_pack.csv`,
`09`), **gold owner** (`04_edge_cases/`), **QA owner** (`05`, `06`, `07_blind_handoff/`). Mỗi file một người sửa
chính để tránh xung đột git. Calibration thì mọi người cùng label.
