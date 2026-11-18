# Báo cáo Day 08: 2A202603220 - Trịnh Vinh Khánh

> Một file duy nhất cho cả bài. Mọi con số kèm **mẫu số** và nguồn (file nào).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (run_id) |
|---|---|---|---|
| person, motorbike, car, bus, truck, bicycle | YOLO26-m (`--model yolo26 --size m`) | 0.30 (mặc định của merge, `--conf 0.3`) | r01 và r02: YOLO26 nhận đúng person, motorbike, car, bus phổ thông với conf 0.52–0.95. Ngưỡng 0.5 (r02) lọc được nhiễu nhưng không giải quyết được three_wheeler, nên dùng mặc định. |
| three_wheeler | YOLOE-s, prompt `three_wheeler` | 0.15 (theo `threshold_sweep`) | r03: nhận đúng tuk-tuk ngang (0.93), tuk-tuk cắt mép (0.28) và xích lô; không nhận nhầm trên ảnh không có xe ba bánh. r01 và r02 cho thấy YOLO26 COCO không bắt được lớp này. |

- Lệnh `merge_prelabels.py` đã dùng:
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  Kết quả: 606 box (person 311, motorbike 154, car 65, three_wheeler 52, bicycle 13, truck 7, bus 4).

- **Phiên bản SAM và lý do:** SAM 2.1 (`sam2.1_b.pt`) theo r04. Do tốc độ (r04 chạy 5.84 s/ảnh trên CPU), RAM

- **Lỗi không sửa được bằng đổi ngưỡng:** tuk-tuk bị gọi là motorbike (r01, img_007, conf 0.28–0.33) hoặc truck (r01 và r02, img_011, conf 0.77). Theo r02, nâng ngưỡng lên 0.5 làm mất các xe ba bánh có conf thấp nhưng không sửa được việc gán nhầm lớp. Cách sửa là đổi detector sang YOLOE với prompt three_wheeler (r03).

- **Box điểm cao mà sai:** truck 0.77 trên img_011 (r01, r02), thực chất là tuk-tuk. Box điểm cao nhưng sai lớp nên ngưỡng không lọc được.
  **Box điểm thấp mà đúng:** tuk-tuk ở img_011 với conf 0.28 (YOLOE, r03) và xe ba bánh ở img_007 với conf 0.28–0.33 (YOLO26, r01). Nếu đặt ngưỡng cao hơn, các box này sẽ bị loại.

- **Giới hạn:** chọn ngưỡng trên chính 4 ảnh gold nên số đo của máy trên gold là lạc quan. Với person, motorbike và car, recall trên gold chỉ đạt khoảng 88% ở ngưỡng mặc định, chưa đạt mục tiêu 90% trong `threshold_sweep.md`. Với bicycle, bus và truck, gold có ít hơn 5 vật nên không chọn ngưỡng riêng được.

## 2. AI error profile (`review_stats`, lần chạy mới nhất)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 448 | 86.5% | 518 box máy |
| edited | 16 | 3.1% | 518 box máy |
| deleted | 50 | 9.7% | 518 box máy |
| added | 34 | 6.8% | 498 đối tượng sau duyệt |

- Tổng accepted + edited + deleted = 514, còn thiếu **4 box** so với 518 box máy. Báo cáo đã loại 27 đối tượng quanh bẫy khỏi bảng.
- Trong 84 box deleted + added có 7 trường hợp đổi lớp, nên số vật máy sót thật ≈ 34 − 7 = **27**.
- **Lớp nào máy tệ nhất?** Tỉ lệ accepted trên số box máy của từng lớp (accepted / (accepted + edited + deleted)):
  - three_wheeler: 15 / 40 = **37.5%**, 23 box bị xoá. Đây là lớp tệ nhất.
  - truck: 2 / 5 = 40%, mẫu quá nhỏ.
  - bicycle: 6 / 8 = 75%.
  - person: 245 / 269 = 91.1%.
  - motorbike: 127 / 138 = 92.0%.
  - car: 50 / 51 = 98.0%.
  - bus: 3 / 3 = 100%, mẫu quá nhỏ.

  Kết luận: **three_wheeler** là lớp máy tệ nhất. Dẫn ảnh: img_003 có hai box three_wheeler (0.90 và 0.80) chồng lên cùng một xe đẩy, và img_013 là tuk-tuk bị gán `car` trong nhãn máy.

- **So với gold:** precision / recall của máy là **54.7% / 90.6%** (mẫu số: 32 đối tượng gold, gồm 29 TP, 24 FP, 3 FN). Con số này lạc quan vì ngưỡng được chọn trên chính 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

**Kết quả bắt bẫy:** 11/12 = **91.7%**, đạt ngưỡng 80%. Khoảng tin cậy 95% rất rộng (65–99%) vì chỉ có 12 bẫy.

| Loại bẫy | Bắt được | Lọt |
|---|---|---|
| wrong_class | 3/3 | — |
| extra | 3/3 | — |
| loose | 2/3 | img_033 |
| missing | 3/3 | — |

**Bẫy bị lọt:**

