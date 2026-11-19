# Báo cáo Day 08 — 2A202603280 - Trần Anh Tùng

> Bộ gold gồm 4 ảnh gán tay. Batch gồm 24 ảnh được rà soát bằng thị giác máy và qua hai vòng QC.

## 1. Model và recipe (Part 2–3)

| Lớp | Model | Ngưỡng | Kết quả trên 4 ảnh gold |
|---|---|---|---|
| `motorbike` | YOLO26m (r05) | 0,10 | Recall 11/12 (92%), precision 11/20 (55%). |
| `three_wheeler` | YOLOE-26s (r06) | 0,15 | Recall 7/7 (100%), precision 7/9 (78%). Prompt lấy từ taxonomy gồm `tuk-tuk`, `cyclo`, `cargo tricycle`. |
| `person`, `car` | YOLO26m (r05) | 0,30 | Recall tại ngưỡng này lần lượt là 18/27 và 7/8. Trong dải đã quét, giá trị cao nhất là 19/27 và 7/8, đều dưới mục tiêu 90%. |
| `bicycle`, `bus`, `truck` | YOLO26m (r05) | 0,30 | Gold lần lượt có 3, 1 và 0 vật, dưới mức 5 vật cần để chọn ngưỡng riêng. |

Các tham số và số đo lấy từ [model_log.csv](model_log.csv) và [threshold_sweep.csv](qc/threshold_sweep.csv). Ngưỡng được chọn và chấm trên cùng 4 ảnh gold nên precision và recall có thể lạc quan khi áp dụng cho ảnh mới.

Lệnh sau tái tạo đúng 726 box trên 24 ảnh batch và 4 ảnh gold. Khi hai nguồn có box chồng nhau với IoU từ 0,6, bước gộp ưu tiên đề xuất của YOLOE.

```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json \
    --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" \
    --out work/prelabel/merged_review/pre.coco.json
```

- Ở `img_013`, YOLO26 gọi tuk-tuk là `car` và box chỉ ôm một phần xe. SAM 2.1 (`sam2.1_b.pt`, r04) lấy box đó làm đầu vào nên mask cũng thiếu. Box người trên xe cũng ôm thiếu. Đổi ngưỡng chỉ giữ hoặc loại box hiện có. Ca này cần kiểm tra lại cả lớp và đường bao.
- Ở `img_002`, YOLOE gán xe đẩy hàng rong thành `three_wheeler`. Lần chạy r03 chỉ yêu cầu lớp này. Khi xem ảnh còn có ô tô chỉ lộ mui và người đi xe máy gần camera cần kiểm tra ở kết quả detector nhiều lớp.
- Ở `img_036`, dự đoán `motorbike` số 1650 của r05 có điểm 0,1165 và khớp nhãn gold số 58 với IoU 0,9715. Ngưỡng 0,10 giữ được chiếc xe này. Nguồn đối chiếu là `work/prelabel/y26_all/pre.coco.json` và [gold_manual.json](labels/gold_manual.json).

## 2. Các chỉnh sửa và kết quả trên gold

Lượt này dùng bộ bẫy tạo riêng bằng MSHV `2A202603280`. Có 634 box trên 24 ảnh lúc nhập và 620 box khi xuất. Script loại các vùng gần bẫy khi tính bảng dưới đây. Vì vậy mẫu số thống kê nhỏ hơn số box trong file COCO. [Thống kê toàn batch](qc/review_stats.md)

| Thao tác | Số lượng | Tỉ lệ và mẫu số |
|---|---:|---|
| Giữ nguyên | 602 | 602/620 box máy (97,1%) |
| Chỉnh vị trí hoặc kích thước | 0 | 0/620 box máy |
| Xóa | 18 | 18/620 box máy (2,9%) |
| Thêm | 2 | 2/604 đối tượng sau duyệt (0,3%) |

Khi đối chiếu toàn bộ nhãn, script loại 16 box thấp hơn 20 px và một box trùng, thêm 3 box, sửa lớp của 2 box. Ở `img_013`, hai box gần như trùng nhau cho mask có IoU 0,999. Ở `img_031`, Apple Vision và YOLO26n-seg đều phát hiện một người chưa có box. [Nhật ký sửa theo ảnh](qc/batch_ai_changes.csv)

