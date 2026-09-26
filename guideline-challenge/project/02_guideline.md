# Annotation guideline — Ranh giới trái/phải của làn xe chủ (ego lane) tại merge/split, vạch mờ và vạch bị che

**Version:** v2

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

**Tóm tắt 30 giây.** Mỗi ảnh vẽ tối đa hai đường gấp khúc (polyline): `ego_left` trên vạch sơn bên trái làn xe
đang chạy và `ego_right` trên vạch sơn bên phải. Đặt điểm lên **tâm vạch sơn**, vẽ **từ gần xe ra xa**. Bên nào
không có hoặc không tìm được vạch sơn thì không vẽ polyline mà gắn tag `ego_boundary_unknown`. Không chắc thì vẫn
làm theo cách hợp lý nhất và bật `needs_review`.

## 1. Objective + scope

Dữ liệu dùng để huấn luyện và đánh giá model phát hiện làn xe chủ cho chức năng giữ làn (LKA) và cảnh báo lệch làn.
Model cần biết **hai mép của làn mà xe đang chạy**, liên tục từ ngay trước đầu xe ra xa nhất có thể nhìn thấy.

- **Trong scope:** vạch sơn (trắng hoặc vàng; liền, đứt hoặc đôi) giới hạn bên trái và bên phải của làn xe chủ,
  kể cả đoạn bị xe che, đoạn mờ, và đoạn chạy cạnh điểm nhập/tách làn hoặc vùng kẻ sọc.
- **Ngoài scope:** mọi vạch khác (xem mục 5).

**Làn xe chủ** là làn chứa **điểm giữa đáy ảnh** (ngay trước mũi xe, x ≈ 640 trên ảnh rộng 1280 px). Kéo dài tưởng
tượng hai vạch gần nhất xuống đến mép nắp capo: vạch gần nhất nằm bên trái điểm đó là ranh giới trái, vạch gần nhất
nằm bên phải là ranh giới phải.

## 2. Annotation unit

- Đơn vị là **một ảnh tĩnh**.
- Mỗi ảnh có **tối đa một** `ego_left` và **tối đa một** `ego_right`. Không bao giờ vẽ hai `ego_left` trên cùng ảnh.
- Một ranh giới = **một polyline liền**, kể cả khi vạch là vạch đứt hoặc có đoạn bị che (xem mục 3 và 6). Không cắt
  thành nhiều polyline.
- Mỗi ảnh có **tối đa một** tag `ego_boundary_unknown`.

## 3. Geometry rule

- **Công cụ:** Polyline trong CVAT. Không dùng polygon, box hay points.
- **Vị trí điểm:** mỗi điểm đặt trên **tâm bề ngang vạch sơn**. Vạch đôi (hai vạch song song, ví dụ vạch vàng đôi):
  đặt điểm **ở giữa hai vạch**, không đặt lên một trong hai vạch.
- **Hướng vẽ:** điểm đầu tiên ở gần xe (phía dưới ảnh), điểm cuối ở xa (phía trên ảnh).
- **Điểm đầu:** chỗ thấp nhất vạch còn thấy trên mặt đường, ngay trên mép nắp capo. Vạch chạm cạnh trái/phải của ảnh
  trước khi tới nắp capo thì điểm đầu đặt đúng ở cạnh ảnh. Không đặt điểm trên hình phản chiếu trong nắp capo.
- **Điểm cuối:** chỗ xa nhất vạch còn nhìn thấy và còn là ranh giới của làn xe chủ. Không kéo dài quá chỗ đó, không
  đoán tiếp vào sau xe ở cuối đường hay tới điểm tụ.
- **Mật độ điểm:** ít nhất 4 điểm. Đường thẳng: khoảng 80–120 px theo chiều dọc một điểm. Đường cong: thêm điểm cho tới
  khi đoạn nối giữa hai điểm liền nhau vẫn nằm trên phần sơn.
