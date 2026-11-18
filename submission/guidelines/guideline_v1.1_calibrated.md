# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603109 - Nguyen Hoang Viet Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Tuk-tuk/xích lô/ba gác thuộc `three_wheeler` hay `car`/`truck`? | Luôn gán `three_wheeler`; người lái gán riêng `person`. | img_013, img_004 | CAL-01 |
| 2 | Cụm vật dưới 20 px hoặc vật chen sát nhau xử lý thế nào? | Có thể tách từng cá thể thì gán riêng; cụm không thể tách rõ dùng `ignore_region`. | img_017, img_023 | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_006, img_027 | Box motorbike/three_wheeler bị thiếu hoặc lỏng | YOLO26/YOLOE | Xác nhận bằng mắt, chỉnh 4 cạnh và đánh dấu ca khó khi cần | Quy tắc review mục 4 của v1.0 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt lớp `three_wheeler`, quy tắc vật nhỏ/`ignore_region` và checklist quét toàn ảnh | CAL-01, CAL-02, CAL-03 |
