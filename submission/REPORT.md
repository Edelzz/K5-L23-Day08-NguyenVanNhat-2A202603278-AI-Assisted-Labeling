# Báo cáo Day 08 — 2A202603278 - Nguyễn Văn Nhất

Ngày cập nhật: 2026-11-18. Báo cáo của bản nhãn hiện tại: **24 ảnh batch, 474 annotations**. Kết luận **Rework**; không phát hành bộ nhãn như một bộ đạt QC.

## 1. Model và recipe (Part 2–3)

Tôi gán tay 4 ảnh gold trước khi chạy tour model. Dùng Python 3.12.10, PyTorch CPU, YOLO26n, YOLOE-26s và SAM 2.1 base. Chọn YOLO26n có sẵn để chạy trên CPU.

| Run | Cấu hình | Số ảnh | Kết quả | Giây/ảnh |
|---|---|---:|---|---:|
| r01 | YOLO26n, conf 0.25 | 8 pilot | 148 box | 0.58 |
| r02 | YOLO26n, conf 0.5 | 8 pilot | 81 box | 0.33 |
| r03 | YOLOE-26s, conf 0.25 | 8 pilot | 8 box three_wheeler | 0.34 |
| r04 | YOLO26n + SAM 2.1 base, conf 0.3 | 2 | 8 box, 8 mask | 4.80 |

Nguồn: `model_log.csv`, `work/runs.csv` và log terminal. Thời gian trong bảng không gồm nạp model. YOLO26n nhận nhầm xe ba bánh thành truck ở **img_002.jpg**. Tăng ngưỡng làm giảm 67/148 box (45.3%) nhưng chưa chứng minh tốt hơn; bicycle vẫn được phát hiện, số box giảm từ 6 xuống 1, không phải không phát hiện được xe đạp nào.

YOLOE dùng prompt tuk-tuk, cyclo, cargo tricycle ánh xạ về three_wheeler; tìm được xe ba bánh nhưng còn bỏ sót xe bị che và nhầm bán tải. Chỉ đổi confidence không bổ sung được lớp xe ba bánh cho detector danh mục cố định. Cần kết hợp YOLOE và review lớp/box trùng.

Ở conf 0.05 trên toàn bộ 36 ảnh: YOLO26n tạo 1,525 box (0.20 giây/ảnh), YOLOE tạo 125 box (0.40 giây/ảnh). Đây là số dự đoán, không phải số vật đúng.

### Ngưỡng theo lớp

Nguồn: `qc/threshold_sweep.csv`, `qc/threshold_sweep.md`; mục tiêu recall 90%, quét trên 4 ảnh gold.

| Lớp | Model | Ngưỡng dùng | Cơ sở |
|---|---|---:|---|
| person | YOLO26n | 0.3 mặc định | Recall tối đa khoảng 53% trên 43 vật; không đạt mục tiêu |
| bicycle | YOLO26n | 0.3 mặc định | Recall tối đa 20% trên 10 vật; không đạt mục tiêu |
| motorbike | YOLO26n | 0.3 mặc định | Recall tối đa khoảng 37% trên 27 vật; không đạt mục tiêu |
| car | YOLO26n | 0.1 | Recall khoảng 91%, precision khoảng 48%, 11 vật gold |
| bus | YOLO26n | 0.3 mặc định | Chỉ 1 vật, không đủ 5 mẫu để chọn ngưỡng riêng |
| truck | YOLO26n | 0.3 mặc định | Không có vật gold, không đủ cơ sở chọn ngưỡng |
| three_wheeler | YOLOE-26s | 0.15 | Recall 100%, precision khoảng 78%, 7 vật gold |

Recall tối đa của các lớp giữ mặc định không phải recall tại 0.3. Đã gộp với YOLOE ưu tiên trước YOLO26n:

