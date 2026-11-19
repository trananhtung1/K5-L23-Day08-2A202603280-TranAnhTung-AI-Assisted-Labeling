# Thông tin học viên

- Họ và tên: Trần Anh Tùng
- MSHV: 2A202603280
- Nhóm repo: K5-L23. Phân lớp L2/L3 chưa xác nhận.
- Tài khoản GitHub: trananhtung1
- Repo bài nộp (URL): https://github.com/trananhtung1/K5-L23-Day08-2A202603280-TranAnhTung-AI-Assisted-Labeling
- Commit chốt bài: chưa chốt.
- Máy: macOS 15.7.5, Apple M2, RAM 8 GB. Dùng Colab T4 cho r01–r06 và GPU M2 cho r07.
- CVAT (Docker): v2.81.0

## Kết quả trên bộ bẫy theo MSHV

- Trước QC có 620 box trên 24 ảnh. Sau khi loại vùng gần bẫy, giữ 602/620 box, chỉnh 0/620, xóa 18/620 và thêm 2/604 đối tượng. Trên gold, precision đạt 45/74 và recall đạt 45/58.
- Bắt 3/12 bẫy (25%). Học viên gán tay gold mất 22 phút cho 4 ảnh. Script xử lý 24 ảnh trong 2,627 phút trước QC.
- QC lần đầu ghi nhận lỗi trên 2/7 ảnh và 2/162 đối tượng. Sau khi bổ sung hai người bị sót, bản sau QC có 622 đối tượng. Kiểm lại bằng các phép kiểm tự động ghi nhận lỗi trên 0/8 ảnh và 0/202 đối tượng.
- **Rework.** Tỉ lệ bắt bẫy thấp hơn mức 80%. [Báo cáo và nguồn số liệu](submission/REPORT.md)