- **Vạch đứt:** nối thẳng qua khoảng trống giữa các đoạn sơn. Khoảng trống của vạch đứt **không** phải là bị che.
- **Đoạn bị che hoặc mờ nằm giữa hai đoạn còn thấy:** nối qua theo hướng của vạch (xem mục 6).
- **Tolerance:** mọi điểm và mọi đoạn nối nằm trên phần sơn của vạch (vạch đôi: nằm giữa hai mép ngoài của cặp vạch);
  điểm cuối lệch không quá 20 px so với chỗ vạch hết nhìn thấy.

## 4. Taxonomy

| Tên | Loại CVAT | Là gì | Giá trị |
|---|---|---|---|
| `ego_left` | polyline (class) | ranh giới trái của làn xe chủ | — |
| `ego_right` | polyline (class) | ranh giới phải của làn xe chủ | — |
| `marking` | attribute của polyline | kiểu vạch | `solid` (liền) · `dashed` (đứt) · `double` (hai vạch song song, bất kể liền hay đứt) |
| `visibility` | attribute của polyline | mức nhìn thấy trong đoạn đã vẽ | `visible` · `occluded` (bị vật che) · `faded` (khó thấy vì mọi lý do khác: sơn mòn, tối, mưa, loá) |
| `topology` | attribute của polyline | ranh giới có chạy cạnh điểm nhập/tách làn không | `normal` · `merge_split` |
| `needs_review` | checkbox của polyline và của tag | cần người khác xem lại | bật / tắt (mặc định tắt) |
| `ego_boundary_unknown` | tag cấp ảnh | không vẽ được ranh giới ở một hoặc hai bên | — |
| `side` | attribute của tag | bên nào không vẽ được | `left` · `right` · `both` |

- Trái và phải là **class** vì downstream dùng riêng từng bên và mỗi bên có tối đa một đường. Kiểu vạch, mức nhìn
  thấy, merge/split là **attribute** của cùng một đường.
- `marking`, `visibility`, `topology`, `side` mặc định là `__undefined__`. **Phải chọn giá trị**, để
  `__undefined__` là lỗi.
- Vạch đổi kiểu dọc đường (ví dụ gần xe là vạch đứt, xa hơn là vạch liền): `marking` lấy theo **đoạn gần xe nhất**.
- **Phân biệt liền / đứt khi khó nhìn** (đêm, mưa, xa): chỉ chọn `dashed` khi thấy **ít nhất 2 đoạn sơn tách nhau bởi
  khoảng trống**; chỉ chọn `solid` khi thấy **một đoạn sơn liền dài hơn 2 lần** một đoạn vạch đứt bình thường ở cùng
  khoảng cách. Không đủ bằng chứng cho cả hai: chọn theo phán đoán tốt nhất và bật `needs_review`.
- `visibility` có nhiều trạng thái trong cùng một đường: chọn theo thứ tự ưu tiên `occluded` > `faded` > `visible`.
- `topology = merge_split` chỉ khi trong đoạn đã vẽ, phía bên kia của vạch có **vạch sơn** của vùng kẻ sọc (gore),
  của nhánh rẽ/lối ra, hoặc của làn đang nhập vào. Còn lại là `normal`.
- **Không tính là merge_split:** bó vỉa, đảo giao thông bê tông, dải phân cách, lề đường, làn đỗ xe, lối vào cây xăng
  hay bãi đỗ, giao lộ. Những thứ này không phải vạch sơn của điểm nhập/tách làn.

## 5. Inclusion / exclusion

**Bắt buộc vẽ:**

- Vạch sơn gần nhất bên trái và gần nhất bên phải làn xe chủ (mục 1), màu gì cũng vẽ.
- Vạch đi qua bóng cây/bóng cầu nhưng vẫn nhìn thấy: vẽ, `visibility = visible`.
- Mép trong của vùng kẻ sọc (gore) khi mép đó giáp làn xe chủ: vẽ trên vạch liền viền vùng kẻ sọc,
  `topology = merge_split`.

**Không vẽ (ignore):**