```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Kết quả gộp trước cài bẫy: 429 box trên 28 ảnh batch + gold. Inject bẫy với MSHV 2A202603278; file nạp CVAT chỉ có 24 ảnh batch.

**Giới hạn:** chọn và chấm ngưỡng trên cùng 4 ảnh gold có thể lạc quan. Gold có 116 annotations nhưng còn 20 cảnh báo box dưới 20 px; cần rà trước lần phát hành tiếp theo. Chưa có ghi nhận điểm confidence cụ thể đủ để dẫn ví dụ “điểm cao mà sai/điểm thấp mà đúng”; không suy diễn điểm từ việc nhầm lớp.

## 2. AI error profile

Nguồn: `qc/review_stats.json` và `.md`, chạy trên bản hiện tại 474 annotations. Script loại 19 đối tượng quanh bẫy; mẫu số sau xử lý là 370 box máy, 455 đối tượng sau duyệt.

| Trạng thái | Số | Tỷ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 339 | 91.6% | 370 box máy |
| edited | 9 | 2.4% | 370 box máy |
| deleted | 16 | 4.3% | 370 box máy |
| added | 107 | 23.5% | 455 đối tượng sau duyệt |

Giữ số liệu đúng như script xuất; các nhóm không được suy diễn là một phân hoạch có cùng mẫu số. Trong deleted/added có 1 ca đổi lớp, số vật sót thật ước tính 107 − 1 = 106. Person có 53 added, motorbike có 48 added; cần ưu tiên tìm vật sót ở hai lớp này.

So với gold: TP=39, FP=11, FN=60; precision **39/50 = 78.0%** (KTC 95%: 65–87%), recall **39/99 = 39.4%** (KTC 95%: 30–49%). Mẫu số 99 là số vật gold theo xử lý của script, không phải toàn bộ 116 annotations. Không có gold sau review để tính residual/new error, vì gold không đưa vào Task batch.

## 3. Thiên kiến tự động hóa và ảnh bẫy

| Vòng | Kết quả | Cách diễn giải |
|---|---|---|
| Review đầu | 4/12 = 33.3% | Kết quả trước khi biết đáp án, KTC 95% khoảng 14–61% |
| Sau sửa lần hai | 7/12 = 58.3% | Đã biết vị trí bẫy |
| Bản hiện tại | 11/12 = 91.7% | Sau biết đáp án; không thay thế phép đo mù |

Nguồn kết quả đầu: `qc/review_stats_first_pass.md` và `.json`. Kết quả hiện tại: missing 3/3, wrong_class 3/3, loose 2/3, extra 3/3; còn **img_003.jpg, loose**. KTC 95% khoảng 65–99%.

Accepted lần đầu 96.5% nhưng bắt bẫy chỉ 33.3% cho thấy giữ nguyên nhiều không đủ chứng minh review tốt. Tôi chưa có nhật ký để khẳng định bước checklist nào bị bỏ ở từng ảnh. Cần kiểm đồng thời vật bị sót, lớp và độ ôm sát box.

Có một lượt AI sửa nhãn ngoài phạm vi đã được **khôi phục hoàn toàn**; kết quả 12/12 của lượt đó không áp dụng cho bản hiện tại. Việc xóa 9 box theo yêu cầu riêng trước đó vẫn được giữ.

## 4. Máy có đáng dùng không?

Nguồn `time_log.csv`: gán tay **15/4 = 3.75 phút/ảnh**; review đầu **30/24 = 1.25 phút/ảnh**. Review đầu nhanh hơn khoảng 3 lần theo trung bình hai tập, nhưng độ khó khác nhau và tôi đã quen cảnh khi gán gold.

**Chưa ghi thời gian rework**: để thiếu số đo và không bịa phút. Hai dòng log chỉ phản ánh thời gian ban đầu; chưa tính chi phí sửa, QC và báo cáo. Vì QC không đạt, chưa thể kết luận tiết kiệm thời gian ở mức chất lượng tương đương.

AI có ích để tạo đề xuất; cần review, đặc biệt xe ba bánh và vật bị sót. Ba quy tắc cho vòng tiếp theo:

1. Ẩn nhãn, tự quét đủ 24/24 ảnh trước khi đối chiếu máy.
2. Mỗi ảnh kiểm đủ bỏ sót, thừa, sai lớp và box lỏng; không miễn kiểm vì confidence cao.
3. Export trước khi xem đáp án; giữ số đo mù đầu tiên, ghi riêng thời gian và kết quả sửa sau biết đáp án.

## 5. Mask SAM cho 3 vật (Part 5)

Dùng **SAM 2.1 base, sam2.1_b.pt**, chạy CPU qua sam2_points.py. Học viên đã xác nhận cả ba ảnh đúng vật chọn lúc thử; QC bằng AI sau đó vẫn ghi nhận giới hạn, không xem xác nhận ban đầu là mask hoàn hảo.

| Ảnh | Prompt | Kết quả và quan sát | File trong masks/ |
|---|---|---|---|
| img_013.jpg | 1 dương (500,476), 0 âm | Có mái, thân và bánh xe; không thấy hàng hóa nhô ra rõ. Vẫn lấy một phần người lái và một số vùng qua khung; cần rework nếu yêu cầu mask xe loại người và khoảng trống lớn. | sam_img013_v1.jpg |
| img_004.jpg | 1 dương (680,430), 0 âm | Tách xe tuk-tuk phía trước bên phải; người đứng bên phải nằm ngoài mask; không có hàng hóa nhô ra rõ. Một ít vùng mép/khoảng trống cabin cần kiểm kỹ khi xuất mask chính xác. | sam_vat02.jpg |
| img_012.jpg | 2 dương (344,202), (305,100); box (235,55)–(418,272); 0 âm | Xe tuk-tuk phía trên trái; có mái và thân, các người đứng tách khỏi xe không vào mask; không có hàng hóa rõ. Có vùng tối bên trong cabin cần rà theo quy tắc lỗ lớn. | sam_vat03_img012.jpg |

Quan sát bằng Codex trên ảnh overlay, chưa có mask gold để định lượng IoU. Guideline 4.2 giúp phân biệt phần nhìn thấy, bóng đổ và khoảng trống. Định nghĩa lớp tách người khỏi xe; vì vậy phần người bị lấy vào mask phải được ghi là giới hạn. Không đổi ảnh mask trong vòng QC này. Chưa thử Interactor SAM trong CVAT; kết quả được tạo offline.

## 6. Calibration (Part 6)

Đã có **3 quyết định**, khớp `qc/calibration_decisions.csv` và phần bổ sung `guidelines/guideline_v1.1_calibrated.md`, kèm ảnh ví dụ:

| Case | Nguồn | Nội dung |
|---|---|---|
| CAL-01 | loi_may | img_002.jpg: three_wheeler thành truck; giữ định nghĩa lớp, bổ sung lượt kiểm xe ba bánh |
| CAL-02 | bay_lot | img_003.jpg: box person lệch; giữ quy tắc hình học, kiểm lần lượt bốn cạnh |
| CAL-03 | cau_hoi_mo | Chốt câu hỏi 7: confidence chỉ hỗ trợ ưu tiên; không thay thế kiểm lớp và hình học |

Tôi giao Codex hoàn thiện tài liệu calibration trong phạm vi QC/báo cáo. CAL-01 và CAL-02 là lỗi máy/áp dụng quy tắc, không phải lý do đổi định nghĩa lớp hoặc ngưỡng hình học. CAL-03 làm rõ cách dùng conf. Câu hỏi mở 1–6 chưa chốt vì chưa đủ ca cụ thể; không khẳng định đã giải quyết toàn bộ guideline. Các bổ sung là quy tắc cho vòng sau, không phải xác nhận đã sửa nhãn.

## 7. QC và quyết định (Part 6)

QC hiện tại do **Codex-AI kiểm trực quan** theo yêu cầu học viên, giữ nguyên nhãn. Nguồn: `qc/qc_sample.csv`, `qc/qc_review.csv`, `qc/qc_report_auto.md`. Mẫu seed 42: 5/24 ảnh, 0 ca can_review + 5 ngẫu nhiên, **91 đối tượng** không tính ignore_region.

| Ảnh | ID release hiện tại | Phát hiện QC |
|---|---|---|
| img_003.jpg | 4; 13 | Box person lệch xuống; box xe ba bánh thiếu mái gắn liền |
| img_006.jpg | 77 | Box motorbike gần trùng box 76 quanh xe trong hàng đỗ; cần kiểm riêng từng xe khi rework |
| img_014.jpg | 131 | Box motorbike cao 17.75 px, dưới ngưỡng 20 px, không nên gán riêng |
| img_031.jpg | 411 | Box bicycle cắt phần trên của giỏ/khung gắn cố định |
| img_035.jpg | 456 | Xe ba bánh bị chia thân và mái giữa box 456/457, thay vì một box đủ xe |

- Lỗi theo đối tượng được ghi: **6/91 = 6.6%**.
- Ảnh có lỗi: **5/5 = 100%**, vượt ngưỡng 2%.
- Bốn lỗi hình học và hai box thừa; hình học chiếm 4/6 lỗi, là lỗi lặp.
- Cận trên 95% Wilson theo script khoảng **100%**; mẫu nhỏ không chứng minh tỷ lệ ảnh lỗi thật dưới 2%.
- Đây là đánh giá bằng AI ở độ phân giải ảnh overlay; vùng xa/dày có thể còn lỗi chưa phát hiện. Nhận xét gần trùng cần xác nhận từng vật khi sửa, không tự xóa.
- Kết quả người kiểm ban đầu **9/100**, 3/5 ảnh lỗi là của bản trước xóa 9 box; không trộn với vòng hiện tại. ID đã thay đổi sau build release.
- Validate cấu trúc báo OK nhưng batch còn **13** cảnh báo box dưới 20 px, gold còn **20**. Script không chứng minh lớp/box đúng hoặc phát hiện vật cả người và máy cùng bỏ sót.

**Quyết định: Rework — 2026-11-18.** Bẫy sau biết đáp án đạt 11/12 nhưng QC còn lỗi hình học và box thừa; không đủ căn cứ Release. Đây là bộ release candidate để ghi nhận trạng thái bài, không phải bộ dữ liệu đã được chứng nhận chất lượng.

Kế hoạch vòng sau: học viên xem các ca trên ảnh gốc, sửa bằng CVAT nếu quyết định tiếp tục; rà các box nhỏ và mask có người lái; export lại rồi build release, lấy mẫu và QC lại. Việc sửa nhãn không thuộc vòng hoàn thiện QC/báo cáo này.

## 8. Ghi chú cá nhân

- **AI hỗ trợ:** Codex đọc hướng dẫn, kiểm định dạng, soạn tài liệu từ log, chạy script thống kê, kiểm trực quan QC và mask. Người kiểm vòng hiện tại là Codex-AI, không ghi giả là học viên tự kiểm.
- **Phạm vi nhãn:** giữ bản 24 ảnh/474 annotations. Các chỉnh box ngoài phạm vi của AI đã được khôi phục từ bản sao; 9 box xóa theo yêu cầu rõ ràng trước đó vẫn được giữ. CVAT chưa tự đồng bộ với JSON local.
- **Gold:** tự gán tay trước tour model. Có hỏi AI giải thích BBox, ignore_region, conf; không có yêu cầu AI gán hộ gold trong trao đổi. Vòng hoàn thiện này không đổi gold.
- **Môi trường:** Windows PowerShell, Python 3.12.10, PyTorch 2.15.0+cpu, Ultralytics 8.4.215, CVAT Docker v2.81.0. Bổ sung transformers 5.23.0 và huggingface_hub 2.4.0 vào môi trường có sẵn.
- **Nguồn dự đoán:** tự chạy model, không dùng data/cache cho các kết quả đã báo cáo.
- **Vướng mắc:** Docker chưa chạy ở lần kiểm ban đầu; model nằm ở thư mục cài riêng; YOLOE có FutureWarning torch.jit.load nhưng hoàn tất. Lỗi mất dấu khi ghi PowerShell đã sửa bằng UTF-8.
- **Thời gian thiếu:** chưa có số phút rework/QC được đo riêng; không dùng 30 phút review đầu làm tổng thời gian toàn bài.
- **Bài học:** confidence và số box không thay thế chất lượng; kiểm bẫy và QC độc lập với tỷ lệ accepted.
- **Git:** các mốc đã có 48f26a8, fccc740 và 6b31b7b. Chưa tạo commit chốt bài cho các tài liệu cập nhật; cần commit/push ở bước nộp.

### Kiểm tra hồ sơ trước nộp

Đã chạy validate_labels.py --all: gold 4 ảnh/116 annotations và batch 24 ảnh/474 annotations hợp lệ về cấu trúc; còn 20 và 13 cảnh báo kích thước nhỏ như đã ghi. check_submission.py hoàn tất với **0 lỗi, 0 cảnh báo** về hồ sơ; kết quả này không thay thế QC hình ảnh hoặc đổi quyết định Rework.

Đã cập nhật username GitHub từ origin là Edelzz và URL repo trong STUDENT.md. Commit chốt chưa tạo, không dùng HEAD cũ làm commit nộp cuối. Chưa push/nộp link trong vòng này.

Git cho biết data/images/img_013.jpg khác bản được theo dõi. Ảnh hiện vẫn hiển thị cùng cảnh và được giữ nguyên; chưa xác định đây là lưu lại JPEG hay chỉnh sửa khác. Cần đối chiếu nguồn trước khi đóng gói dữ liệu; không tự ghi đè ảnh của học viên. 