| Ảnh | Loại | Chi tiết lỗi | Bước checklist bị bỏ | Nguyên nhân khả dĩ |
|---|---|---|---|---|
| img_033 | loose | Box `motorbike` lệch, IoU ≈ 0.56 so với box gốc | Bước 5: kiểm tra độ khít | Chấp nhận box mà không chỉnh |


**Cần xác nhận về quy trình:** bắt bẫy tăng qua ba lần chạy: 8/12, rồi 10/12, rồi 11/12. Nếu bạn sửa nhãn sau khi đã mở `traps_key.json`, con số này không phản ánh đúng năng lực duyệt ban đầu. Bạn phải ghi rõ trong báo cáo: lần duyệt nào làm trước khi mở key, lần nào làm sau. Nếu không, con số bắt bẫy sau cùng chỉ có giá trị minh họa.

**Nhận định về thiên kiến tự động hóa:**
- Accepted 86.5% trong khi bắt bẫy 91.7%. Bẫy `missing` đã bắt đủ 3/3 trong lần duyệt này, nhưng ban đầu lọt 1 bẫy, cho thấy việc quét ảnh trước khi nhìn nhãn máy là bước quan trọng nhất.
- Bẫy còn lọt (img_033) là lỗi độ khít, không phải lỗi về lớp.

## 4. Máy có đáng dùng không?

**Thời gian** (từ `time_log.csv`):
- Gán tay gold: 15 phút / 4 ảnh = **3.75 phút/ảnh**
- Duyệt batch có pre-label: 39 phút / 24 ảnh = **1.63 phút/ảnh**

**Thiên lệch:** Gán gold là lần đầu, chưa quen guideline và chưa có nhãn máy tham khảo, nên tốn nhiều thời gian hơn. Duyệt batch làm sau, đã quen guideline. Chênh lệch 2.3 lần không thể quy hoàn toàn cho máy.

**Thời gian theo nhóm ảnh** (ghép `time_log.csv` với danh sách 12 ảnh có bẫy trong `traps_key.json`):

| Nhóm | Số ảnh | Phút | Trung bình |
|---|---|---|---|
| Có bẫy, bắt được | 11 | 16 | 1.45 |
| Có bẫy, lọt | 1 | 1 | 1.00 |
| Không có bẫy | 12 | 22 | 1.83 |

Ảnh có bẫy được duyệt nhanh hơn ảnh không có bẫy. Đây là quan sát, không phải kết luận: nhóm lọt chỉ có 1 ảnh.

**Kết luận theo lớp:** tỉ lệ accepted trên số box máy của từng lớp.

| Lớp | Accepted | Tỉ lệ | Đánh giá |
|---|---|---|---|
| car | 50/51 | 98.0% | Đáng tin |
| bus | 3/3 | 100% | Mẫu quá nhỏ |
| motorbike | 127/138 | 92.0% | Đáng tin |
| person | 245/269 | 91.1% | Đáng tin |
| bicycle | 6/8 | 75% | Cần kiểm tra thêm |
| truck | 2/5 | 40% | Mẫu quá nhỏ |
| three_wheeler | 15/40 | 37.5% | Không đáng tin |

- **Kết luận:** Máy **có ích có điều kiện**. Dùng pre-label cho person, motorbike và car, nhưng phải duyệt kỹ. Với three_wheeler, 23 trên 40 box bị xoá, nên chỉ dùng làm gợi ý để vẽ tay.
- Máy giảm công vẽ, nhưng không thay được kiểm tra. Accepted cao vẫn đi kèm bắt bẫy chưa đạt 100%.

**Ba quy tắc giảm thiên kiến** (mỗi quy tắc kiểm tra được bằng số):
1. **Quét ảnh trước khi bật nhãn máy.** Ghi số vật đếm được bằng mắt, so với số box máy. Kiểm tra: mọi ảnh đều có số đếm trước khi duyệt. Mục tiêu: giảm lỗi `missing`.
2. **Kiểm tra độ khít của từng box trước khi Accept.** Kiểm tra: số box được chỉnh (edited) tăng so với 16 lần trước, và bắt bẫy `loose` đạt 3/3.
3. **Nghỉ 1–2 phút sau mỗi 10 ảnh và ghi thời gian.** Kiểm tra: `time_log.csv` có dòng nghỉ.

## 5. Mask SAM cho 3 vật (Part 5)

Phiên bản SAM dùng cho cả ba mask là **SAM 2.1 (`sam2.1_b.pt`)**. Tính đến lần chạy này, **chưa có mask nào đạt đủ quy tắc 4.2**. Bài yêu cầu tối thiểu 3 mask đạt, nên mục này chưa hoàn thành.

