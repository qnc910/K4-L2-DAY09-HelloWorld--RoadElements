# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Tạo guideline drivable_area, areaType direct/alternative | Khởi tạo ontology |---|
|v2| Bổ sung rule occlusion, intersection, parking, merge | Calibration phát hiện ambiguity | Calibration report |
|v3| Chốt annotation unit và số polygon cho 6 ảnh; làm rõ `direct/alternative`, parking, occlusion và escalation | So 6 ảnh từ 4 annotator cho kết quả 0% đồng thuận count và 0% đồng thuận attribute/tag | `06_calibration_measure.csv`, `06_calibration_report.csv`, BDD05, BDD11, BDD17, BDD20, BDD22, BDD26 |