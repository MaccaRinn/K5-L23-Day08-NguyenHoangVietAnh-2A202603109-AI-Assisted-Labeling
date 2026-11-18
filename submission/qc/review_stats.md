# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 474; đối tượng sau duyệt: 462; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 447 | 447/474 = 94.3% | 474 box máy |
| edited (sửa) | 3 | 3/474 = 0.6% | 474 box máy |
| deleted (xoá) | 24 | 24/474 = 5.1% | 474 box máy |
| added (vẽ thêm) | 12 | 12/462 = 2.6% | 462 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 271 | 0 | 12 | 0 |
| bicycle | 4 | 0 | 0 | 1 |
| motorbike | 79 | 1 | 1 | 11 |
| car | 59 | 0 | 0 | 0 |
| bus | 2 | 0 | 0 | 0 |
| truck | 2 | 0 | 0 | 0 |
| three_wheeler | 30 | 2 | 11 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 42 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 35 | 26 | 7 | 57.4% (KTC 95%: 45–69%) | 83.3% (KTC 95%: 69–92%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 1/12 = 8.3% (KTC 95%: 1–35%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 0/3
  - wrong_class: 0/3
  - loose: 1/3
  - missing: 0/3
  - lọt: img_023.jpg (extra), img_025.jpg (wrong_class), img_017.jpg (missing), img_009.jpg (wrong_class), img_033.jpg (wrong_class), img_018.jpg (missing), img_006.jpg (loose), img_027.jpg (loose), img_004.jpg (extra), img_005.jpg (missing), img_034.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
