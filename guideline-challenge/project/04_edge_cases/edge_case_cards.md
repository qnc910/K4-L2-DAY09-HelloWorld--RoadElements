# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

## CASE ID: DA_EC_01

Sample: BDD_example_parking

Scene: Parking area cạnh đường.

Observation: Vùng asphalt rộng nhưng có dấu hiệu là khu vực đỗ xe.

Decision: IGNORE

Expected: - Không tạo drivable_area polygon.

Rationale: Không phải mọi vùng asphalt đều là functional driving area.

Common mistake: Tô toàn bộ vùng mặt đường.

Diversity: ambiguity

------------------------------------------------------------------------

## CASE ID: DA_EC_02

Sample: BDD_example_sidewalk

Scene: Sidewalk cùng màu với đường.

Observation: Vùng cạnh đường có màu gần giống lane xe.

Decision: IGNORE

Expected: - Không label.

Rationale: Xác định theo chức năng giao thông, không theo màu.

Common mistake: Include sidewalk.

Diversity: conflict

------------------------------------------------------------------------

## CASE ID: DA_EC_03

Sample: BDD_example_occlusion

Scene: Xe phía trước che một phần lane.

Observation: Boundary phía sau xe không nhìn thấy.

Decision: LABEL

Expected: drivable_area: - areaType=direct -
visibility=partially_occluded

Rationale: Có bằng chứng vùng xe đang đi nhưng bị che một phần.

Common mistake: Vẽ polygon xuyên qua xe.

Diversity: occlusion

------------------------------------------------------------------------

## CASE ID: DA_EC_04

Sample: BDD_example_intersection

Scene: Giao lộ nhiều hướng.

Observation: Có nhiều vùng asphalt giao nhau.

Decision: ESCALATE

Expected: needs_review=true

Rationale: Cần xác định hướng di chuyển và right-of-way.

Common mistake: Gộp toàn bộ giao lộ.

Diversity: critical, escalation

------------------------------------------------------------------------

## CASE ID: DA_EC_05

Sample: BDD_example_merge

Scene: Làn nhập.

Observation: Lane có thể tiếp cận nhưng không phải đường hiện tại.

Decision: LABEL

Expected: drivable_area: - areaType=alternative

Rationale: Có khả năng sử dụng nhưng không phải current path.

Common mistake: Gán direct.

Diversity: ambiguity

------------------------------------------------------------------------

## CASE ID: DA_EC_06

Sample: BDD_example_median

Scene: Median/island giữa đường.

Observation: Có vùng asphalt nhỏ nằm giữa các lane.

Decision: IGNORE

Expected: Không label.

Rationale: Không phục vụ xe di chuyển.

Common mistake: Include vì nhìn giống đường.

Diversity: conflict

------------------------------------------------------------------------

## CASE ID: DA_EC_07

Sample: BDD_example_unknown

Scene: Boundary bị che gần như hoàn toàn.

Observation: Không đủ thông tin.

Decision: UNKNOWN

Expected: visibility=unknown

Rationale: Không suy đoán khi thiếu bằng chứng.

Common mistake: Extrapolate.

Diversity: escalation

------------------------------------------------------------------------

## CASE ID: DA_EC_08

Sample: BDD_example_disconnected

Scene: Hai vùng drivable tách biệt.

Observation: Có nhiều vùng hợp lệ không liên tục.

Decision: LABEL

Expected: Nhiều polygon drivable_area riêng.

Rationale: Không nối các vùng không liên tục.

Common mistake: Tạo polygon self-intersecting.

Diversity: critical

---
