# Báo cáo Day 08 — <2A202603220 - Trịnh Vinh Khánh>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (run_id) |
|---|---|---|---|
| person, motorbike, car, bus, truck, bicycle | YOLO26-m (`--model yolo26 --size m`) | 0.30 (mặc định của merge, `--conf 0.3`) | r01 và r02: YOLO26 nhận đúng person, motorbike, car, bus phổ thông với conf 0.52–0.95. Ngưỡng 0.5 (r02) lọc được nhiễu nhưng vẫn không giải quyết được three_wheeler, nên dùng mặc định. |
| three_wheeler | YOLOE-s, prompt `three_wheeler` | 0.15 (theo `threshold_sweep`) | r03: nhận đúng tuk-tuk ngang (0.93), tuk-tuk cắt mép (0.28) và xích lô; 0 nhận nhầm trên ảnh không có xe ba bánh. r01 và r02 cho thấy YOLO26 COCO không bắt được lớp này. |

- Lệnh `merge_prelabels.py` đã dùng:
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  Kết quả: 606 box (person 311, motorbike 154, car 65, three_wheeler 52, bicycle 13, truck 7, bus 4).

- **Phiên bản SAM và lý do:** <<SAM 2.1 (`sam2.1_b.pt`) theo r04. Điền lý do cụ thể: chất lượng mask, tốc độ (r04 chạy 5.84 s/ảnh trên CPU), RAM, và việc SAM 3 cần quyền truy cập>>

- **Lỗi không sửa được bằng đổi ngưỡng:** tuk-tuk bị gọi là motorbike (r01, img_007, conf 0.28–0.33) hoặc truck (r01 và r02, img_011, conf 0.77). Theo r02, nâng ngưỡng lên 0.5 làm mất các xe ba bánh có conf thấp nhưng không sửa được việc gán nhầm lớp. Cách sửa là đổi detector sang YOLOE với prompt three_wheeler (r03).

- **Box điểm cao mà sai:** truck 0.77 trên img_011 (r01, r02), thực chất là tuk-tuk. Box điểm cao nhưng sai lớp, nên ngưỡng không lọc được.
  **Box điểm thấp mà đúng:** tuk-tuk ở img_011 với conf 0.28 (YOLOE, r03) và xe ba bánh ở img_007 với conf 0.28–0.33 (YOLO26, r01). Nếu đặt ngưỡng cao hơn, các box này sẽ bị loại.

- **Giới hạn:** chọn ngưỡng trên chính 4 ảnh gold nên số đo của máy trên gold là lạc quan. Với person, motorbike và car, recall trên gold chỉ đạt khoảng 88% ở ngưỡng mặc định, chưa đạt mục tiêu 90% trong `threshold_sweep.md`. Với bicycle, bus và truck, gold có ít hơn 5 vật nên không chọn ngưỡng riêng được.

## 2. AI error profile (đã điền)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 451 | 87.1% | 518 box máy |
| edited | 15 | 2.9% | 518 box máy |
| deleted | 48 | 9.3% | 518 box máy |
| added | 31 | 6.2% | 497 đối tượng sau duyệt |

- **Lớp nào máy tệ nhất?** Dựa trên tỉ lệ accepted trong số box máy của từng lớp (accepted / (accepted + edited + deleted)):
  - three_wheeler: 16 / 40 = **40%**, có 22 box bị xoá, nhiều nhất trong các lớp. Đây là lớp tệ nhất về số lượng lỗi.
  - truck: 2 / 5 = **40%**, nhưng mẫu quá nhỏ (5 box) nên chưa đủ để kết luận.
  - bicycle: 6 / 8 = 75%
  - person: 247 / 269 = 92%
  - motorbike: 127 / 138 = 92%
  - car: 50 / 51 = 98%
  - bus: 3 / 3 = 100%, mẫu quá nhỏ.

  Kết luận: **three_wheeler** là lớp máy tệ nhất. Dẫn ảnh: img_003_1 có hai box three_wheeler (0.90 và 0.80) chồng lên cùng một xe đẩy, và img_013 là tuk-tuk bị gán sai trong nhãn gốc.

