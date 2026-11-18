# Báo cáo Day 08 — 2A202603109 - Nguyen Hoang Viet Anh

## 1. Model và recipe

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---|---|
| person/bicycle/motorbike/car/bus/truck | YOLO26n | person 0.15, car 0.15; lớp khác 0.3 | `threshold_sweep.csv` |
| three_wheeler | YOLOE-26s, prompt tuk-tuk/cyclo/cargo tricycle | 0.15 | Chấp nhận tuk-tuk như một lớp riêng thay vì COCO `car`/`truck` |

Lệnh merge đã dùng:

`python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,car=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`

- SAM: SAM 2.1 tiny chạy trên CPU; MobileSAM dùng để đối chiếu `img_013`.
- YOLO26 không tạo ra lớp `three_wheeler`, nên đổi ngưỡng không giải quyết được lỗi tuk-tuk; cần YOLOE.
- Ngưỡng được chọn trên chính 4 ảnh gold, nên precision/recall lạc quan.

## 2. AI error profile

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 447 | 94.3% | 474 box máy |
| edited | 3 | 0.6% | 474 box máy |
| deleted | 24 | 5.1% | 474 box máy |
| added | 12 | 2.6% | 462 đối tượng sau duyệt |

Từ `review_stats.json`: `three_wheeler` có 11 box bị xóa và 2 box sửa; `person` có 12 box bị xóa. So với gold: precision 57.4%, recall 83.3%, mẫu 42 đối tượng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Bắt bẫy: 1/12 = 8.3%. Loại extra 0/3, wrong_class 0/3, loose 1/3, missing 0/3. Tỉ lệ này thấp rõ rệt nên batch chưa được xem là duyệt đáng tin.

Accepted 447/474 dù bắt bẫy thấp cho thấy duyệt chủ yếu giữ nguyên box máy mà chưa phát hiện đủ bẫy nhân tạo.

## 4. Máy có đáng dùng không?

Gán gold tay hết 15 phút cho 4 ảnh, 3.75 phút/ảnh; duyệt batch 30 phút cho 24 ảnh, 1.25 phút/ảnh.

Machine chưa đáng tin cho toàn lô ở thời điểm hiện tại. Gold cho recall 83.3% và precision 57.4%, đồng thời bắt bẫy chỉ 8.3%, nên cần Rework thay vì Release.

Ba quy tắc giảm thiên kiến:
1. Quét toàn ảnh trái sang phải trước khi xem box máy.
2. Kiểm box thừa, thiếu, sai lớp và chỉnh 4 cạnh.
3. Tick `can_review` cho ca khó rồi rà lại trước export.

## 5. Mask SAM

| Ảnh | SAM | Số điểm | Kết quả quan sát | File |
|---|---|---|---|---|
| img_013 | SAM 2.1 tiny + MobileSAM | 2 positive, 1 negative | SAM2 giữ/lem viền cần kiểm; MobileSAM đối chiếu chất lượng | tuktuk_img013_sam2.jpg, tuktuk_img013_sam1.jpg |
| img_004 | SAM 2.1 tiny | 2 positive, 1 negative | Cần tách người lái/hàng hóa khỏi xe | tuktuk_img004_sam2.jpg |
| img_017 | SAM 2.1 tiny | 2 positive, 1 negative | Cần kiểm phần mask dính người lái | tuktuk_img017_sam2.jpg |

Guideline mục 4.2 cho phép dùng điểm âm để tách người lái/hàng hóa, nhưng mask vẫn cần thêm điểm hoặc box nếu dính nền.

## 6. Calibration

Có 3 quyết định trong `qc/calibration_decisions.csv`. Quan trọng nhất là CAL-01: tuk-tuk/xích lô/ba gác luôn dùng `three_wheeler`; người lái dùng `person`. CAL-02 chốt vật nhỏ/`ignore_region`; CAL-03 chốt checklist review.

Nguyên nhân gốc: kết hợp quy tắc mở và sai số model.

## 7. QC và quyết định

Mẫu seed 42 gồm 5 ảnh ngẫu nhiên: img_003, img_006, img_014, img_031, img_035; không có ca `can_review` trong release candidate.

QC phiếu kiểm: 0/97 đối tượng lỗi (0.0%), 0/5 ảnh lỗi (0.0%). Theo `qc_report_auto.md`, cận trên Wilson 95% cho tỉ lệ ảnh lỗi thật là 43%, nên mẫu 5 ảnh không chứng minh ngưỡng 2%.

Không ghi nhận top lỗi nào trong mẫu 5 ảnh, nhưng kết quả không khẳng định lô sạch vì bẫy chỉ bắt 1/12.

**Quyết định: Rework**, ngày 2026-11-19.

Việc cần làm: duyệt lại batch theo checklist 4 bước, sửa đủ các bẫy extra/wrong_class/missing/loose, đạt ít nhất 10/12 bẫy, export lại `batch_reviewed.json`, chạy lại review_stats và QC.

## 8. Ghi chú cá nhân

- Dùng AI để chạy model, tạo mask, gộp pre-label và sinh báo cáo; kiểm tra bằng validate/review_stats và file nộp.
- Gold không bị nhiễm pre-label; gán trước khi chạy pre-label.
- Máy Windows, CPU, Python 3.12, CVAT Docker tại localhost:8080.
- Không dùng cache thay inference; có lỗi thiếu NumPy trong `.venv-1` Python 3.15, chuyển sang `.venv` Python 3.12.
- Lần sau sẽ dành thời gian xem đủ 24 ảnh và bắt 12 bẫy trước export.
