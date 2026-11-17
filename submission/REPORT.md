# Báo cáo Day 08 — 2A202603174 - Nguyễn Hoàng Diệu

> Một file duy nhất cho cả bài (gộp recipe, QC và ghi chú cá nhân). Mọi con số đều kèm **mẫu số** và nguồn tệp trong `submission/`.

---

## 1. Tour model & chọn ngưỡng (Part 2–3)

### So sánh các lượt chạy thử nghiệm (nguồn: `submission/model_log.csv`)
* **Detector nền (`yolo26-m`):** Trên 8 ảnh `pilot`, `r01` (`conf=0.25`) phát hiện `213` box với tốc độ `1.01` s/ảnh trên CPU, bắt nhạy `person`, `motorbike`, `car`, `bus` ở tiền cảnh và trung cảnh (`img_015.jpg`, `img_016.jpg`, `img_030.jpg`). Khi tăng ngưỡng chung lên `conf=0.5` ở `r02` (`144` box, `1.16` s/ảnh), mô hình lọc bỏ được `69/213` (`32.4%`) box nhiễu đám đông (`img_030.jpg`) nhưng làm rơi nhiều đối tượng thật ở trung/hậu cảnh có điểm tin cậy `0.27–0.49` (`img_019.jpg`, `img_007.jpg`, `img_015.jpg`).
* **Open-vocabulary detector (`yoloe-s`):** Ở `r03` (`--classes three_wheeler`, `conf=0.25`, `8` box, `0.92` s/ảnh trên CPU), YOLOE bắt trúng tuk-tuk (`three_wheeler 0.92` ở `img_002.jpg`) và `3/4` chiếc xích lô (`three_wheeler 0.87, 0.40, 0.32` ở `img_032.jpg`) mà `yolo26-m` bỏ sót hoàn toàn. Tuy nhiên, YOLOE dễ sinh box trùng đè lên cùng một xe (`three_wheeler 0.43` đè lên `0.92` ở `img_002.jpg`) và nhận nhầm xe đẩy hàng (`img_002.jpg`) hoặc ô tô xa (`img_019.jpg`) thành `three_wheeler`.
* **Segmenter (`sam2.1_b.pt`):** Ở `r04` (`yolo26-m + sam2.1_b.pt`, `13` box / `13` mask trên 2 ảnh `img_004.jpg` và `img_013.jpg`), thời gian chạy tăng lên `8.83` s/ảnh (chậm gấp `8.7` lần so với `yolo26-m` đơn lẻ trên CPU). Vì cấu hình máy chỉ có CPU và RAM giới hạn, chọn `sam2.1_b.pt` chạy cục bộ offline cho từng đối tượng khó ở Part 5 thay vì chạy tự động toàn bộ lô.

### Phân tích ngưỡng theo lớp (nguồn: `submission/qc/threshold_sweep.md`)
* **Chuỗi ngưỡng đã chọn:** `--thr "person=0.4,car=0.15,three_wheeler=0.25"` (các lớp còn lại dùng ngưỡng mặc định `--conf 0.3`), mục tiêu `recall >= 90%` quét trên `4` ảnh gold (`48` đối tượng đánh giá):
  * `person = 0.4`: đạt recall `93%` (`>= 90%`), precision `48%` trên `15` vật gold.
  * `car = 0.15`: đạt recall `100%` (`>= 90%`), precision `40%` trên `8` vật gold.
  * `three_wheeler = 0.25`: đạt recall `100%` (`>= 90%`), precision `62%` trên `5` vật gold.
  * Các lớp `bicycle` (`2` vật gold), `bus` (`1` vật gold), `truck` (`0` vật gold) đều có `< 5` mẫu trong tập gold nên giữ ngưỡng mặc định `0.3` để tránh quá khớp (*overfitting*). Riêng lớp `motorbike` có `17` vật gold nhưng không ngưỡng nào `<= 0.5` đạt mục tiêu recall `90%` (tối đa chỉ đạt `65%`) nên giữ ngưỡng mặc định `0.3`.
