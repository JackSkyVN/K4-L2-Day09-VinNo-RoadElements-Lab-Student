# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `ego_left` | polyline | class | — | — | — | Ranh giới trái của làn xe chủ; downstream (LKA/LDW) dùng riêng từng bên, mỗi ảnh tối đa một đường |
| `ego_right` | polyline | class | — | — | — | Ranh giới phải của làn xe chủ; tách class với bên trái để không phải suy ra trái/phải từ toạ độ |
| `marking` | — | attribute của `ego_left`, `ego_right` | `__undefined__`, `solid`, `dashed`, `double` | `__undefined__` | không | Kiểu vạch quyết định xe được phép vượt vạch hay không; vạch đôi cần đặt điểm ở giữa |
| `visibility` | — | attribute của `ego_left`, `ego_right` | `__undefined__`, `visible`, `occluded`, `faded` | `__undefined__` | không | Cho downstream biết đoạn nào là nội suy (bị che) hay sơn mờ, để lọc hoặc giảm trọng số khi train/đánh giá |
| `topology` | — | attribute của `ego_left`, `ego_right` | `__undefined__`, `normal`, `merge_split` | `__undefined__` | không | Đánh dấu ranh giới chạy cạnh gore/nhánh rẽ/làn nhập — nơi lỗi critical dễ xảy ra nhất; gộp merge và split vì ảnh tĩnh khó phân biệt |
| `needs_review` | — | attribute của `ego_left`, `ego_right`, `ego_boundary_unknown` | checkbox `true` / `false` | `false` | không | Đường escalation: người vẽ vẫn vẽ phán đoán tốt nhất nhưng gắn cờ cho spec owner xem lại |
| `ego_boundary_unknown` | tag (cả ảnh) | class | — | — | — | Thể hiện quyết định UNKNOWN trong file export: một hoặc hai bên không có vạch sơn hoặc không dò được |
| `side` | — | attribute của `ego_boundary_unknown` | `__undefined__`, `left`, `right`, `both` | `__undefined__` | không | Cho biết bên nào thiếu ranh giới, để mỗi bên luôn có polyline hoặc tag |

## Class hay attribute

- **Class:** `ego_left`, `ego_right`, `ego_boundary_unknown`. Trái và phải là hai đối tượng downstream dùng riêng, mỗi
  bên tối đa một đường, và rule QA khác nhau (thiếu trái hay thiếu phải là hai lỗi khác nhau). `ego_boundary_unknown`
  là tag cấp ảnh vì nó nói về việc *không có* hình vẽ, không có geometry.
- **Attribute:** `marking`, `visibility`, `topology`, `needs_review`, `side`. Đây là thuộc tính của cùng một đường;
  nếu tách thành class sẽ nổ ra 2 × 3 × 3 × 2 = 36 tổ hợp.
- **Mutable:** tất cả là `không` vì task ảnh tĩnh, không có track.
- **Default và bias:** mọi select đều mặc định `__undefined__` để buộc người vẽ chọn. Nếu để default `visible` hay
  `normal`, người vẽ quên đổi sẽ tạo ra vạch "rõ" và "bình thường" im lặng — đúng ở chỗ nguy hiểm nhất (gore, bị
  che). `needs_review` mặc định `false` là chấp nhận được vì bật cờ là hành động chủ động; rủi ro là người vẽ không
  chắc mà quên bật, nên QA sẽ soi riêng các shape có `topology = merge_split` hoặc `visibility = occluded`.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): CVAT 2.74.1 tại http://localhost:8080 (máy Minh Đức)
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `Guideline_challenge` (CVAT máy Minh Đức,
  tạo với guideline v1; tên task không ghi version — lần sau nên đặt dạng `VinNo-calib-v1-<tên>`)
- **Guide của task đã dán `02_guideline.md`?** Có
- **Nhóm dùng Track hay Shape, vì sao:** Shape — task ảnh tĩnh, mỗi ảnh độc lập, không cần nối đối tượng qua frame

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

Các thành viên mở task calibration và label được ngay, không báo vấp chỗ nào về setup (label, tool polyline, attribute,
tag). Ghi chú trung thực: cả 3 thành viên đều đã đọc guideline trước khi mở task, nên đây không phải một setup test
"lạnh" hoàn toàn. Bằng chứng thao tác CVAT từ người ngoài nhóm có ở blind test: Hồ Minh Hậu tự dựng task từ
blind pack và làm được, nhưng chỉ ra tag `ego_boundary_unknown`, `side`, `needs_review` và `__undefined__` dễ thao tác
sai (`07_blind_handoff/peer_feedback.md` câu 4) — đã thêm mục thao tác CVAT vào guideline v3.
