# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 370; đối tượng sau duyệt: 467; ngưỡng accepted: IoU >= 0.9
- Đã **loại 18 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 357 | 357/370 = 96.5% | 370 box máy |
| edited (sửa) | 7 | 7/370 = 1.9% | 370 box máy |
| deleted (xoá) | 5 | 5/370 = 1.4% | 370 box máy |
| added (vẽ thêm) | 103 | 103/467 = 22.1% | 467 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 102.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 165 | 1 | 0 | 50 |
| bicycle | 3 | 0 | 2 | 4 |
| motorbike | 77 | 6 | 0 | 48 |
| car | 69 | 0 | 0 | 0 |
| bus | 2 | 0 | 0 | 0 |
| truck | 2 | 0 | 0 | 0 |
| three_wheeler | 39 | 0 | 3 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 99 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 11 | 60 | 78.0% (KTC 95%: 65–87%) | 39.4% (KTC 95%: 30–49%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - wrong_class: 1/3
  - loose: 0/3
  - extra: 2/3
  - lọt: img_024.jpg (missing), img_013.jpg (wrong_class), img_003.jpg (loose), img_029.jpg (extra), img_005.jpg (missing), img_034.jpg (loose), img_025.jpg (loose), img_028.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
