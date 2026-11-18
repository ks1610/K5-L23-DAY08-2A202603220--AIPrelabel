# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603220 - Trịnh Vinh Khánh

> Chỉ viết phần thay đổi so với v1.0. Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp một dòng trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở từ mục 6 của v1.0

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Hàng hoá trên phương tiện có nằm trong mask không? | Không. Chỉ mask phần nhìn thấy của xe như bánh xe và khung xe, không mask hàng hoá | img_008 | CAL-02 |
| 2 | Xe không động cơ, có bánh (xích lô, xe đẩy hai bánh) thuộc lớp nào? | Gán nhãn `three_wheeler`| — | — |
| 3 | Người ngồi trên phương tiện có gán `person` riêng không? | Gán nhãn person cho những người ngồi trên xe mà có thể nhìn thấy được | img_023 | CAL_03 |
| 4 | Xe đạp điện: `bicycle` hay `motorbike`? | Chưa chốt. Chưa có ca nào trong `calibration_decisions.csv` quyết định câu này | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Hai box `three_wheeler` chồng lên cùng một xe đẩy hai bánh chở rau (img_003) | YOLOE `three_wheeler` tạo hai box trùng nhau (0.90 và 0.80) trên cùng một xe | Xóa một box trùng, kéo box còn lại bao phủ toàn bộ vật thể | Mục 4 của v1.0: mỗi phương tiện chỉ có một box |
| img_008 | Hai xe đẩy chở chuối, hàng hoá có nằm trong mask không (img_008) | YOLOE `three_wheeler` nhận hai xe (0.90 và 0.86), box 0.90 gộp cả xe trái và một phần xe giữa. SAM mask cả chuối khi click dương nằm trên chuối | Chỉ mask phần nhìn thấy của xe như bánh xe và khung xe. Không mask chuối. Mỗi xe một box, không gộp hai xe | Mục 4 của v1.0: hàng hoá không thuộc mask |
| img_023 | Box `person` bị đổi thành `bicycle` (bẫy wrong_class, img_023) | YOLO26 gán `person` 0.79, box đúng. Lỗi do bẫy trong `pre_traps.coco.json`, không phải do model | Gán `person` cho người nhìn thấy được, không gán `bicycle` | Mục 4 của v1.0: kiểm tra lớp của từng box trước khi Accept |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ảnh, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Hàng hoá trên phương tiện không thuộc mask | img_008 (CAL-02) |
| v1.1 | Box trùng chỉ giữ một box | img_003 (CAL-01) |
| v1.1 | Kiểm tra lớp của từng box trước khi Accept | img_023 (CAL-03) |