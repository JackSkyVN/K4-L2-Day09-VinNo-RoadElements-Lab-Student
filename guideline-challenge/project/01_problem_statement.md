# Problem statement + downstream contract

Tối đa nửa trang, viết **trước khi mở CVAT**. Đây là bằng chứng của gate G1 (topic lock). Thay mọi placeholder
mới là xong.

## Bài toán

Vẽ ranh giới trái và phải của làn xe chủ (ego lane) bằng polyline chạy dọc tâm vạch sơn trên ảnh dashcam BDD100K;
khó ở ba chỗ: vạch mờ/đứt đoạn, vạch bị xe phía trước đè che, và điểm nhập/tách làn (merge/split, lối ra) nơi không
rõ ranh giới đi theo làn chính hay theo nhánh rẽ.

## Downstream contract

1. **Downstream task / model / user là ai?** Model phát hiện làn xe chủ cho chức năng giữ làn (LKA) và cảnh báo
   lệch làn (LDW). Người dùng dữ liệu là kỹ sư huấn luyện và đánh giá model đó.
2. **Output annotation nào thực sự cần?** Mỗi ảnh tối đa hai polyline: `ego_left` và `ego_right`, điểm đặt trên tâm
   vạch sơn, vẽ từ gần xe ra xa. Thuộc tính: `marking` (solid / dashed / double), `visibility` (visible / occluded /
   faded), `topology` (normal / merge / split). Bên nào không xác định được thì gắn tag ảnh `ego_boundary_unknown`
   với thuộc tính `side` (left / right / both).
3. **Failure nào gây hậu quả lớn nhất?** Ranh giới đi theo nhánh tách/lối ra hoặc lấy nhầm vạch của làn bên cạnh:
   model học rằng làn xe chủ rẽ sai hướng, LKA sẽ lái xe lệch khỏi làn. Lỗi lớn thứ hai: bỏ sót ranh giới có thật chỉ
   vì một đoạn bị xe che.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Người vẽ bật checkbox `needs_review` trên
   polyline hoặc trên tag `ego_boundary_unknown`. Spec owner của nhóm xem lại các shape có cờ này, quyết định, và ghi
   thành thẻ trong `04_edge_cases/edge_case_cards.md`.

## Scope

- **Trong scope (bắt buộc label):** vạch sơn (trắng hoặc vàng; liền, đứt hoặc đôi) giới hạn hai bên làn xe đang
  chạy; đoạn vạch bị che hoặc mờ nằm giữa hai đoạn còn nhìn thấy (vẽ nối qua, đánh `visibility` tương ứng); ranh giới
  của làn chính tại điểm merge/split.
- **Ngoài scope (ignore):** vạch của các làn khác, vạch qua đường, vạch dừng, mũi tên và chữ sơn trên đường, sọc chéo
  bên trong vùng kẻ sọc (gore), bó vỉa/mép đường không có sơn, hình phản chiếu vạch trên nắp capo.
- **Geometry tolerance:** mọi điểm của polyline nằm trên phần sơn của vạch (vạch đôi: nằm giữa hai vạch); điểm đầu
  ở ngay trên mép nắp capo, điểm cuối không vượt quá chỗ cuối cùng còn nhìn thấy vạch và lệch không quá 20 px so với
  chỗ đó.

## Output chấm được

- **LABEL:** có polyline `ego_left` / `ego_right` trên ảnh (class = bên trái hay bên phải).
- **IGNORE:** không có polyline nào đặt trên vạch ngoài scope (ví dụ vạch lối ra, vạch làn bên cạnh).
- **UNKNOWN:** tag `ego_boundary_unknown` với `side` đúng.
- **ESCALATE:** `needs_review` = true trên shape hoặc tag.
- **Attribute:** giá trị `marking`, `visibility`, `topology` của từng polyline.
- **Geometry:** vị trí các điểm so với tâm vạch, điểm đầu và điểm cuối theo tolerance ở trên.

Tất cả đều nằm trong file export CVAT for images 1.1 (thẻ `<polyline>`, `<tag>`, `<attribute>`).

## Dữ liệu và giới hạn

Nguồn `data/bdd100k`: 26 ảnh tĩnh 1280×720, không phải video nên không dùng thông tin theo thời gian. Dự kiến dùng
khoảng 16 ảnh: 3–5 example, 5–8 calibration, 4–5 blind. Giới hạn đã biết: phần lớn là đường Mỹ ban ngày, chỉ vài
ảnh đêm/hoàng hôn/mưa/tuyết; nắp capo và phản chiếu chiếm đáy nhiều ảnh; một số ảnh phố có vạch qua đường cắt ngang
ranh giới làn; có ảnh đường khu dân cư gần như không có vạch.