- Vạch của các làn khác, kể cả vạch viền mép đường ở làn xa hơn.
- Sọc chéo hay chữ V bên trong vùng kẻ sọc; vạch của nhánh rẽ/lối ra khi làn xe chủ đi thẳng.
- Vạch qua đường, vạch dừng, mũi tên, chữ và ký hiệu sơn trên mặt đường.
- Bó vỉa, lề cỏ, mép nhựa, dải phân cách bê tông, hàng xe đỗ. Đây không phải vạch sơn.
- Hình phản chiếu trên nắp capo, vệt đèn phản chiếu trên đường ướt.
- Vạch bên kia giao lộ, **khi giữa nắp capo và vạch qua đường có vạch làn**: vẽ vạch làn đó và dừng polyline ở mép
  gần của vạch qua đường, không vẽ tiếp sang bên kia.

**Xe đã ở sát vạch qua đường hoặc trong giao lộ** (giữa nắp capo và vạch qua đường không có vạch làn nào): vẽ vạch
của làn **phía bên kia vạch qua đường mà xe đang hướng thẳng vào** (làn chứa điểm giữa đáy ảnh khi kéo thẳng lên).
Điểm đầu đặt ở mép **xa** của vạch qua đường, không kéo polyline băng qua vạch qua đường.

**Tại điểm nhập/tách làn:** ranh giới đi theo **làn mà xe đang ở trong**. Làn xe chủ đi thẳng, bên cạnh có nhánh tách
ra: vẽ theo vạch của làn đi thẳng, bỏ vạch của nhánh. Xe đang ở trong chính làn lối ra: vẽ theo vạch của làn lối ra.
Không phân biệt được xe đang ở làn nào: xem mục 7.

## 6. Visibility / occlusion

| Tình huống | Làm gì | `visibility` |
|---|---|---|
| Vạch rõ từ đầu đến cuối (kể cả khoảng trống của vạch đứt) | vẽ bình thường | `visible` |
| Một đoạn giữa bị xe/vật che, hai đầu đoạn che vẫn thấy vạch | nối thẳng qua đoạn bị che theo hướng của vạch | `occluded` |
| Vạch bị che từ một chỗ trở ra xa, không thấy lại | dừng ở điểm cuối còn thấy, không đoán tiếp | giữ theo đoạn đã vẽ |
| Từ **1/3 chiều dài** đoạn đã vẽ trở lên không thấy rõ sơn — do sơn mòn/tróc, trời tối, mưa ướt, loá đèn — nhưng vẫn dò được đường đi của vạch (không tính khoảng trống của vạch đứt) | vẽ trên phần sơn còn thấy | `faded` |
| Dưới 1/3 chiều dài khó thấy | vẽ bình thường | `visible` |
| Bên đó **không đủ sơn để dựng đường** — không có ít nhất 2 đoạn sơn tách nhau trên cùng một đường, và cũng không có đoạn sơn liền nào dài từ 50 px trở lên (tuyết phủ, tối hẳn, mòn hết, không có vạch) | không vẽ bên đó, gắn tag unknown | — |
| Chỉ thấy 2–3 vệt sáng mà không chắc là sơn hay phản chiếu | vẽ theo các vệt đó | `faded`, bật `needs_review` |
| Vạch bị cắt ở cạnh ảnh | điểm đầu đặt ở cạnh ảnh | theo đoạn đã vẽ |
| Vạch ở xa, nhỏ | vẽ tới chỗ còn phân biệt được vạch với mặt đường | theo đoạn đã vẽ |

Đoạn bị che ở **gần xe** (xe bên cạnh che phần dưới của vạch) mà phía trên vẫn thấy: bắt đầu polyline từ chỗ thấp
nhất còn thấy vạch, không đoán xuống tới nắp capo. `occluded` chỉ dùng khi đoạn bị che nằm **giữa** hai đoạn đã vẽ;
che ở đầu hoặc ở cuối thì chỉ cắt ngắn polyline, `visibility` tính theo đoạn đã vẽ.

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | tìm được vạch sơn là ranh giới làn xe chủ | polyline `ego_left` / `ego_right` với đủ attribute |
| **IGNORE** | vạch nằm ngoài scope (mục 5) | không có polyline nào trên vạch đó |
| **UNKNOWN** | một bên **không có vạch sơn** (chỉ có bó vỉa, xe đỗ, mép đường), hoặc bên đó **không đủ sơn để dựng đường** (dưới 2 đoạn sơn tách nhau và không có đoạn sơn liền nào dài từ 50 px) | không vẽ polyline bên đó; thêm tag `ego_boundary_unknown`, `side` = `left` / `right` / `both` |
| **ESCALATE** | có vạch nhưng **không chắc** nó có phải ranh giới làn xe chủ: xe đang đè vạch hoặc đang chuyển làn, không rõ xe ở làn đi thẳng hay làn lối ra, hai vạch gần nhau không rõ vạch nào của làn mình | vẫn vẽ polyline theo phán đoán tốt nhất, bật `needs_review` trên polyline đó |