* **Lỗi không sửa được bằng đổi `--conf`:** Các lỗi lệch bộ nhãn COCO như tuk-tuk bị `yolo26-m` gọi nhầm thành `truck 0.73` (`img_011.jpg`), `car 0.34–0.35` (`img_004.jpg`), hoặc xích lô bị gọi nhầm thành `bicycle 0.74` (`img_032.jpg`) đều có điểm tin cậy cao hoặc sai bản chất lớp nên chỉnh `--conf` không khắc phục được. Cách xử lý là kết hợp nguồn `yoloe_all` đứng trước `y26_all` khi chạy `merge_prelabels.py` để nhãn `three_wheeler` ghi đè box trùng tại `IoU >= 0.6`.
* **Box điểm cao mà sai / điểm thấp mà đúng:** Trên `img_011.jpg`, hộp `truck 0.73` điểm cao nhưng gán sai lớp cho xe tuk-tuk mép trái; ngược lại trên `img_032.jpg`, hộp `three_wheeler 0.32` điểm thấp nhưng bắt đúng chiếc xích lô bên phải.
* **Giới hạn ước lượng:** Bộ ngưỡng được quét và đánh giá trên chính `4` ảnh gold (*in-sample*), nên các chỉ số precision/recall của máy trên gold mang tính lạc quan và chỉ là ước lượng thô.

---

## 2. AI error profile (nguồn: `submission/qc/review_stats.md`)

Tính trên `24` ảnh lô duyệt (`475` box máy đề xuất, `458` đối tượng thực tế sau duyệt; đã loại `30` đối tượng quanh vùng bẫy để không làm sai lệch chất lượng thật của mô hình):

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 390 | 82.1% | 475 box máy |
| edited (sửa biên) | 17 | 3.6% | 475 box máy |
| deleted (xóa) | 68 | 14.3% | 475 box máy |
| added (vẽ thêm) | 51 | 11.1% | 458 đối tượng sau duyệt |

*(Trong `deleted + added` có `10` trường hợp là đổi lớp tại cùng vị trí, nên số đối tượng máy bỏ sót thật là `51 - 10 = 41` đối tượng.)*

* **Lớp máy làm tệ nhất:** `three_wheeler` và `car` có tỉ lệ xóa bỏ cao nhất (`three_wheeler`: xóa `16/28 = 57.1%` box máy, sửa `5/28 = 17.9%`; `car`: xóa `21/77 = 27.3%` box máy) do hạ ngưỡng tự tin xuống `0.25` và `0.15` ở Part 3 để kéo recall, khiến máy sinh nhiều dương tính giả trên xe đẩy (`img_002.jpg`) hoặc nền đường (`img_019.jpg`). Ngoài ra `motorbike` bị sót nhiều (`added = 13` box, `deleted = 18` box) do trần recall gốc chỉ đạt `65%`.
* **So với nhãn gold (nguồn: `submission/qc/review_stats.md` mục 2):** Trên `4` ảnh gold (`48` đối tượng gold), pre-label đạt **Precision = `56.1%` (`37/66`, KTC 95%: `44–67%`)** và **Recall = `77.1%` (`37/48`, KTC 95%: `63–87%`)**. Đây là số đo lạc quan do chọn ngưỡng trực tiếp trên tập gold và khoảng tin cậy 95% rất rộng.

---

## 3. Thiên kiến tự động hóa và ảnh bẫy

* **Kết quả bắt bẫy (nguồn: `submission/qc/review_stats.md` mục 3):** Bắt được **`6/12 = 50.0%`** (KTC 95%: `25–75%`, thấp hơn ngưỡng đạt `80%`):
  * `extra` (box ma): `3/3` (`100%`) — dễ phát hiện nhất vì khung nằm trên nền trống không có vật thể.
  * `missing` (xóa mất vật): `2/3` (`66.7%`), lọt `img_033.jpg` do chưa thực hiện triệt để bước 1 của checklist (quét toàn bộ ảnh độc lập trước khi nhìn nhãn máy).
  * `wrong_class` (sai lớp): `1/3` (`33.3%`), lọt `img_008.jpg` và `img_028.jpg` do bỏ qua bước kiểm tra kỹ nhãn chữ của từng box ở bước 2.
  * `loose` (khung bao rộng/lỏng): `0/3` (`0%`), lọt cả 3 ảnh `img_005.jpg`, `img_021.jpg` và `img_029.jpg` do bỏ qua bước soi độ khít 4 cạnh biên của các box đã được máy vẽ sẵn.
