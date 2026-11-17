# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603220 - Trịnh Vinh Khánh

> Chỉ viết phần thay đổi so với v1.0. Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao) và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe không động cơ, có bánh, chở người hoặc hàng (xích lô, xe đẩy hai bánh): thuộc lớp nào? | Gán lớp `three_wheeler`, vì guideline định nghĩa xe xích lô thuộc `three_wheeler` | img_003, img_008 | C02, C04 |
| 2 | Người ngồi trên phương tiện có gán `person` riêng không? | Gán `person` cho người ngồi trên phương tiện mà nhìn thấy được. Người đó không nằm trong mask của phương tiện | img_003, img_013 | C03, C07 |
| 3 | Hàng hoá trên phương tiện có nằm trong mask không? | Không. Chỉ mask phần nhìn thấy của xe như bánh xe và khung xe | img_003, img_008 | C05 |



## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Hai box `three_wheeler` (0.90 và 0.80) chồng lên cùng một xe đẩy hai bánh chở rau (C01) | YOLOE `three_wheeler` tạo hai box trùng nhau trên overlay | Xóa một box trùng, kéo box còn lại bao phủ toàn bộ vật thể | Mục 4 của v1.0: box trùng lặp |
| img_003 | Xe đẩy hai bánh chở rau, trẻ em và người lớn ngồi trên xe (C02, C03, C05) | Gán `three_wheeler` cho xe. Người ngồi trên xe không được gán riêng | Gán `three_wheeler` cho xe. Gán `person` cho người nhìn thấy được. Mask chỉ phần xe, không gồm hàng hoá | Mục 4 của v1.0: xe xích lô, người trên phương tiện, hàng hoá |
| img_008 | Hai xe đẩy hai bánh chở chuối (C04, C05) | <<chưa có overlay nhãn máy cho img_008, bạn cần bổ sung>> | Gán `three_wheeler`. Mask chỉ phần xe, không gồm chuối | Mục 4 của v1.0: xe xích lô, hàng hoá |
| img_013 | Tuk-tuk ba bánh có mái, bị gán `car` (conf 0.42), box cắt cụt đuôi xe (r04, C06) | YOLO26 gán `car`. Kết quả SAM từ box này bị sai (r04 trong `model_log.csv`) | Sửa thành `three_wheeler` | Mục 4 của v1.0: tuk-tuk |
| img_013 | Người lái và hành khách trong tuk-tuk (C07) | Mask tuk-tuk phủ một phần người lái và hành khách | Gán `person` cho người nhìn thấy được. Không nằm trong mask tuk-tuk | Mục 4 của v1.0: người trên phương tiện |


## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt: xe xích lô và xe đẩy thuộc `three_wheeler`; người trên phương tiện gán `person` riêng; hàng hoá không thuộc mask; box trùng lặp chỉ giữ một box; tuk-tuk gán `three_wheeler` | C01–C07 |
