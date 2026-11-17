# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603174 - Nguyễn Hoàng Diệu

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy / lỗi khi duyệt. Mỗi ca khó **có ảnh ví dụ** (`img_0xx.jpg`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện có bàn đạp xếp vào `bicycle` hay `motorbike`? Phụ kiện giỏ xe xử lý thế nào? | Xe hai bánh có bàn đạp nhưng trang bị động cơ điện/bình ắc-quy xếp vào `motorbike`; xe đạp cơ có giỏ gắn cố định phía trước thì gộp giỏ vào khung/mask xe và luôn tách riêng người lái (`person`). | `img_015.jpg` | `CAL-01` |
| 2 | Vật thể quá nhỏ ở hậu cảnh (`< 20px`) có gán nhãn lẻ không? | Không vẽ box đơn lẻ cho vật thể có chiều cao `< 20px` (như các box `13.7–18.0px` ở hậu cảnh); chỉ khoanh 1 khung `ignore_region` khi có từ 3 vật `< 20px` đứng thành cụm sát nhau. | `img_018.jpg` | `CAL-02` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002.jpg` (`img_011.jpg`, `img_032.jpg`) | Xe tuk-tuk và xích lô bị nhận nhầm hoặc vẽ trùng khung | `yolo26-m` bỏ sót tuk-tuk (`img_002.jpg`) hoặc gọi nhầm thành `truck` (`img_011.jpg`), `bicycle` (`img_032.jpg`); `yoloe-s` bắt được nhưng vẽ 2 box trùng đè lên nhau (`0.92` và `0.43`) và nhận nhầm xe đẩy hàng (`0.34`). | Tất cả xe cơ giới/thô sơ 3 bánh chở khách hoặc hàng đều gán `three_wheeler`; người duyệt bắt buộc xóa box trùng độ tin cậy thấp của YOLOE và đổi nhãn các box `car`/`truck`/`bicycle` bị YOLO26 gán nhầm. | Bổ sung vào **Mục 2.5** (định nghĩa lớp `three_wheeler`) và **Mục 5.1** (checklist duyệt pre-label). |
| `img_013.jpg` (`img_004.jpg`) | Tách mask phương tiện khoang mở có tài xế ngồi bên trong và xe đỗ sát cạnh | `sam2.1_b.pt` khi chỉ dùng 1 click âm trên lưng tài xế bị tràn mask sang toàn bộ người lái trong khoang và lem sang bánh xe máy đỗ sát mũi xe (`img_013.jpg`), hoặc đứt rời biển hiệu `TAXI` trên nóc (`img_004.jpg`). | Đặt tối thiểu 2 click âm (`--neg`) lên đầu/thân người ngồi trong khoang lái và 1 click âm tại ranh giới tiếp xúc với xe kế cận để đảm bảo mask phương tiện không nuốt `person`. | Bổ sung vào **Mục 4.2** (quy tắc biên segmentation và đặt điểm âm SAM). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân loại xe đạp điện, phụ kiện gắn liền, ngưỡng lọc `< 20px`, quy tắc gộp/xóa trùng `three_wheeler` giữa YOLO26–YOLOE và quy tắc đặt điểm âm SAM cho xe khoang mở. | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` |