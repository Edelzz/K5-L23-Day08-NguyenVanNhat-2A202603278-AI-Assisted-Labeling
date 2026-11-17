# Báo cáo Day 08 — 2A202603278 - Nguyễn Văn Nhất

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n | <<theo lớp, từ threshold_sweep>> | Đã thử 0.25 và 0.5 trên pilot ở r01–r02; chưa chốt ngưỡng cuối cùng |
| three_wheeler | YOLOE-26s; lớp yêu cầu: three_wheeler | <<từ threshold_sweep>> | r03 tìm được xe ba bánh nhưng còn bỏ sót xe bị che và nhầm xe bán tải; cần review thủ công |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM đã thử:** SAM 2.1 base (`sam2.1_b.pt`) trong `r04`, nhận box từ YOLO26n. Theo quan sát, mask bám biên person và tách nền tốt nhưng có trường hợp thiếu phần chân. Dùng làm mask ban đầu; cần kiểm tra box và phần chân nhìn thấy trước khi chỉnh mask hoặc thêm click. Lần chạy tạo 8 mask từ 8 box trên 2 ảnh, trung bình 4.8 giây/ảnh trên CPU (nguồn: `work/runs.csv`, `r04`).
- **Lỗi không sửa được chỉ bằng đổi ngưỡng:** ở `r01`, YOLO26n nhận nhầm `three_wheeler` thành `truck` theo quan sát của tôi. Đổi ngưỡng chỉ lọc dự đoán theo điểm tin cậy, không bổ sung lớp xe ba bánh cho detector danh mục cố định. Hướng xử lý là dùng YOLOE bổ sung three_wheeler và review thủ công để sửa lớp, loại box trùng hoặc sai. Ảnh minh họa cần bổ sung: `img_002.jpg`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- **Giới hạn đánh giá ngưỡng:** gold chỉ có 4 ảnh. Khi chọn và đánh giá ngưỡng trên cùng tập gold, precision/recall có thể lạc quan; không coi đó là kết quả trên tập kiểm thử độc lập.

### Quan sát từ tour model (Part 2)

| Run | Tập ảnh | Số box | Thời gian/ảnh trên CPU | Quan sát và quyết định |
|---|---|---:|---:|---|
| r01: YOLO26n, conf 0.25 | 8 ảnh pilot | 148 | 0.58 giây | Phát hiện đúng nhiều xe; nhầm three_wheeler thành truck và bỏ sót bicycle. Dùng làm pre-label ban đầu, kiểm tra lại lớp và vật bị sót. |
| r02: YOLO26n, conf 0.5 | 8 ảnh pilot | 81 | 0.33 giây | Vẫn bỏ sót three_wheeler và bicycle. Chưa chọn 0.5 làm ngưỡng chung; đánh giá theo lớp ở Part 3. |
| r03: YOLOE, conf 0.25 | 8 ảnh pilot | 8 | 0.34 giây | Tìm đúng một số three_wheeler; còn sót xe bị che và nhầm xe bán tải. Dùng bổ sung lớp xe ba bánh và review thủ công. |
| r04: YOLO26n + SAM 2.1 base | 2 ảnh: img_004.jpg, img_013.jpg | 8 | 4.8 giây | Tạo 8 mask; quan sát mask bám biên người nhưng thiếu phần chân. Kiểm tra phần chân có nhìn thấy trong ảnh gốc hay bị che. |

- Tăng ngưỡng từ 0.25 lên 0.5 làm số box giảm 67 (148 → 81 trên cùng 8 ảnh), nhưng chưa chứng minh chất lượng tăng. Cần đối chiếu các box bị loại là đúng hay sai.
- Mask tốt chưa bảo đảm nhãn lớp đúng. Cần kiểm tra cả lớp, box đầu vào và đường biên mask.
- Nguồn số liệu: `work/runs.csv`; nhận xét và quyết định: `submission/model_log.csv`. Các nhận xét là quan sát định tính; chưa có precision/recall hay recipe cuối cùng. Tour SAM này chưa thay thế yêu cầu tạo mask bằng click cho 3 vật ở Part 5.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** dùng Codex hỗ trợ đọc hướng dẫn, kiểm tra cấu trúc nhãn, định dạng CSV và diễn đạt báo cáo từ nhận xét của tôi. Nhãn gold do tôi gán tay; nhận xét model cần được tôi đối chiếu với ảnh kết quả.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng: Windows, PyTorch CPU (không dùng GPU), CVAT Docker v2.81.0.
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