* **Giải thích hiện tượng `accepted` cao (`82.1%`) nhưng bắt bẫy thấp (`50.0%`):** Tỉ lệ `edited` chỉ đạt `3.6%` (`17/475`) trong khi tốc độ duyệt trung bình chỉ `1.25` phút/ảnh (`30` phút / `24` ảnh) chứng tỏ người duyệt bị chi phối mạnh bởi **hiệu ứng neo (*anchoring*) và thiên kiến tự động hóa (*automation bias*)**: khi thấy box của máy đã bao trùm vật thể, người duyệt mặc định tin tưởng nhãn và biên khung có sẵn thay vì tự kiểm tra lại từng đối tượng.

---

## 4. Máy có đáng dùng không?

* **Hiệu quả thời gian (nguồn: `submission/time_log.csv`):** Gán tay hoàn toàn `4` ảnh gold mất `22.5` phút (`5.62` phút/ảnh), trong khi duyệt `24` ảnh batch có pre-label mất `30` phút (`1.25` phút/ảnh) — nhanh gấp **`4.5` lần** (tiết kiệm `77.8%` thời gian mỗi ảnh).
* **Kết luận vận hành:** Pre-label **đáng dùng** để giảm tải khâu vẽ khung thủ công cho các lớp phổ biến có độ ổn định cao như `person` (`210/217 = 96.8%` box máy được giữ nguyên) và `motorbike` (`108` box giữ nguyên), nhưng **không thể dùng tự động hoàn toàn** mà bắt buộc phải đi kèm checklist duyệt chống *automation bias* (đặc biệt phải kiểm tra lại toàn bộ box ngưỡng thấp của `car` và `three_wheeler` và chỉnh biên khung thay vì mặc định chấp nhận).

---

## 5. Thử nghiệm SAM & tách mask (Part 5)

Sử dụng `scripts/sam2_points.py` với mô hình **`sam2.1_b.pt`** trên `3` ca khó (mỗi ảnh dùng `2` điểm dương `--pos` và `2` điểm âm `--neg`, lưu tại `submission/masks/`):

1. **`mask_03_img015_bicycle.jpg` (`img_015.jpg` — Xe đạp có người lái và giỏ trước):** Nhờ `2` click âm đặt lên áo khoác và chân người đạp, SAM 2.1 tách rời hoàn toàn khung xe, yên và hai bánh xe khỏi người lái (`person` không bị dính mask), đồng thời giữ lại giỏ xe gắn cố định phía trước đúng theo quy tắc phụ kiện của guideline mục 4.2.
2. **`mask_02_img004_tuktuk.jpg` (`img_004.jpg` — Xe tuk-tuk tiền cảnh đứng cạnh người đi bộ):** Toàn bộ thân xe, mái che và bánh xe tuk-tuk được tách gọn khỏi người đàn ông đứng sát bên phải và chiếc tuk-tuk xanh phía sau; tuy nhiên biển hiệu chữ `TAXI` màu trắng gắn trên nóc bị đứt tách khỏi vùng mask chính do cổ nối mảnh.
3. **`mask_01_img013_tuktuk.jpg` (`img_013.jpg` — Xe tuk-tuk khoang mở lớn có tài xế ngồi bên trong):** Khắc phục được việc `yolo26-m` bỏ sót hoàn toàn chiếc tuk-tuk này ở Part 2 và loại bỏ được hành khách ngồi phía sau; tuy nhiên `1` click âm trên lưng tài xế không đủ chặn mask tràn vào toàn bộ người lái trong khoang và lem sang bánh trước chiếc xe máy đỗ sát mũi xe.
* **Đối chiếu guideline mục 4.2:** Quy tắc tách người lái khỏi phương tiện và gộp phụ kiện gắn cố định giúp xác định rõ biên đúng/sai của mask, nhưng guideline v1.0 còn thiếu hướng dẫn thao tác đặt nhiều điểm âm cho phương tiện khoang mở (tuk-tuk, xe máy) — đã được bổ sung vào `v1.1` (`CAL-04`).
* *(Tùy chọn)* **So sánh với Interactor trong CVAT:** Trên CVAT Docker bản cục bộ không bật sẵn container dịch vụ SAM tự động nên sử dụng công cụ vẽ polygon thủ công; chạy `sam2_points.py` offline giúp tận dụng trực tiếp `sam2.1_b.pt` trên CPU nhưng đòi hỏi nhập tọa độ pixel thủ công nên mất nhiều lượt thử điểm hơn thao tác click trực tiếp trên giao diện.

