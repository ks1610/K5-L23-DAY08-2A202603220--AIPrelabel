# Thông tin học viên

- Họ và tên: Trịnh Vinh Khánh
- MSHV: 2A202603220
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: ks1610
- Repo bài nộp (URL): https://github.com/ks1610/K5-L23-DAY08-2A202603220--AIPrelabel/tree/main
- Commit chốt bài (hash): d3bd2056e8a9ead4649e4240d7c67faccc0c2ca1 
- Máy:  Microsoft Windows 11 Pro, NVIDIA RTX A2000 Laptop GPU 
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (số lấy từ `submission/qc/*`, kèm mẫu số)

- **Máy:** accepted / edited / deleted: 448 / 16 / 50 (mẫu số: 518 box máy; tương ứng 86.5% / 3.1% / 9.7%); added: 34 (mẫu số: 498 đối tượng sau duyệt; tương ứng 6.8%). Tổng accepted + edited + deleted = 514, còn thiếu 4 box so với 518. Trong 84 box deleted + added có 7 trường hợp đổi lớp, nên số vật máy sót thật ≈ 27.
- **So với gold:** precision / recall của máy 54.7% / 90.6% (29 TP, 24 FP, 3 FN trên 32 vật gold). Con số này lạc quan vì ngưỡng được chọn trên cùng 4 ảnh gold.
- **Bắt bẫy:** 11/12 = 91.7% (đạt ngưỡng 80%). 
- **Thời gian:** gán tay gold 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 1.63 phút/ảnh (39 phút / 24 ảnh). Dòng `batch_review` trong `time_log.csv`.
- **Lỗi QC (phiếu lần đầu):** 2/97 = 2.06% (extra 1 ở img_003, loose 1 ở img_031). Phiếu lần hai không có ghi chú kiểm tra nên chưa được chấp nhận. Sau khi kiểm tra trên release: <<số lỗi>>/<<số đối tượng>> = <<...%>>.
- **Quyết định:** Hold (tạm dừng, chờ xác minh trên release). Lý do: chưa có phiếu QC hợp lệ trên release và mẫu số chưa thống nhất (96 / 97 / 98).