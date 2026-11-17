# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 97 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/97 = 0.0% | 97 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/5 = 0.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 451/518 = 87.1% / 15/518 = 2.9% / 48/518 = 9.3% | 518 box máy |
| Máy: added | 31/497 = 6.2% | 497 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Cảnh báo
- Dòng 2: error_type 'extra' sai (chỉ nhận khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask) - bỏ qua.
- Dòng 5: error_type 'loose' sai (chỉ nhận khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask) - bỏ qua.
- 2 ảnh trong mẫu CHƯA kiểm: img_003.jpg, img_031.jpg

## Giới hạn thống kê
- 0 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **43%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin
- LƯU Ý: còn ảnh mẫu chưa kiểm -> kết luận chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