- Ảnh đêm, mưa hay loá **không tự động** là UNKNOWN: đếm đoạn sơn theo tiêu chí ở trên. Thấy từ 2 đoạn sơn tách nhau,
  hoặc một đoạn sơn liền dài từ 50 px, thì vẽ (`faded` nếu khó thấy); không đủ thì gắn tag unknown.
- Không chắc một vệt là sơn hay phản chiếu đèn: làm theo phán đoán tốt nhất (vẽ hoặc gắn tag) và **luôn** bật
  `needs_review` trên shape hoặc tag đó.
- Xe **đè đúng lên** một vạch (điểm giữa đáy ảnh nằm trên vạch): chọn làn phía mũi xe đang hướng tới, bật
  `needs_review` trên cả hai polyline.
- Không để trống một bên mà không có tag: mỗi bên phải có **hoặc** polyline **hoặc** tag unknown.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh.

## 9. Examples

Nhóm peer không nhận các ảnh này, nên mỗi ví dụ mô tả đủ bằng chữ. Toạ độ tính theo ảnh 1280×720, gốc ở góc trên
trái, chỉ là khoảng.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD14 | Cao tốc nhiều làn, trời sáng. Hai bên làn xe chủ đều là vạch trắng đứt; xa hơn bên phải có vạch trắng liền viền mép đường | `ego_left`: dashed, visible, normal, bắt đầu khoảng (300, 480). `ego_right`: dashed, visible, normal, bắt đầu khoảng (650, 495). **Không** vẽ vạch trắng liền bên phải | Mục 1 (vạch gần nhất), mục 3 (nối qua khoảng trống vạch đứt), mục 5 |
| BDD20 | Đường khu dân cư. Bên trái là vạch vàng đôi; bên phải là vạch trắng liền, xa hơn là một vạch trắng liền thứ hai sát hàng xe đỗ. Giá hút kính che giữa đáy ảnh | `ego_left`: double, visible, normal, điểm đặt **giữa** hai vạch vàng, từ khoảng (230, 583) tới khoảng (580, 355). `ego_right`: solid, visible, normal, trên vạch trắng trong, từ khoảng (880, 578). **Không** vẽ vạch trắng sát xe đỗ | Mục 3 (vạch đôi), mục 5 (chỉ vạch gần nhất) |
| BDD08 | Cao tốc. Bên trái vạch vàng liền; bên phải vạch trắng đứt. Xe trắng phía trước che phần xa của vạch vàng | `ego_left`: solid, visible, normal, từ khoảng (0, 615) tới khoảng (395, 380), **dừng** ở chỗ xe trắng bắt đầu che. `ego_right`: dashed, visible, normal. Xe bên phải nằm ở làn bên cạnh, không che vạch | Mục 3 (điểm đầu ở cạnh ảnh, điểm cuối), mục 6 (bị che tới cuối thì dừng) |
| BDD05 | Đường có vạch vàng liền bên trái. Bên phải là vạch trắng liền, bên kia vạch là vùng kẻ sọc trắng | `ego_left`: solid, visible, normal. `ego_right`: solid, visible, **merge_split**, trên vạch trắng liền giáp làn, từ khoảng (895, 600) tới khoảng (615, 350). **Không** vẽ sọc trong vùng kẻ sọc | Mục 4 (`topology`), mục 5 (gore) |
| BDD01 | Cao tốc nhiều làn, bên phải xa có lối ra với vùng kẻ sọc. Vạch đứt bên phải làn xe chủ: đoạn xa thấy rõ, đoạn gần gần như mòn hết, chỉ còn vệt nhỏ | `ego_left`: dashed, visible, normal. `ego_right`: dashed, **faded**, normal (lối ra cách làn xe chủ một làn nên không phải merge_split). **Không** vẽ vạch lối ra và vùng kẻ sọc | Mục 5 (lối ra ngoài scope), mục 6 (faded) |
| (mô tả) | Phố một chiều, không có vạch sơn nào giữa hai hàng xe đỗ | Không có polyline. Tag `ego_boundary_unknown`, `side = both` | Mục 7 (UNKNOWN) |
| BDD26 | Ảnh đêm. Bên trái làn xe chủ là bó vỉa và đảo giao thông tại giao lộ, không có vạch kẻ sọc | Polyline bên trái (nếu có): `topology = normal` — bó vỉa, đảo bê tông không phải vạch sơn gore. Liền hay đứt, vẽ hay unknown: đếm đoạn sơn theo mục 4 và mục 6 | Mục 4 (không tính là merge_split), mục 6 |
| BDD17 | Phố trời mưa, đường ướt, bên trái là xe đỗ sát lề, không có vạch kẻ sọc hay nhánh rẽ | `ego_left`: `topology = normal` — xe đỗ và lề phố không phải làn nhập/tách. Vạch khó thấy do đường ướt tính vào `faded` nếu từ 1/3 chiều dài trở lên | Mục 4, mục 6 |

