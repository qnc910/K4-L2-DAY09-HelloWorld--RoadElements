# Problem statement + downstream contract

Tối đa nửa trang, viết **trước khi mở CVAT**. Đây là bằng chứng của gate G1 (topic lock). Thay mọi placeholder
mới là xong.

## Bài toán

Phân đoạn đa giác (polygon segmentation) vùng mặt đường xe có thể chạy (**Drivable Area**) trên ảnh camera trước (BDD100K), tập trung giải quyết ranh giới phức tạp giữa làn xe ego đang chạy (**direct drivable**), làn đường có thể chuyển sang (**alternative drivable**), và các vùng dễ nhầm lẫn như vỉa hè, lề đường chưa lát, khu vực đỗ xe ven đường, bề mặt tuyết/ngập nước hoặc bị xe cộ che khuất.

## Downstream contract

1. **Downstream task / model / user là ai?**
   Module Lập kế hoạch quỹ đạo (Trajectory / Motion Planning) và model Semantic/Instance Segmentation (như Mask2Former / SegFormer) của hệ thống xe tự hành (Autonomous Driving Level 2+ / ADAS).
2. **Output annotation nào thực sự cần?**
   - **Geometry:** `Polygon` khép kín (visible surface — chỉ vẽ vùng mặt đường nhìn thấy được, bám chân vật cản).
   - **Class:**
     - `drivable_area`: đoạn đường trên ảnh 
   - **Attribute:**
     - `areaType`: `direct` | `alternative` (loại làn xe có thể đi thẳng ngay hay làn liền kề để chuyển sang).
     - `visibility`: `visible` | `partially_occluded` | `unknown` (mức độ nhìn rõ mặt đường).
     - `needs_review`: checkbox `true` | `false` (ảnh/vùng có cần review lại không).
3. **Failure nào gây hậu quả lớn nhất?**
   - **False Positive (gán nhầm vùng không thể lái thành drivable):** Gán nhầm vỉa hè, dải phân cách cứng, vực lề đường, làn ngược chiều hoặc khu vực công trường rào chắn thành drivable area. Lỗi này là nguy cơ an toàn tính mạng cấp độ **Critical** vì khiến xe lập quỹ đạo lao lên vỉa hè hoặc đâm vào chướng ngại vật/xe ngược chiều.
   - **False Negative trên ego lane:** Bỏ sót hoặc cắt đứt làn đường xe đang chạy khiến xe phanh khẩn cấp không lý do (phantom braking) gây tai nạn dồn toa.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   Annotator đánh dấu attribute `needs_review = true` (có thể kết hợp gán `areaType = undefined` nếu không rõ làn) kèm comment mô tả vị trí và lý do phân vân trên shape CVAT. Sau đó thông báo sample ID lên kênh thảo luận nhóm và escalate trực tiếp cho Spec Owner (@leductu204) và QA Owner (@vutrungdinh0103-sketch) để thống nhất quyết định và cập nhật vào `04_edge_cases/edge_case_cards.md` trong vòng 15 phút.

## Scope

- **Trong scope (bắt buộc label):**
  - Tất cả các vùng mặt đường nhựa/bê tông bằng phẳng mà xe cơ giới được phép lưu thông hợp pháp (làn xe ego đang chạy và các làn hợp lệ cùng chiều liền kề).
  - Vùng giao lộ mở (intersection) mà xe được phép đi qua hoặc rẽ vào.
  - Bề mặt đường nhìn thấy thực tế (visible surface convention) tới chân bánh xe hoặc chân chướng ngại vật (không vẽ xuyên thấu qua thân xe).
- **Ngoài scope (ignore):**
  - Vỉa hè (sidewalk), dải phân cách cứng, bồn cây, đảo giao thông, rãnh thoát nước, lề đất/cỏ không dành cho xe chạy.
  - Làn đường ngược chiều có dải phân cách hoặc vạch liền đôi phân cách rõ ràng.
  - Chướng ngại vật tĩnh/động đè lên đường (thân xe, người đi bộ, rào chắn công trường) — cắt viền đa giác quanh chân vật cản (no amodal).
  - Vùng mặt đường quá xa (chiều cao phối cảnh < 15 pixel hoặc khoảng cách ước lượng > 80m nơi không phân định được ranh giới rõ ràng).
- **Geometry tolerance:**
  - Polygon bám sát viền phân cách (vạch kẻ đường, mép curb, chân bánh xe) với độ lệch biên (boundary deviation) ≤ 3 pixel đối với các ranh giới rõ ràng.
  - Mật độ điểm hợp lý: khoảng 10–20 pixel một điểm ở đoạn cong, góc bo; không đặt điểm thừa trên đoạn thẳng dài; không để hở khoảng trống giữa các polygon kề nhau.

## Output chấm được

Mọi quyết định trong blind test đều được phản ánh tường minh trong file export CVAT:
- **LABEL**: Tạo polygon với class `drivable_area`, chọn `areaType` là `direct` (làn xe ego) hoặc `alternative` (làn cùng chiều có thể chuyển sang).
- **IGNORE**: Không vẽ polygon ở các vùng ngoài scope (vỉa hè, làn ngược chiều, chướng ngại vật nổi, vùng quá xa).
- **UNKNOWN**: Tạo polygon `drivable_area` với `visibility = unknown` khi mặt đường bị che phủ (tuyết dày, ngập nước lóa sáng) không đủ bằng chứng để xác định rõ ràng.
- **ESCALATE**: Bật cờ `needs_review = true` trên shape (kèm comment trên CVAT) khi có tình huống xung đột vạch kẻ, công trường bất thường mà annotator không tự quyết định được theo guideline.

## Dữ liệu và giới hạn

- **Nguồn ảnh:** Tập dữ liệu BDD100K tại thư mục `data/bdd100k/` gồm 26 ảnh tĩnh (BDD01 – BDD26), kích thước chuẩn 1280x720.
- **Số ảnh dự kiến dùng:** 16 ảnh (chia thành: 4 ảnh `example`, 7 ảnh `calibration`, 5 ảnh `blind`).
- **Giới hạn đã biết:** Dữ liệu là các frame ảnh đơn tĩnh (không có chuỗi temporal liên tục để theo dõi hướng di chuyển của xe khác); có ảnh ban đêm (BDD18, BDD26) và thời tiết tuyết/mưa (BDD17, BDD23, BDD24) làm giảm độ tương phản giữa mép đường và lề/vỉa hè.