Trên bốn ảnh gold, máy khớp đúng 45 vật, thừa 29 dự đoán và bỏ sót 13 vật. Precision đạt **45/74 = 60,8%**, recall đạt **45/58 = 77,6%**. Trong 64 annotation gán tay có 6 vùng bỏ qua. Khoảng tin cậy Wilson 95% lần lượt là 49–71% và 65–86%. [Số liệu gold](qc/review_stats.md)

## 3. Thiên kiến tự động hóa và ảnh bẫy

Lượt duyệt tự động bắt **3/12 bẫy (25%)**. Cụ thể, tìm lại được 2/3 vật bị xóa và sửa đúng 1/3 ca sai lớp. Cả 3 box thừa và 3 box lệch đều lọt. Ngưỡng của lab là 80%, tương đương ít nhất 10/12 bẫy.

Tỉ lệ giữ nguyên 97,1% cần đọc cùng điểm bẫy. Quy trình cần thêm bằng chứng từ detector hoặc mask mới sửa box. Một box thừa có thể đứng yên qua cả lượt nếu các mô hình không đưa ra bằng chứng đủ mạnh để xóa. Cách làm này tiện để sàng lọc, nhưng chưa đủ để giao nhãn cuối. [Kết quả chấm bẫy](qc/review_stats.md)

## 4. Thời gian và cách duyệt

Gán tay 4 ảnh gold mất 22 phút, trung bình 5,5 phút/ảnh. Script xử lý 24 ảnh trong 2,627 phút, tương đương 6,6 giây/ảnh trước QC. Nhật ký chưa có thời gian một người duyệt đủ 24 ảnh để tính mức tiết kiệm khi dùng pre-label. [Nhật ký](time_log.csv) · [Mốc đo và mã kiểm tra bản export](qc/ai_review_run.json)

Ba thao tác trong checklist phù hợp với các lỗi đã gặp:

1. Mỗi ảnh đều bắt đầu bằng một lượt ẩn box và quét từ tiền cảnh ra hậu cảnh. Ở `img_011`, cần nhìn kỹ chỗ người đứng sát phương tiện.
2. Khi hiện box, kiểm từng người và xe riêng. Với `img_013`, kiểm lớp xe cùng phần thân xe và người nằm ngoài box.
3. Với cụm khó tách, kiểm chiều cao trên ảnh gốc. Chỉ khoanh `ignore_region` sau khi xác nhận cụm đáp ứng quy tắc của guideline.

## 5. Mask SAM cho ba vật

Lần chạy Colab r04 dùng SAM 2.1 Base với box. Ba kết quả dưới đây dùng **SAM 2.1 Tiny** trên máy M2, qua `scripts/sam2_points.py`. Ba vật được chọn gồm một xe máy, một tuk-tuk và một người. Script lấy điểm gợi ý từ polygon của r07. Mỗi lượt dùng một box, hai điểm dương và một điểm âm. Tổng cộng 9/9 điểm nằm đúng phía yêu cầu của mask đầu ra.

| Ảnh và vật | Điểm dương · điểm âm (x, y) | Kết quả | Kiểm tra đường biên |
|---|---|---|---|
| [img_003 · xe máy bên phải](masks/img_003_motorbike.jpg) | (952,308), (984,383) · (884,292) | 15.190 px, 2 polygon | Có 12 px giao với các mask người của r07. Cần nhìn kỹ chỗ người ngồi sát xe. |
| [img_004 · xe trước cụm tuk-tuk](masks/img_004_tuktuk.jpg) | (672,382), (528,464) · (587,552) | 96.147 px, 1 polygon | Có 329 px giao với mask người của r07. Hai mô hình cũng bất đồng về lớp xe tại vị trí này. |
| [img_013 · người bên phải xe](masks/img_013_person.jpg) | (678,290), (747,387) · (658,467) | 31.542 px, 2 polygon | Không có pixel giao với mask của **người khác** trong r07. Biên tiếp giáp giữa người và xe vẫn cần kiểm trên ảnh. |

IoU giữa mask SAM và polygon r07 lần lượt là 0,834, 0,951 và 0,874. Hai kết quả dùng chung vùng gợi ý, nên mức trùng nhau chỉ giúp định vị chỗ cần soi. Guideline mục 4.2 yêu cầu giữ phần vật nhìn thấy, tách người khỏi xe ở chỗ tiếp xúc và bỏ bóng đổ. Chưa có mask gold để tính lỗi đường biên hay kết luận phần hàng hóa đã được tách đúng. [Tọa độ box, điểm và kết quả từng lượt](masks/sam_prompts.json)

