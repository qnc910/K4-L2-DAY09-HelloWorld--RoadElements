# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** Một thành viên khác người annotation chính review 100% sample calibration và 20% sample production chọn theo risk tag
- **Chọn sample theo rule nào** (random, theo tag rủi ro, theo annotator mới…): Ưu tiên edge case, ambiguity, occlusion, intersection
- **Issue được ghi ở đâu, đóng thế nào:** Ghi trong QA report, đóng bằng sửa annotation hoặc cập nhật guideline.
- **Khi phát hiện guideline gap thì update và version ra sao:** Tạo revision mới trong `08_revision_log.md`.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Sai vùng xe có thể đi | Include sidewalk thành drivable | Rework + update rule |
| Major | Sai areaType hoặc visibility | direct thành alternative | Rework |
| Minor | Boundary lệch nhỏ | Polygon lệch mép đường | Fix annotation |
| Question | Chưa rõ rule | Case cần review | Escalate |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Polygon accuracy | Review pass / total polygon | Đánh giá chất lượng hình học |
| Attribute accuracy | Attribute đúng / total attribute | Kiểm tra semantic |
| Critical Defect Rate | Critical lỗi / total sample | Đánh giá rủi ro |

Metric high-risk tách riêng (ví dụ critical defect escape rate): Critical defect escape rate

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```Quality gate
PASS if:
  - Không có critical defect (Critical Defect Rate = 0).
  - Attribute accuracy >= 95%.
  - Polygon accuracy >= 90%.
  - Toàn bộ QA issue đã được sửa và đóng.
REWORK if:
  - Có >= 1 critical defect.
  - Attribute accuracy < 95% hoặc Polygon accuracy < 90%.
  - Phát hiện guideline gap nhưng chưa cập nhật vào tài liệu và 08_revision_log.md.
REJECT / ESCALATE if:
  - Không xác định được rule hoặc có xung đột ontology không thể phân giải.
  - Tỷ lệ lỗi trên mẫu kiểm tra vượt quá 30%, yêu cầu đào tạo lại annotator và gán nhãn lại.
```

Trade-off:
- **Zero-tolerance đối với Critical defect (Ưu tiên an toàn tính mạng):** Vì downstream model phục vụ hệ thống Trajectory Planning và điều khiển lái tự động (ADAS Level 2+), lỗi nhận nhầm vỉa hè, dải phân cách hay làn ngược chiều thành vùng xe chạy được (Critical False Positive) tiềm ẩn nguy cơ tai nạn nghiêm trọng. Vì vậy nhóm chấp nhận chi phí thẩm định cao (review 100% calibration và review theo risk tag cho production) để triệt tiêu hoàn toàn rủi ro này.
- **Dung sai hình học thực dụng (Tối ưu chi phí và tốc độ):** Thiết lập ngưỡng Polygon accuracy >= 90% với dung sai viền <= 3 px cho các cạnh quan sát rõ. Nhóm không yêu cầu annotator soi zoom cực đại chỉnh từng pixel ở vùng quá xa (> 80m), bóng râm mờ hoặc trời mưa tuyết. Quyết định này giúp tiết kiệm khoảng 35-40% thời gian gán nhãn mà vẫn đảm bảo độ tin cậy vận hành cho downstream planner.