- **So với gold:** precision / recall của máy là **54.7% / 90.6%** (mẫu số: 32 đối tượng gold, gồm 29 TP, 24 FP, 3 FN). Con số này lạc quan vì ngưỡng được chọn trên chính 4 ảnh gold.

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


## 3. Thiên kiến tự động hóa và ảnh bẫy

**Kết quả bắt bẫy:** 8/12 = 66.7%, dưới ngưỡng 80% trong `review_stats.md`. Khoảng tin cậy 95% rất rộng (39–86%) vì chỉ có 12 bẫy.

| Loại bẫy | Bắt được | Lọt |
|---|---|---|
| wrong_class | 2/3 | img_023 |
| extra | 3/3 | — |
| loose | 1/3 | img_033, img_009 |
| missing | 2/3 | img_006 |

**Phân tích 4 bẫy bị lọt.** Các số IoU tính từ tọa độ trong `traps_key.json`, so với box gốc. Ngưỡng accepted là IoU ≥ 0.9.

| Ảnh | Loại | Chi tiết lỗi | Bước checklist bị bỏ | Nguyên nhân khả dĩ |
|---|---|---|---|---|
| img_023 | wrong_class | Box `person` bị đổi thành `bicycle` (113×91 px) | Bước 4: kiểm tra nhãn | Chỉ kiểm tra vị trí box, không đối chiếu lớp |
| img_006 | missing | Box `car` bị xoá khỏi nhãn máy (85×48 px) | Bước 1: quét ảnh trước | Nhìn nhãn máy trước nên không tìm vật còn thiếu |
| img_033 | loose | Box `motorbike` lệch, IoU ≈ 0.56 | Bước 5: kiểm tra độ khít | Không chỉnh box, chấp nhận |
| img_009 | loose | Box `person` lệch, IoU ≈ 0.66 | Bước 5: kiểm tra độ khít | Không chỉnh box, chấp nhận dù xem 3 phút |

**Nhận định về thiên kiến tự động hóa:**
- Accepted đạt 87.1% nhưng bắt bẫy chỉ 66.7%. Khoảng cách này cho thấy phần lớn box máy được chấp nhận mà không được kiểm tra thực sự.
- Thời gian duyệt không khác nhau giữa ảnh có bẫy và không có bẫy, nên quá trình duyệt không tập trung vào những chỗ cần kiểm tra.

## 4. Máy có đáng dùng không?

**Thời gian** (từ `time_log.csv`):
- Gán tay gold: 15 phút / 4 ảnh = **3.75 phút/ảnh**
- Duyệt batch có pre-label: 39 phút / 24 ảnh = **1.63 phút/ảnh**

Dòng `batch_review` ghi tổng 40 phút nhưng cộng các ảnh được 39 phút. Báo cáo này dùng 39 phút. Cần sửa lại dòng này trước khi nộp.

**Thiên lệch:** Gán gold là lần đầu làm, chưa quen guideline và chưa có nhãn máy tham khảo, nên tốn nhiều thời gian hơn bình thường. Duyệt batch làm sau, đã quen guideline. Vì vậy chênh lệch 2.3 lần không thể quy hoàn toàn cho máy. Hai số này chỉ dùng để tham khảo xu hướng.

**Thời gian không phân biệt ảnh có lỗi và không có lỗi:**

| Nhóm | Số ảnh | Phút | Trung bình |
|---|---|---|---|
| Có bẫy, bắt được | 8 | 11 | 1.38 |
| Có bẫy, lọt | 4 | 6 | 1.50 |
| Không có bẫy | 12 | 22 | 1.83 |

Thời gian gần như đồng đều giữa các nhóm, cho thấy bạn không dành thêm chú ý cho ảnh có vấn đề.

**Kết luận theo lớp:** Dùng tỉ lệ accepted trên số box máy của từng lớp (accepted / accepted + edited + deleted).

| Lớp | Accepted | Tỉ lệ | Đánh giá |
|---|---|---|---|
| car | 50/51 | 98% | Đáng tin |
| bus | 3/3 | 100% | Mẫu quá nhỏ |
| motorbike | 127/138 | 92% | Đáng tin |
| person | 247/269 | 92% | Đáng tin |
| bicycle | 6/8 | 75% | Cần kiểm tra thêm |
| truck | 2/5 | 40% | Mẫu quá nhỏ |
| three_wheeler | 16/40 | 40% | Không đáng tin |

