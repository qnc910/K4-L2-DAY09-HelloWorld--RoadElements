# Annotation guideline — Functional Drivable Area for Ego Vehicle

**Version:** v3

Guideline này áp dụng cho ảnh tĩnh BDD100K trong bài Day 9. Mục tiêu là để một annotator mới tạo được cùng kiểu annotation mà không cần tác giả đứng cạnh giải thích.

## 1. Objective + scope

Gán nhãn vùng mặt đường mà xe gắn camera (ego vehicle) có thể đi vào hợp pháp theo cấu trúc đường và hướng giao
thông. Đây là **functional drivable area**, không phải toàn bộ vùng có màu asphalt.

Trong scope:

- Hành lang xe ego đang đi hoặc tiếp tục đi thẳng: `areaType=direct`.
- Làn cùng chiều hoặc nhánh rẽ mà ego có thể tiếp cận hợp pháp bằng chuyển làn/rẽ: `areaType=alternative`.
- Phần vạch qua đường nằm trên mặt đường mà xe được phép đi qua.

Ngoài scope, không vẽ:

- Vỉa hè, curb, dải phân cách, đảo giao thông, bãi cỏ.
- Vùng kẻ chéo/gore, vai đường và làn khẩn cấp nếu không có dấu hiệu cho phép lưu thông bình thường.
- Làn ngược chiều, chỗ đỗ xe, bãi đỗ và lối vào tư nhân nếu không có bằng chứng là làn lưu thông hợp pháp.
- Nhà, cây, bầu trời, phương tiện và mọi vùng không phải bề mặt giao thông.

Chỉ dùng label `drivable_area`; vùng ngoài scope để trống, không tạo label `not_drivable`.

## 2. Annotation unit

- Geometry: `Polygon` dạng Shape trên ảnh tĩnh.
- Một polygon biểu diễn một vùng liên thông có cùng `areaType`.
- Các làn kề nhau có cùng `areaType` được gộp thành một polygon; không tách chỉ vì vạch đứt giữa làn.
- Phải tách polygon khi `areaType` khác nhau hoặc khi hai vùng bị chia cách bởi curb, median, island, gore hay vùng
  ngoài scope.
- Hai vùng `alternative` nằm ở hai phía của vùng `direct` là hai polygon khác nhau vì chúng không liên thông với
  nhau nếu không đi qua vùng `direct`.
- Xe và người đi bộ là vật cản tạm thời, không tự động làm phát sinh polygon mới.

## 3. Geometry rule

Ranh giới polygon đi theo:

- Mép curb, median, island, guardrail hoặc mép mặt đường.
- Vạch liền, vùng kẻ chéo/gore và ranh giới cấm lưu thông.
- Hướng và cấu trúc của các làn xe.
- Vùng gần ego bắt đầu ở đáy ảnh; vùng xa chỉ kéo tới nơi còn đủ bằng chứng.

Không dùng màu asphalt, vị trí xe khác hoặc vết bánh xe làm bằng chứng duy nhất.

Xử lý vật cản tạm thời:

- Nếu hai phía của vật cản cho thấy rõ cùng mặt đường và ranh giới chỉ bị che một đoạn ngắn, giữ một polygon liên
  tục qua phần bị che và chọn `visibility=partially_occluded`.
- Không tách một vùng thành nhiều polygon chỉ vì ô tô hoặc người đi bộ che giữa vùng.
- Nếu không thể suy ra ranh giới một cách chắc chắn, chỉ vẽ phần có bằng chứng; chọn `visibility=unknown` và `needs_review=true`.

Polygon không được lấn rõ ràng sang sidewalk, island, gore, làn ngược chiều hoặc vùng đỗ xe ngoài scope.

## 4. Taxonomy

| Thành phần | Kiểu | Giá trị | Ý nghĩa |
|---|---|---|---|
| `drivable_area` | polygon label | — | Vùng mặt đường ego có thể đi vào hợp pháp |
| `areaType` | select | `direct`, `alternative` | Quan hệ của vùng với hành lang hiện tại của ego |
| `visibility` | select | `visible`, `partially_occluded`, `unknown` | Mức độ quan sát ranh giới và bề mặt |
| `needs_review` | checkbox | `false`, `true` | Đánh dấu polygon cần người khác xem lại |

### `areaType`

- `direct`: vùng đi tiếp tự nhiên từ vị trí ego mà không cần chuyển làn hoặc rẽ sang nhánh khác.
- `alternative`: vùng ego có thể tiếp cận hợp pháp bằng chuyển làn hoặc rẽ nhưng không phải hành lang hiện tại.

Không dùng `alternative` cho làn ngược chiều, vùng đỗ xe, vai đường, vùng kẻ chéo hoặc nhánh bị rào chắn. Nếu vùng `direct` và `alternative` tiếp xúc nhau, vẫn vẽ riêng vì attribute khác nhau.

### `visibility`

- `visible`: bề mặt và ranh giới cần thiết nhìn đủ rõ.
- `partially_occluded`: một phần bị che nhưng có thể nội suy ngắn và chắc chắn từ bằng chứng hai phía.
- `unknown`: thiếu bằng chứng để xác định ít nhất một ranh giới quan trọng.

Mỗi polygon phải được kiểm tra lại ba attribute; không giữ mặc định `direct`, `visible`, `false` chỉ vì quên chọn.

## 5. Inclusion / exclusion

Chỉ gán `drivable_area` khi có đủ ba điều kiện:

