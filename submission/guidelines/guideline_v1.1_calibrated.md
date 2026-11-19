# Bổ sung guideline v1.1 — 2A202603280 - Trần Anh Tùng

Bản bổ sung dựa trên ba ca đã ghi nhận. Hai ca đầu dùng nhận xét khi xem ảnh. Ca thứ ba đối chiếu dự đoán với gold. Các quyết định tương ứng nằm trong [bảng calibration](../qc/calibration_decisions.csv).

## 1. Điểm tin cậy khi duyệt

**CAL-03 · img_036 · mục 6, câu 7.** Chiếc xe máy có điểm 0,1165 và IoU 0,9715 với box gold. Ngưỡng 0,10 giữ chiếc xe trong danh sách duyệt. Khi xét giữ box, kiểm tra lớp và bốn cạnh trên ảnh. Sáu câu hỏi còn lại của mục 6 vẫn mở vì ba ca dưới đây chưa đủ để chốt chúng.

## 2. Xe đẩy hàng và người trên tuk-tuk

| Ca | Dấu hiệu trong ảnh | Cách xử lý |
|---|---|---|
| CAL-01 · img_002 | YOLOE đặt box xe ba bánh lên xe đẩy của người bán hàng. | Loại đề xuất cho chiếc xe đẩy. Với phương tiện chở hàng, đối chiếu hình dạng với các lớp xích lô và ba gác trước khi chọn nhãn. |
| CAL-02 · img_013 | Tuk-tuk bị gọi là ô tô. Box chỉ phủ một phần xe, và box người trên xe cũng thiếu. | Chọn `three_wheeler` cho xe. Kiểm tra riêng phần người nhìn thấy và giữ box `person` riêng. Sửa box trước khi dùng làm đầu vào cho SAM. |

## 3. Lịch sử thay đổi

| Phiên bản | Nội dung |
|---|---|
| v1.0 | Quy tắc gốc về lớp, box, mask và vùng bỏ qua. |
| v1.1 (lượt duyệt tự động) | Bổ sung cách xử lý xe đẩy, người trên tuk-tuk và box xe máy có điểm thấp. |