| Ảnh | SAM (phiên bản) | Số click (dương / âm) | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_003 (xe đẩy rau, có trẻ em) | SAM 2.1 (`sam2.1_b.pt`) | 2 / 4 (lần chạy `cart_img003`) | Trẻ em và người kéo xe đã loại khỏi mask. Rau và bát không được phủ. | Thiếu bánh xe bên phải (x≈610, y≈570). | `cart_img003.jpg` (**đạt một phần**) |
| img_008 (hai xe đẩy chở chuối, xe trái) | SAM 2.1 (`sam2.1_b.pt`) | 2 / 2 (lần chạy `cart_img008`) | Không có người trong mask. Click dương trên chuối làm mask có thể bao hàng hoá. | Thiếu bánh thứ hai (x≈345–475, y≈440–580). | `cart_img008.jpg` (**chưa đạt**) |
| img_013 (tuk-tuk ba bánh) | SAM 2.1 (`sam2.1_b.pt`) | 2 / 1 (lần chạy `tuktuk_img013`) | Người lái và hành khách sau bị phủ một phần. Không có hàng hoá. | Lỗ tối ở vùng kính trước và thân xe (quy tắc 2). Người lái và hành khách vi phạm quy tắc 4. | `tuktuk_img013.jpg` (**chưa đạt**) |

### Đối chiếu guideline mục 4.2

- **Quy tắc 1 (biên ≤ 3 px):** chưa kiểm tra được bằng mắt. Cần đo trong CVAT.
- **Quy tắc 2 (lỗ thủng):** img_013 chưa đạt. Các lỗ ở kính và thân xe chưa được lấp.
- **Quy tắc 3 (bóng đổ, kính):** cả ba mask không phủ bóng đổ. Kính trước của tuk-tuk thuộc mask.
- **Quy tắc 4 (vật chạm nhau không chồng):** img_003 đạt một phần, img_008 chưa đạt do click dương trên hàng hoá, img_013 chưa đạt do người lái và hành khách còn bị phủ.

### Câu hỏi mở chưa có quy tắc trong guideline 4.2

- Hàng hoá trên xe (chuối, rau, bát) có thuộc mask phương tiện không? Guideline v1.1 đã chốt là không (CAL-02).
- Người ngồi trên xe có được gán `person` riêng và loại khỏi mask không? 

## 6. Calibration (Part 6)

- **Số quyết định:** 3 ca (CAL-01 đến CAL-03) trong `calibration_decisions.csv`. Đạt yêu cầu ≥3 dòng, có 1 `loi_may` (CAL-01), 1 `cau_hoi_mo` (CAL-02), và 1 `bay_lot` (CAL-03).
- **Quan trọng nhất: CAL-01.** Box trùng trên img_003 là lỗi đã được QC phát hiện, cần kiểm tra lại trên release xem quyết định đã được áp dụng chưa.
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:**
  - Hàng hoá trên xe không thuộc mask (CAL-02).
  - Box trùng chỉ giữ một box (CAL-01).
  - Kiểm tra lớp của từng box trước khi Accept (CAL-03).
  - Chưa hoàn thiện: các câu hỏi mục 1 không có ca trong CSV (xe không động cơ, người ngồi trên phương tiện, xe đạp điện) được ghi là "chưa chốt".
- **Nguyên nhân gốc:**
  - **CAL-01 (box trùng img_003): máy.** YOLOE `three_wheeler` tạo hai box trên cùng một xe. Guideline v1.0 không có quy tắc xử lý box trùng, nên đây cũng là điểm bổ sung guideline.
  - **CAL-02 (hàng hoá trong mask img_008): guideline mơ hồ.** Mục 4.2 v1.0 không quy định hàng hoá trên phương tiện.
  - **CAL-03 (bẫy wrong_class img_023): quy trình.** Lỗi do bẫy đổi `person` thành `bicycle`.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh (img_003, img_006, img_014, img_031, img_035), 97 đối tượng theo `qc_sample_1.csv`. Script báo 96, cần giải thích sự chênh lệch. Mẫu chọn ngẫu nhiên, seed 42 
- **Phiếu QC lần đầu:** 2 lỗi (`extra` ở img_003, `loose` ở img_031) → 2/97 = **2.06%**, vượt ngưỡng Release (≤ 2%).
- **Phiếu QC lần hai:** 0 lỗi, nhưng không có ghi chú kiểm tra và không có `object_id` → **chưa được chấp nhận làm bằng chứng**.
- **Lỗi theo ảnh:** 2/5 = 40% (phiếu lần đầu).
- **Cận trên 95% (Wilson):** 2/97 → khoảng 7.2%. Đối chiếu với `qc_report_auto.md`.
- **Giới hạn:** mẫu 5 ảnh quá nhỏ để chứng minh tỉ lệ lỗi dưới 2%. Bạn đã mở `traps_key.json` trước khi kiểm, nên kết quả bắt bẫy không tách bạch hoàn toàn với QC.

**Quyết định: Hold** (tạm dừng, chờ xác minh trên release), ngày 2026-11-18.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Gemini, hỗ trợ khắc phục lỗi trong quá trình chạy code.
- **Gold có bị nhiễm không** (có nhìn pre-label hoặc hỏi AI trước khi gán xong): không.
- **Máy / GPU / CVAT dùng; vướng mắc cài đặt:** 
- **Dùng cache `data/cache/` thay vì tự chạy:** không.
- **Lỗi / bug script đã gặp:** 
- **Một điều bạn làm khác lần sau:** 