---

## 6. Calibration (Part 6)

* **Số quyết định đã chốt (nguồn: `submission/qc/calibration_decisions.csv`):** **`4` quyết định** (`CAL-01` đến `CAL-04`), gồm `2` ca từ câu hỏi mở (`cau_hoi_mo`) và `2` ca từ lỗi hệ thống của mô hình (`loi_may`).
* **Quyết định quan trọng nhất (`CAL-03` & `CAL-04`):** `CAL-03` (`img_002.jpg`, `img_011.jpg`, `img_032.jpg`) chốt quy tắc hợp nhất `three_wheeler` giữa YOLOE và YOLO26 (xóa box trùng độ tin cậy thấp của YOLOE và đổi nhãn box `car`/`truck`/`bicycle` sai của YOLO26); `CAL-04` (`img_013.jpg`, `img_004.jpg`) quy định đặt tối thiểu `2` click âm lên người ngồi trong khoang lái và `1` click âm tại ranh giới xe kế cận khi tách mask SAM.
* **Quy tắc mới đã cập nhật vào `guideline_v1.1_calibrated.md`:** Bổ sung quy tắc phân loại xe đạp điện và gộp giỏ xe gắn cố định (`CAL-01`, `img_015.jpg`), ngưỡng lọc vật thể `< 20px` và điều kiện khoanh `ignore_region` khi có từ `3` vật `< 20px` đứng cụm (`CAL-02`, `img_018.jpg`), cùng quy trình xử lý `three_wheeler` (`CAL-03`) và điểm âm SAM (`CAL-04`).
* **Nguyên nhân gốc:** Kết hợp giữa **guideline v1.0 còn câu hỏi mở** (xe đạp điện, phụ kiện gắn liền, ngưỡng kích thước hậu cảnh) và **giới hạn kiến trúc mô hình** (bộ nhãn COCO của YOLO26 thiếu `three_wheeler`, SAM tràn biên ở phương tiện khoang mở).

---

## 7. QC và quyết định (Part 6)

