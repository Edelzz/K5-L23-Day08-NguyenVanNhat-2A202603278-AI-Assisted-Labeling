# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603278 - Nguyễn Văn Nhất

Chỉ bổ sung quy trình so với v1.0; không đổi taxonomy hoặc ngưỡng QC. Các ca tham chiếu nằm trong `../qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| Câu hỏi v1.0 | Quyết định | Ảnh ví dụ | Case |
|---|---|---|---|
| Mục 6, câu 7: điểm tin cậy tối thiểu để chấp nhận box không cần sửa | Không có ngưỡng confidence thay cho review. Điểm chỉ hỗ trợ ưu tiên kiểm tra. Chỉ giữ nguyên sau khi kiểm tra đúng vật, đúng lớp và box ôm phần nhìn thấy. | img_002.jpg, img_003.jpg | CAL-03 |

Hai ảnh trên minh họa lỗi cần kiểm tra, không phải bằng chứng rằng lỗi có confidence cao. Chưa ghi nhận điểm cụ thể của từng box.

Các câu hỏi mở 1–6 chưa chốt trong phiên bản này vì chưa có đủ ca đối chiếu và quyết định cụ thể. Khi gặp ca chưa rõ phải ghi can_review và xem xét trước release.

## 2. Bổ sung checklist từ lỗi máy và bẫy lọt

| Case | Ảnh và tình huống | Cách xử lý | Phạm vi thay đổi |
|---|---|---|---|
| CAL-01 | img_002.jpg: YOLO26n gọi xe ba bánh là truck | Kiểm tra hình dáng và số bánh nhìn thấy; áp dụng three_wheeler theo v1.0; nếu không chắc thì can_review. | Nhắc kiểm tra car/truck nghi là xe ba bánh; không đổi định nghĩa lớp. |
| CAL-02 | img_003.jpg: box person bị lệch | Kiểm tra trên, dưới, trái, phải so với phần người nhìn thấy; chỉnh cạnh sai, không đoán phần bị che. | Bổ sung thao tác kiểm bốn cạnh; không đổi quy tắc hình học. |
| CAL-03 | img_002.jpg và img_003.jpg: lỗi lớp/hình học có thể tồn tại trong pre-label | Ẩn nhãn để quét ảnh; hiện lại và kiểm đủ bỏ sót, thừa, lớp, hình học, bất kể confidence. | Làm rõ cách dùng conf ở mục 3 v1.0. |

Ảnh ví dụ: [img_002.jpg](../../data/images/img_002.jpg), [img_003.jpg](../../data/images/img_003.jpg).

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Cơ sở |
|---|---|---|
| v1.0 | Guideline gốc, giữ nguyên | Bản giảng viên |
| v1.1 | Chốt câu hỏi 7 về confidence; bổ sung checklist kiểm lớp xe ba bánh và bốn cạnh box | CAL-01, CAL-02, CAL-03 |

Các bổ sung trên không tự sửa nhãn. Box lệch trong img_003.jpg và các box dưới 20 px vẫn cần kiểm tra trực quan trước khi chốt release.
