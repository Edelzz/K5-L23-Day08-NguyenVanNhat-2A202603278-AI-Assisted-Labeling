# Thông tin học viên

- Họ và tên: Nguyễn Văn Nhất
- MSHV: 2A202603278
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: Edelzz
- Repo bài nộp (URL): https://github.com/Edelzz/K5-L23-Day08-NguyenVanNhat-2A202603278-AI-Assisted-Labeling
- Commit chốt bài (hash): 
- Máy: Windows, CPU, không dùng GPU
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt

- Máy: accepted 339/370 = 91.6%; edited 9/370 = 2.4%; deleted 16/370 = 4.3% (mẫu số: 370 box máy); added 107/455 = 23.5% (mẫu số: 455 đối tượng sau xử lý). Nguồn: submission/qc/review_stats.json.
- So với gold: precision 39/50 = 78.0%; recall 39/99 = 39.4%, trên 4 ảnh gold; chọn/chấm cùng gold nên có thể lạc quan.
- Bắt bẫy: lần đầu 4/12 = 33.3%; bản hiện tại sau sửa và biết đáp án 11/12 = 91.7%. Không dùng kết quả sau sửa thay cho đo mù.
- Thời gian: gán tay 15/4 = 3.75 phút/ảnh; review đầu 30/24 = 1.25 phút/ảnh. Chưa đo riêng thời gian rework/QC.
- Lỗi QC: 5/5 ảnh = 100%; 6/91 đối tượng = 6.6%. Mẫu seed 42, 5 ảnh ngẫu nhiên; reviewer Codex-AI. Cận trên Wilson 95% của tỷ lệ ảnh lỗi khoảng 100%.
- **Quyết định: Rework** — còn lỗi hình học/box thừa, chưa đạt ngưỡng QC. Giữ nguyên nhãn trong vòng QC/báo cáo hiện tại.
- Bộ candidate: 24 ảnh batch, 474 annotations; 3 ảnh mask SAM 2.1 base. Chi tiết và giới hạn trong submission/REPORT.md.
