# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_area` | polygon | class | `drivable_area` | - | false | Vùng mặt đường xe cơ giới có thể di chuyển hợp pháp (visible surface). |
| `areaType` | - | attribute (select) | `direct`, `alternative` | `direct` | false | Phân biệt làn xe ego đang chạy (`direct`) và làn cùng chiều liền kề có thể chuyển sang (`alternative`). |
| `visibility` | - | attribute (select) | `visible`, `partially_occluded`, `unknown` | `visible` | false | Trạng thái hiển thị của mặt đường khi bị che khuất bởi phương tiện/vật cản hoặc thời tiết xấu. |
| `needs_review` | - | attribute (checkbox) | `true`, `false` | `false` | false | Đánh dấu các trường hợp mơ hồ (ambiguous) hoặc bất thường cần escalate lên team lead / chuyên gia. |

## Class hay attribute

- **Vì sao `drivable_area` là Class:** Đây là thực thể đối tượng chính cần phân đoạn ranh giới hình học (polygon) cho bài toán Semantic/Instance Segmentation của xe tự hành.
- **Vì sao `areaType`, `visibility`, `needs_review` là Attribute:** 
  - `areaType` là thuộc tính phân loại ngữ cảnh di chuyển trên cùng một vùng mặt đường, tránh làm nổ số lượng class độc lập và giúp model học đặc trưng mặt đường thống nhất.
  - `visibility` mô tả mức độ nhìn rõ bề mặt đường trong điều kiện thực tế (bị xe khác đè lên mép hoặc thời tiết mờ ảo).
  - `needs_review` là cờ siêu dữ liệu phục vụ quy trình kiểm định chất lượng (QA) và escalation.
- **Default nào có thể gây bias khi annotator quên đổi?**
  - Default `areaType = direct`: Đa phần các làn đường bên cạnh là `alternative`. Nếu annotator vẽ làn liền kề mà quên đổi dropdown, nhãn sẽ bị gán nhầm thành `direct` (làn ego). Đây là lỗi Critical nghiêm trọng ảnh hưởng đến hệ thống điều khiển tự động. Cần nhấn mạnh kiểm tra thuộc tính này trong QA checklist.
  - Default `visibility = visible`: Annotator có xu hướng giữ nguyên mặc định dù mặt đường bị xe phía trước che khuất một phần.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): 3.6
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `HelloWorld-calib-v1`
- **Guide của task đã dán `02_guideline.md`?** Có
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape** vì tập dữ liệu BDD100K bao gồm các ảnh tĩnh đơn lẻ (không phải chuỗi video liên tục). Mỗi đa giác mặt đường là một đối tượng độc lập trong từng khung hình, không yêu cầu liên kết track ID xuyên suốt các frame.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào escalate. Ghi lại ai test và chỗ họ vấp:

- **Người thực hiện test:** Vũ Trung Định (@vutrungdinh0103-sketch - QA owner), kiểm tra task do Phạm Anh Huy (@PhAnhHuy - CVAT owner) tạo.
- **Kết quả trả lời:**
  - *Label:* `drivable_area`.
  - *Tool:* Công cụ Polygon (`N`), vẽ khép kín theo viền mặt đường nhìn thấy (`visible surface`), bám chân bánh xe/vật cản.
  - *Attribute:* Chọn `areaType` (`direct` cho làn ego, `alternative` cho làn phụ kề bên); chọn `visibility` tương ứng (`partially_occluded` nếu bị che); tích `needs_review` nếu có nghi ngờ.
  - *Escalate:* Khi gặp vạch kẻ mâu thuẫn, công trường không rõ lối đi, hoặc ranh giới lề/vỉa hè không thể phân định thì tích `needs_review = true` kèm ghi chú comment trên shape.
- **Chỗ vấp / Rủi ro phát hiện được:** Dễ quên đổi giá trị `areaType` từ `direct` sang `alternative` khi vẽ các làn bên cạnh do giá trị mặc định được gán sẵn là `direct`. Đã thống nhất bổ sung nhắc nhở vào mục Common Mistakes của guideline và kiểm tra 100% trong checklist QA.
