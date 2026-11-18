# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 518; đối tượng sau duyệt: 498; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 448 | 448/518 = 86.5% | 518 box máy |
| edited (sửa) | 16 | 16/518 = 3.1% | 518 box máy |
| deleted (xoá) | 50 | 50/518 = 9.7% | 518 box máy |
| added (vẽ thêm) | 34 | 34/498 = 6.8% | 498 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 27.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 245 | 9 | 15 | 4 |
| bicycle | 6 | 0 | 2 | 2 |
| motorbike | 127 | 5 | 6 | 8 |
| car | 50 | 0 | 1 | 12 |
| bus | 3 | 0 | 0 | 3 |
| truck | 2 | 0 | 3 | 4 |
| three_wheeler | 15 | 2 | 23 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 32 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 24 | 3 | 54.7% (KTC 95%: 41–67%) | 90.6% (KTC 95%: 76–97%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 3/3
  - loose: 2/3
  - missing: 3/3
  - lọt: img_033.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