* **Mẫu kiểm định (nguồn: `submission/qc/qc_sample.csv` & `submission/qc/qc_review.csv`):** Rút mẫu **`5/24` ảnh (`20.8%` lô)** gồm `0` ảnh `can_review` và `5` ảnh ngẫu nhiên (`seed 42`: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`), tổng cộng **`95/458` đối tượng (`20.7%` lô)**.
* **Kết quả soi lỗi trên 5 ảnh mẫu:** Ghi nhận **`0/5` ảnh lỗi (`0.0%`)** và **`0/95` đối tượng lỗi (`0.0%`)** ở cả 6 nhóm lỗi (bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask); cả 5 ảnh mẫu đều đạt tiêu chuẩn hiển thị khung bao và phân lớp.
* **Giới hạn mẫu nhỏ & điểm mù của script tự động:** Với `0/5` ảnh lỗi, cận trên khoảng tin cậy 95% (Wilson) vẫn lên tới **`43.4%` theo ảnh** (`3.9%` theo đối tượng) do cỡ mẫu nhỏ (`5` ảnh) không bốc trúng `6` ảnh bẫy đã lọt ở Part 4 (`img_005.jpg`, `img_008.jpg`, `img_021.jpg`, `img_028.jpg`, `img_029.jpg`, `img_033.jpg`). Ngoài ra, script `qc_report.py` chỉ tổng hợp theo phiếu `qc_review.csv` do người kiểm điền nên **không tự kiểm tra được** độ ôm sát 4 cạnh biên khung hay lỗi cùng bỏ sót với máy nếu người kiểm không đối chiếu kỹ ảnh gốc.

**Quyết định: REWORK**, ngày **2026-11-17** (khớp gợi ý trong `submission/qc/qc_report_auto.md`).
* **Lý do (theo luồng Hold → Rework → Release):** Không mắc điều kiện `Hold` (guideline đã chốt đầy đủ ở `v1.1`, không có ảnh hỏng dữ liệu), nhưng tỉ lệ bắt bẫy ở Part 4 chỉ đạt **`50.0%` (`6/12 < 80%`)** và trong lô còn tồn dư `6` lỗi bẫy đã lọt (đặc biệt là `3/3` lỗi `loose` và `2/3` lỗi `wrong_class`), nên chưa đủ độ tin cậy để `Release` ngay.
* **Việc cần làm tiếp:** Thực hiện rà soát lại toàn bộ `24` ảnh batch theo checklist chống *automation bias* (bắt buộc kiểm tra độ khít 4 cạnh biên và nhãn từng box được máy giữ nguyên), chỉnh sửa các box biên lỏng và sai lớp, sau đó chạy lại `build_release.py` và rút mẫu QC vòng 2 trước khi phát hành chính thức.

---

## 8. Ghi chú cá nhân

* **Dùng AI để làm bài:** Có sử dụng trợ lý AI (Gemini) để hỗ trợ đối chiếu lỗi trực quan trên các ảnh đầu ra ở Part 2–5, giải thích cú pháp tham số các script trong `scripts/`, và chuẩn hóa cấu trúc trình bày cho `model_log.csv`, `calibration_decisions.csv`, `guideline_v1.1_calibrated.md` và `REPORT.md`. Toàn bộ số liệu định lượng (`213/144/8/13` box, chuỗi `--thr`, `390/17/68/51`, `6/12` bẫy, `22.5` phút gold / `30` phút batch) đều được chạy trực tiếp trên máy cá nhân và đối chiếu lại với các tệp log trong `submission/`.
* **Gold có bị nhiễm không:** **Không.** Toàn bộ `4` ảnh gold (`img_001.jpg`, `img_012.jpg`, `img_024.jpg`, `img_036.jpg`) được gán tay từ đầu trên CVAT trước khi chạy pre-label hoặc dùng AI hỗ trợ.
* **Môi trường máy / GPU / CVAT & vướng mắc cài đặt:** Chạy trên Windows 11, CPU 8 nhân (không dùng GPU rời), môi trường ảo Python `.venv` và CVAT Docker cục bộ (`http://localhost:8080`). Vướng mắc gặp phải: khi tạo Task `batch-review` trên CVAT ban đầu quên gán Task vào Project `day08-2A202603174` nên gặp lỗi `Label 'person' is not registered for this task` khi import `pre_traps_import.coco.json` — đã khắc phục bằng cách chuyển Task vào đúng Project để kế thừa đủ 8 nhãn.
* **Dùng cache `data/cache/` thay vì tự chạy:** **Không.** Toàn bộ các lượt chạy `yolo26-m`, `yoloe-s` và `sam2.1_b.pt` đều được thực thi trực tiếp trên CPU của máy cá nhân.
* **Lỗi / bug script đã gặp:** Một số tên cờ trong hướng dẫn tổng quát hơi khác với phiên bản script thực tế trong repo (`merge_prelabels.py` nhận danh sách tệp vị trí thay vì `--sources`; `inject_traps.py` dùng `--pre` và tự sinh tệp `*_import.coco.json` không cần `--import-out`; `sam2_points.py` tách riêng `--out` và `--name`). Đã kiểm tra bằng `--help` và chạy thành công toàn bộ.
* **Một điều làm khác lần sau:** Khi duyệt pre-label, luôn tắt hoặc ẩn tạm lớp nhãn máy trong vài giây đầu để tự quét toàn ảnh trước, và kiểm tra 4 cạnh biên của mọi box được máy đề xuất thay vì chỉ tập trung xóa box ma hay vẽ thêm vật sót.