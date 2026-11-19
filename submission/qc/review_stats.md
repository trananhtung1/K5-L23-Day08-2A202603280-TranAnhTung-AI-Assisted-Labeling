# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 620; đối tượng sau duyệt: 604; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 602 | 602/620 = 97.1% | 620 box máy |
| edited (sửa) | 0 | 0/620 = 0.0% | 620 box máy |
| deleted (xoá) | 18 | 18/620 = 2.9% | 620 box máy |
| added (vẽ thêm) | 2 | 2/604 = 0.3% | 604 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 1.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 266 | 0 | 2 | 1 |
| bicycle | 7 | 0 | 1 | 0 |
| motorbike | 229 | 0 | 11 | 0 |
| car | 51 | 0 | 3 | 0 |
| bus | 3 | 0 | 0 | 1 |
| truck | 5 | 0 | 1 | 0 |
| three_wheeler | 41 | 0 | 0 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 58 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 45 | 29 | 13 | 60.8% (KTC 95%: 49–71%) | 77.6% (KTC 95%: 65–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 3/12 = 25.0% (KTC 95%: 9–53%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - extra: 0/3
  - loose: 0/3
  - missing: 2/3
  - lọt: img_017.jpg (extra), img_029.jpg (loose), img_018.jpg (wrong_class), img_021.jpg (extra), img_022.jpg (loose), img_033.jpg (wrong_class), img_027.jpg (extra), img_034.jpg (missing), img_003.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