- **Kết luận:** Máy **có ích có điều kiện**. Chỉ nên dùng pre-label cho person, motorbike và car, và phải duyệt kỹ. Với three_wheeler, máy gần như không đáng tin: 22 trên 40 box bị xoá, và chỉ nên dùng làm gợi ý để vẽ tay.
- Cần lưu ý rằng ngay cả với các lớp tốt, việc chấp nhận 87.1% mà bắt bẫy chỉ 66.7% cho thấy độ tin cậy thực tế của quá trình duyệt chưa cao. Máy giảm công vẽ, nhưng không thay được kiểm tra.

**Ba quy tắc giảm thiên kiến** (mỗi quy tắc kiểm tra được bằng số):
1. **Quét ảnh trước khi bật nhãn máy.** Ghi số vật đếm được bằng mắt, rồi so với số box máy. Kiểm tra: số ảnh có ghi số đếm trước phải đạt 100% trong lần duyệt sau. Mục tiêu: giảm lỗi `missing`.
2. **Kiểm tra cả lớp và độ khít của từng box trước khi Accept.** Với mỗi box, xác nhận lớp và chỉnh nếu lệch rõ. Kiểm tra: số box được chỉnh (edited) tăng so với 15 lần trước, và bắt bẫy `wrong_class` và `loose` đạt ít nhất 80%.
3. **Nghỉ 1–2 phút sau mỗi 10 ảnh và ghi thời gian.** Kiểm tra: `time_log.csv` có dòng nghỉ, và thời gian trung bình ảnh có bẫy không thấp hơn ảnh không có bẫy quá 20%.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>


## 5. Mask SAM cho 3 vật (Part 5)

Phiên bản SAM dùng cho cả ba mask là **SAM 2.1 (`sam2.1_b.pt`)**. Hiện chỉ có **1 mask đạt** (img_008). Bài yêu cầu tối thiểu 3 mask, nên mục này chưa hoàn thành.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_008 (hai xe chở chuối, xe bên trái) | SAM 2.1 (`sam2.1_b.pt`) | 2 dương, 2 âm | Không có người nào trong mask. Chuối trên xe **nằm trong mask**, cần quyết định theo guideline. | Không có lỗ lớn. Mask đạt. | `cart_img008.jpg` (**đạt**) |
| img_003 (xe đẩy rau, có trẻ em) | SAM 2.1 (`sam2.1_b.pt`) | 2 dương, 4 âm | Trẻ em và người kéo xe **đã loại khỏi mask** (đúng quy tắc 4). Rau và bát trên xe **không được phủ**. | Thiếu bánh xe bên phải (phía trước, khoảng x=610, y=570). Thiếu phần rau và bát. | `cart_img003_1.jpg` (**chưa đạt**) |
| img_013 (tuk-tuk ba bánh) | SAM 2.1 (`sam2.1_b.pt`) | 4 dương, 2 âm (ước lượng) | Người lái và hành khách sau **bị phủ một phần** (vi phạm quy tắc 4). Hàng hoá: không có. | Có đốm tối và lỗ ở vùng cửa phải và thân xe (vi phạm quy tắc 2). Quần jeans người lái chưa loại được. | `tuktuk_img013_347b86.jpg` (**chưa đạt**) |

### Đối chiếu guideline mục 4.2

- **Quy tắc 1 (biên ≤ 3 px):** chưa kiểm tra được bằng mắt. Cần đo trong CVAT.
- **Quy tắc 2 (lỗ thủng):** img_013 không đạt. Các lỗ ở thân xe chưa được lấp.
- **Quy tắc 3 (bóng đổ, kính):** img_008 và img_003 đạt, không phủ bóng đổ. img_013 đạt, kính trước thuộc mask.
- **Quy tắc 4 (vật chạm nhau không chồng):** img_008 đạt. img_003 đạt một phần (đã loại trẻ em và người kéo, nhưng thiếu bánh). img_013 không đạt.

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


