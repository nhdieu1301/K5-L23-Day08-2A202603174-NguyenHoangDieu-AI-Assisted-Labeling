# Thông tin học viên

- Họ và tên: Nguyễn Hoàng Diệu
- MSHV: 2A202603174
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: nhdieu1301
- Repo bài nộp (URL): https://github.com/nhdieu1301/K5-L23-Day08-2A202603174-NguyenHoangDieu-AI-Assisted-Labeling
- Commit chốt bài (hash): <4c5a2e6>
- Máy: Windows (8-core CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 390 / 17 / 68 (mẫu số: 475 box máy); added: 51 (mẫu số: 458 đối tượng)
- So với gold: precision / recall của **máy** 56.1% (37/66) / 77.1% (37/48)
- Bắt bẫy: 6/12 = 50.0%
- Thời gian: gán tay 5.62 phút/ảnh; duyệt có pre-label 1.25 phút/ảnh
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/95)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy chỉ đạt 50.0% (6/12 < 80%) và còn 6 lỗi bẫy lọt trong lô nên cần rà soát lại trước khi Release.