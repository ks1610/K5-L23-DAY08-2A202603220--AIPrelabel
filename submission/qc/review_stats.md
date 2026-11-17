# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 518; đối tượng sau duyệt: 497; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 451 | 451/518 = 87.1% | 518 box máy |
| edited (sửa) | 15 | 15/518 = 2.9% | 518 box máy |
| deleted (xoá) | 48 | 48/518 = 9.3% | 518 box máy |
| added (vẽ thêm) | 31 | 31/497 = 6.2% | 497 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 24.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 247 | 8 | 14 | 2 |
| bicycle | 6 | 0 | 2 | 2 |
| motorbike | 127 | 5 | 6 | 7 |
| car | 50 | 0 | 1 | 12 |
| bus | 3 | 0 | 0 | 3 |
| truck | 2 | 0 | 3 | 4 |
| three_wheeler | 16 | 2 | 22 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 32 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 24 | 3 | 54.7% (KTC 95%: 41–67%) | 90.6% (KTC 95%: 76–97%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 3/3
  - loose: 1/3
  - missing: 2/3
  - lọt: img_023.jpg (wrong_class), img_006.jpg (missing), img_033.jpg (loose), img_009.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