Dữ liệu đã đủ để điền mục 6 và 7, nhưng có hai lỗi trong file cần sửa trước, và một số phần phải do bạn xác nhận.


## Mục 6. Calibration (Part 6)

- **Số quyết định:** 7 dòng (C01–C07) trong `calibration_decisions.csv`. Đạt yêu cầu ≥3 dòng, có 2 `loi_may` (C01, C06) và 5 `cau_hoi_mo` (C02, C03, C04, C05, C07).
- **Quan trọng nhất: C01.** QC đã phát hiện img_003 vẫn còn box trùng trong nhãn. Điều này cho thấy quyết định C01 chưa được áp dụng vào nhãn. Phải sửa nhãn img_003 trước khi chốt.
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:**
  - Mục 1 (câu hỏi mở đã chốt): xe xích lô và xe đẩy thuộc `three_wheeler`; người ngồi trên phương tiện được gán `person` riêng; hàng hoá không thuộc mask.
  - Mục 2 (ca khó): box trùng chỉ giữ một box; tuk-tuk gán `three_wheeler`.
  - Chưa hoàn thiện: câu xe đạp điện và dòng img_008 trong mục 2.
- **Nguyên nhân gốc (đề xuất, bạn cần xác nhận):**
  - C01, C06: lỗi của **máy**. YOLOE tạo box trùng, YOLO26 COCO gán tuk-tuk thành `car`.
  - C02, C04: **guideline mơ hồ**. v1.0 không nêu rõ xe không động cơ.
  - C03, C07: **guideline mơ hồ**. v1.0 không nêu người ngồi trong phương tiện.
  - C05: **guideline mơ hồ**. v1.0 không quy định hàng hoá.

## Mục 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh (img_003, img_006, img_014, img_031, img_035), tổng 97 đối tượng. `qc_sample_1.csv` không có cột `can_review`, nên tôi không xác định được số ảnh `can_review` và ngẫu nhiên. Bạn cần cho biết mẫu được chọn thế nào và có dùng seed 42 không. Người kiểm: bạn (**hạn chế:** bạn vừa là người duyệt).
- **Lỗi theo đối tượng:** 2/97 = **2.06%**, vượt ngưỡng Release (≤ 2%). Nếu img_006 có xe ô tô bị thiếu (nghi ngờ đã nêu trước đó), số lỗi là 3/97 = **3.09%**.
- **Lỗi theo ảnh:** 2/5 = **40%**.
- **Top lỗi:** `extra` 1 (img_003, hai box trùng), `loose` 1 (img_031, box bicycle bao cả giỏ hàng). Chưa xác nhận lỗi `missing` ở img_006.
- **Ảnh ví dụ:** img_003 cho `extra`; img_031 cho `loose`.
- **Giới hạn mẫu nhỏ:** cận trên 95% theo Wilson cho 2/97 khoảng **7.2%**, cận dưới khoảng 0.6%. Đây là tính tay, bạn cần đối chiếu với `qc_report_auto.md`.
- **Script kiểm được / không kiểm được (cần bạn xác nhận):**
  - Kiểm được bằng script: định dạng CSV, dòng trống, id khớp file release.
  - Không kiểm được bằng script: box trùng về mặt ngữ cảnh, box lỏng, sai theo quy tắc người ngồi trong phương tiện, mask. Cần kiểm bằng mắt.
  - Bạn không nên dùng `traps_key.json` để kiểm các ảnh này, vì bạn đã mở key trước đó.

**Quyết định: Rework** (đề xuất), ngày 2026-11-18.

Lý do: tỉ lệ lỗi 2.06% vượt ngưỡng Release một chút. Hai lỗi đã xác định cụ thể và có thể sửa: xoá box trùng ở img_003 và chỉnh box bicycle ở img_031. Chưa có bằng chứng lỗi hệ thống nghiêm trọng, nhưng img_006 vẫn chưa xác nhận. Nếu img_006 thiếu xe, tỉ lệ tăng lên 3.09%, vẫn trong phạm vi sửa được bằng Rework.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Gemini cho khắc phục các lỗi trng quá trình chạy code.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không 
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? không 
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>