1. Đây là mặt đường dành cho xe lưu thông.
2. Hướng giao thông cho phép ego tiếp cận vùng đó.
3. Có đủ bằng chứng để xác định ranh giới polygon.

Quy tắc cụ thể:

- Vạch qua đường vẫn thuộc drivable area nếu nằm trên hành lang xe được phép đi qua.
- Làn cùng chiều là `alternative` nếu ego có thể chuyển làn hợp pháp.
- Nhánh rẽ là `alternative` nếu có đường nối hợp pháp và không bị island/gore/rào chắn ngăn cách.
- Xe đang chạy hoặc dừng tạm thời không đổi semantic của mặt đường bên dưới.
- Vùng sát curb có xe đỗ không phải `alternative` nếu không có bằng chứng đây là làn lưu thông.
- Asphalt nằm ngoài hàng rào, curb hoặc median không được gán dù có màu giống mặt đường.

## 6. Visibility / occlusion

- Vật cản ngắn và ranh giới hai phía rõ: nối vùng semantic qua vật cản, chọn `partially_occluded`.
- Ảnh tối, tuyết, phản sáng hoặc vật cản lớn làm mất ranh giới: không mở rộng theo phỏng đoán; dùng `unknown` và  `needs_review=true` cho phần bảo thủ còn lại.
- Không chọn `partially_occluded` chỉ vì ảnh có xe; chỉ chọn khi xe thực sự che vùng hoặc ranh giới đang gán.
- Nếu toàn bộ vùng không đủ bằng chứng để xác định là drivable, không vẽ polygon cho vùng đó.

## 7. Ambiguity / escalation

- **LABEL:** đủ bằng chứng về mặt đường, hướng lưu thông và ranh giới; vẽ polygon và chọn attribute.
- **IGNORE:** vùng rõ ràng ngoài scope; không vẽ.
- **UNKNOWN:** vùng có khả năng là đường nhưng thiếu bằng chứng về ranh giới; vẽ phần bảo thủ, đặt
  `visibility=unknown`, `needs_review=true`.
- **ESCALATE:** giao lộ phức tạp, merge/split không rõ, ảnh tối hoặc bị che lớn làm thay đổi quyết định  `direct/alternative`; thể hiện bằng `needs_review=true`.

Không suy đoán `alternative` chỉ vì vùng đó có màu asphalt.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh. Dùng Shape, không dùng Track.

## 9. Calibration decisions và examples

Các dòng dưới là quyết định chuẩn hóa sau khi so 4 annotator. Ảnh blind không xuất hiện ở đây.

| sample_id | Expected output sau calibration | Quy tắc được kiểm tra |
|---|---|---|
| `BDD03` | Một vùng `direct`; loại vai đường ngoài vạch biên | Ví dụ normal về scope và boundary |
| `BDD01` | `direct` cho hành lang hiện tại; loại vùng kẻ chéo; nhánh hợp pháp là `alternative` riêng | Ví dụ edge về gore |
| `BDD12` | Vạch qua đường nằm trong hành lang xe vẫn thuộc drivable; sidewalk không gán | Ví dụ đô thị |
| `BDD05` | **1 polygon `direct`**, `partially_occluded` nếu phải nội suy qua phần che | Không tách do occlusion |
| `BDD11` | **3 polygon:** 1 `direct` và 2 `alternative` cho hai nhánh rẽ hợp pháp | Tách theo areaType và vùng rời nhau |
| `BDD17` | **3 polygon:** 1 `direct` và 2 `alternative`; không tách thêm theo từng làn/mảng nhìn thấy | Annotation unit |
| `BDD20` | **1 polygon `direct`**; vùng có xe đỗ sát lề không phải alternative | Parking exclusion |
| `BDD22` | **2 polygon:** 1 `direct` và 1 `alternative` cho làn cùng chiều có thể chuyển sang | Direct/alternative |
| `BDD26` | **3 polygon:** 1 `direct` và 2 `alternative`; dùng `partially_occluded` hoặc escalation ở ranh giới thiếu sáng | Low visibility |

## 10. Common mistakes

| Lỗi | Cách tránh |
|---|---|
| Vẽ một polygon cho mỗi làn dù cùng `areaType` | Gộp các làn liên thông có cùng ý nghĩa vận hành |
| Gộp `direct` và `alternative` | Tách polygon vì attribute khác nhau |
| Tách polygon chỉ vì xe che | Giữ cùng vùng semantic nếu có thể nội suy ngắn và chắc chắn |
| Vẽ xuyên qua vùng che quá lớn | Dùng `unknown`, `needs_review=true` và chỉ giữ phần có bằng chứng |
| Gán parking, vai đường hoặc gore là `alternative` | Chỉ gán khi có bằng chứng là làn lưu thông hợp pháp |
| Giữ attribute mặc định mà không kiểm tra | Xem lại `areaType`, `visibility`, `needs_review` cho từng polygon |
| Tô toàn bộ asphalt | Áp dụng scope chức năng và hướng giao thông |

## Revision note — v2

V2 được viết lại sau khi so 6 ảnh từ 4 annotator: `leductu`, `nguyenhoangtung`, `phamanhhuy`, `vutungdinh`. Kết quả đồng thuận count và attribute/tag đều là 0%. Bản này chốt số polygon mong đợi cho từng ảnh calibration, làm rõ annotation unit, `direct/alternative`, occlusion và escalation.