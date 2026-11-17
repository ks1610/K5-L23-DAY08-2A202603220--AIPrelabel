# Thông tin học viên

- Họ và tên: Trịnh Vinh Khánh
- MSHV: 2A202603220
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: ks1610
- Repo bài nộp (URL): https://github.com/ks1610/K5-L23-DAY08-2A202603220--AIPrelabel/tree/main
- Commit chốt bài (hash): 
- Máy:  Microsoft Windows 11 Pro, NVIDIA RTX A2000 Laptop GPU 
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 451 / 15 / 48 (mẫu số: 518 box máy; tương ứng 87.1% / 2.9% / 9.3%); added: 31 (mẫu số: 497 đối tượng sau duyệt, tương ứng 6.2%). Trong 79 box deleted + added có 7 trường hợp đổi lớp, nên số vật máy sót thật ≈ 24.
- So với gold: precision / recall của máy 54.7% / 90.6% (29 TP, 24 FP, 3 FN trên 32 vật gold; con số này lạc quan vì ngưỡng được chọn trên cùng 4 ảnh gold).
- Bắt bẫy: 8/12 = 66.7% (dưới ngưỡng 80%).
Thời gian: gán tay <<chưa có, cần điền từ time_log.csv>>; duyệt có pre-label <<chưa có>>.
- Lỗi QC: <<chưa có: qc_review.csv còn trống>>. Mẫu QC gồm 5 ảnh (img_003, 006, 014, 031, 035) với 97 đối tượng (23+24+17+3+30), dùng làm mẫu số khi điền.
- **Quyết định:** <<chưa có: cần QC và ít nhất 3 dòng calibration_decisions.csv trước>>. Theo chính review_stats.md, bắt bẫy dưới 80% là dấu hiệu nên duyệt lại lô và siết checklist, nên hướng Hold hoặc Rework là hợp lý, nhưng quyết định cuối cùng và lý do phải do bạn viết sau khi có QC.
