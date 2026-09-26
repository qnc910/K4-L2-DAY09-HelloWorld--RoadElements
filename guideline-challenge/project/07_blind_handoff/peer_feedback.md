# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới là xong (gate G5).

- **Nhóm peer:** Hoa Thanh Que
- **Người label blind:** Phan Tấn Đạt

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất?
Quy tắc phân biệt `areaType`: phân biệt giữa làn trực tiếp (`direct`) của ego vehicle và các làn cùng chiều còn lại (`alternative`). Quy tắc loại trừ vỉa hè (`sidewalk`) và dải phân cách bê tông (`barrier/median`) được định nghĩa rất cụ thể.

2. Rule nào mơ hồ hoặc phải tự suy diễn?
Xử lý vùng đỗ xe sát lề ở khu dân cư khi không có vạch phân làn đỗ rõ ràng (như ở sample BDD23), ban đầu phải cân nhắc xem có nên kéo dài polygon ra đến curb hay chỉ giới hạn giữa 2 hàng xe.

3. Sample nào khiến guideline "vỡ"?
Sample BDD24 (tuyết phủ dày che khuất mép đường phải và có xe lớn đỗ bên cạnh). Tuy nhiên nhờ có rule Escalation (`needs_review=true`, `visibility=unknown`) trong guideline v2 nên đã xử lý được an toàn.

4. Attribute / default nào trong CVAT dễ gây thao tác sai?
Default value của `visibility` là `visible`, nếu không chú ý ở các case mưa/tuyết thì dễ quên chuyển sang `unknown` hoặc `partially_occluded`.

5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?
Bổ sung thêm hình ảnh worked-example minh họa trực quan cho trường hợp tuyết phủ mép đường và khu vực có xe đỗ hai bên đường hẹp.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Băn khoăn ranh giới vùng đỗ xe dân cư (BDD23) | guideline_gap | accept + revise: làm rõ thêm rule parking exclusion ở mục 5 guideline v3 | BDD23, feedback câu 2 |
| Dễ quên set `visibility=unknown` khi tuyết che (BDD24) | execution_error | coaching: nhấn mạnh checklist self-QC khi gặp điều kiện low visibility | BDD24, feedback câu 4 |
| Đề xuất thêm ví dụ ảnh trực quan cho tuyết và xe đỗ | guideline_gap | accept + revise: bổ sung card ví dụ vào library và mục worked-example | Feedback câu 5, DA_EC_07 |