## 6. Calibration

Guideline v1.1 ghi lại cách xử lý ba ca. CAL-01 xử lý xe đẩy hàng rong bị nhận nhầm thành xe ba bánh. CAL-02 ghi riêng lỗi lớp và lỗi đường bao của tuk-tuk ở `img_013`, vì sửa lớp vẫn để lại một box thiếu thân xe. CAL-03 đối chiếu chiếc xe máy điểm 0,1165 ở `img_036` để làm rõ cách dùng điểm tin cậy trong câu hỏi mở số 7. Sáu câu hỏi khác của guideline v1.0 cần thêm ca để chốt. [Bảng calibration](qc/calibration_decisions.csv) · [Phần bổ sung v1.1](guidelines/guideline_v1.1_calibrated.md)

## 7. QC và quyết định

**Vòng 1.** Mẫu theo seed 42 gồm 2 ảnh khó và 5 ảnh ngẫu nhiên, tổng cộng 162 đối tượng. Phép kiểm tư thế người tìm được hai vùng trước đó chưa có box `person`: một ở `img_008`, một ở `img_034`. Tỉ lệ lỗi theo ảnh là **2/7 = 28,6%**. Theo quy ước mẫu số của script, tỉ lệ theo đối tượng là **2/162 = 1,2%**. Cận trên Wilson 95% theo ảnh là 64%. [Phiếu vòng 1](qc/round1/qc_review.csv) · [Báo cáo vòng 1](qc/round1/qc_report_auto.md)

**Sau sửa.** Đã thêm hai box người bao vùng khớp và đánh dấu `can_review` để lưu ý về đường biên. Bản sau QC có **622 đối tượng trên 24 ảnh**. Lấy mẫu lại cùng seed cho 3 ảnh khó và 5 ảnh ngẫu nhiên, tổng số 202 đối tượng. Qua các phép kiểm tọa độ, trùng mask và độ phủ vùng khớp, còn **0/8 ảnh và 0/202 đối tượng** ghi nhận lỗi. Cận trên Wilson 95% theo ảnh vẫn là **32%**, cao hơn ngưỡng 2%. Mẫu có ảnh lặp từ vòng đầu, nên kết quả này còn hạn chế khi suy rộng. [Hai ảnh đã sửa](labels/qc_fixed.json) · [Bản sau QC](release/final_det.coco.json) · [Kết quả kiểm lại](qc/qc_report_auto.md)

**Quyết định ngày 19/11/2026: Rework.** Tỉ lệ bắt bẫy 25% còn thấp. Cần kiểm lại các box thừa và box lỏng trước khi phát hành. Hai lỗi sửa ở QC đã cải thiện bộ nhãn. Phép kiểm tự động vẫn có thể bỏ qua một vật mà mọi detector đều bỏ sót.

## 8. Công cụ và nguồn dữ liệu

- Học viên đã gán tay và export bộ gold trước khi chạy model theo nhật ký và bàn giao. Các ngưỡng được chọn sau đó trên cùng bộ gold này.
- Sáu lượt đầu chạy trên Google Colab T4. Lượt r07 dùng GPU Apple M2. Ba mask điểm dùng SAM 2.1 Tiny trên CPU. Đầu vào quét ngưỡng là kết quả Colab r05, r06.
- Job 21 trong CVAT chứa bộ bẫy có mã học viên khác README. Bộ kết quả trong bản này bắt đầu bằng một lần chạy mới của `inject_traps.py --student 2A202603280`, rồi rà ngoại tuyến trên dữ liệu COCO. Mọi nhãn batch và số đo của lượt này gắn với bộ bẫy mới. Job CVAT cũ vẫn lưu riêng.
- Codex hỗ trợ viết và chạy script đối chiếu nhãn, tổng hợp số liệu và biên tập báo cáo. Script dùng Apple Vision, YOLO26n-seg và SAM để rà batch và kiểm tra mẫu QC. Phiếu QC, nhật ký thời gian và mã SHA-256 của bản export trước khi chấm bẫy được lưu kèm.
