# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 475; đối tượng sau duyệt: 458; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 390 | 390/475 = 82.1% | 475 box máy |
| edited (sửa) | 17 | 17/475 = 3.6% | 475 box máy |
| deleted (xoá) | 68 | 68/475 = 14.3% | 475 box máy |
| added (vẽ thêm) | 51 | 51/458 = 11.1% | 458 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 41.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 210 | 1 | 6 | 14 |
| bicycle | 5 | 0 | 4 | 3 |
| motorbike | 108 | 11 | 18 | 13 |
| car | 56 | 0 | 21 | 10 |
| bus | 3 | 0 | 1 | 1 |
| truck | 1 | 0 | 2 | 3 |
| three_wheeler | 7 | 5 | 16 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 29 | 11 | 56.1% (KTC 95%: 44–67%) | 77.1% (KTC 95%: 63–87%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - extra: 3/3
  - loose: 0/3
  - missing: 2/3
  - lọt: img_028.jpg (wrong_class), img_005.jpg (loose), img_021.jpg (loose), img_029.jpg (loose), img_033.jpg (missing), img_008.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