## 10. Common mistakes

1. **Vẽ nhầm vạch của làn bên cạnh.** Luôn kiểm tra lại: kéo dài vạch xuống đáy ảnh, nó phải là vạch gần nhất hai
   bên điểm giữa đáy ảnh.
2. **Đi theo nhánh lối ra hoặc vạch viền vùng kẻ sọc phía xa** khi làn xe chủ đi thẳng. Đây là lỗi nặng nhất: model
   sẽ học rằng làn xe chủ rẽ ra lối ra.
3. **Cắt polyline ở khoảng trống của vạch đứt** hoặc chỗ bị che. Một ranh giới là một polyline.
4. **Đặt điểm lên một vạch của vạch đôi** thay vì ở giữa hai vạch.
5. **Kéo dài quá chỗ nhìn thấy:** đoán tiếp vạch vào sau xe ở cuối đường, hay qua giao lộ.
6. **Đặt điểm trên hình phản chiếu trong nắp capo.**
7. **Để attribute ở `__undefined__`.**
8. **Thiếu một bên mà không có tag unknown.** Mỗi bên phải có polyline hoặc tag.
9. **Quá ít điểm ở đoạn cong**, khiến đoạn nối cắt ra ngoài phần sơn.
10. **Coi bó vỉa, mép đường là ranh giới.** Không có vạch sơn thì là UNKNOWN.
11. **Chọn `merge_split` vì bó vỉa, đảo bê tông hay làn đỗ xe.** Chỉ vạch sơn của gore, nhánh rẽ, làn nhập mới tính.
12. **Coi mưa, tối là `visible`** chỉ vì vạch "vẫn nhìn ra". Khó thấy từ 1/3 chiều dài trở lên thì là `faded`.
13. **Cố vẽ khi không đủ sơn để dựng đường** (dưới 2 đoạn sơn tách nhau và không có đoạn liền nào dài từ 50 px). Đó
    là UNKNOWN; phân vân thì bật `needs_review`.
14. **Kéo polyline băng qua vạch qua đường.** Có vạch làn trước vạch qua đường thì dừng ở mép gần; không có thì bắt đầu
    ở mép xa (mục 5).

**Tự kiểm trước khi export:** (1) mỗi ảnh, mỗi bên có đúng một polyline **hoặc** tag unknown; (2) không còn
attribute nào là `__undefined__` — mở từng shape trong danh sách Objects bên phải để xem; (3) bấm **Ctrl+S** rồi mới
export.
