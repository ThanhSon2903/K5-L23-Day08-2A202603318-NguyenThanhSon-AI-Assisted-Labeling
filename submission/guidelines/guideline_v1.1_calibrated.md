# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603318 - Nguyễn Thanh Sơn

> Chỉ ghi nhận các thay đổi so với v1.0. Các quy tắc chưa đủ căn cứ sẽ được đánh dấu chưa chốt để tiếp tục đối chiếu với giảng viên.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện: bicycle hay motorbike? | Chưa chốt. Cần đối chiếu định nghĩa và ví dụ trong guideline v1.0. Không phân loại chỉ dựa vào hình dáng hoặc dự đoán của model. | img_006 | CAL-002 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Xe ba bánh chở hàng có người ngồi trên xe, ranh giới giữa người và phương tiện khá gần nhau. | YOLO26m dự đoán các box person và three_wheeler. | Giữ một box cho toàn bộ phương tiện và annotation riêng cho từng người nhìn thấy. | Mục Bounding Box và Class: Không gộp người vào box phương tiện; mỗi object có annotation riêng. |
| img_006 | Ảnh có mật độ người và xe máy cao, các box gần nhau và có thể chồng lấn. | YOLO26m dự đoán nhiều box person và motorbike. | Kiểm tra từng object, xóa box trùng và giữ annotation riêng cho từng người, từng xe. | Mục Review: Không xóa box chỉ vì chồng lấn; cần xác định chúng có thuộc cùng một object hay không. |
| img_013 | Xe tuk-tuk có nhiều bộ phận và khoang chở khách, dễ lấy thừa vùng nền khi vẽ box. | YOLO26m/YOLOE tạo pre-label cho phương tiện three_wheeler. | Gán một box cho toàn bộ phương tiện, không tách các bộ phận của cùng một xe thành nhiều object. | Mục Bounding Box: Box phương tiện phải bao phủ toàn bộ phần phương tiện nhìn thấy, hạn chế lấy thừa nền. |
| img_014 | Nhiều người và xe máy xuất hiện gần nhau, một số object bị che khuất. | Model dự đoán các box person và motorbike; có trường hợp cần kiểm tra lại class. | Kiểm tra đặc điểm từng object trước khi xác nhận class; không đoán class khi không đủ thông tin. | Mục Occlusion và Class: Chỉ gán class khi có đủ căn cứ nhận diện; object bị che khuất cần được xử lý nhất quán theo guideline. |
| img_022 | Xe máy nằm sát mép ảnh và chỉ nhìn thấy một phần. | YOLO26m dự đoán box motorbike cho object ở rìa ảnh. | Chỉ annotation phần object thực sự nhìn thấy trong ảnh nếu đủ căn cứ xác định class. | Mục Bounding Box: Object bị cắt bởi biên ảnh chỉ được bao phủ trong phạm vi ảnh; không tự mở rộng box ra ngoài ảnh. |
| img_028 | Nhiều xe máy và người đứng sát nhau, các box dễ chồng lấn hoặc trùng object. | YOLO26m dự đoán nhiều box motorbike và person. | Đối chiếu từng box với ảnh gốc; mỗi phương tiện và mỗi người là một object riêng, xóa box duplicate. | Mục Review và Duplicate: Kiểm tra object trùng trước khi chấp nhận; không gộp các phương tiện độc lập. |
| img_034 | Một phương tiện được model dự đoán là truck trong cảnh có nhiều xe máy. | YOLO26m dự đoán box truck; class cần được xác minh bằng ảnh gốc. | Chưa xác nhận class chỉ dựa trên kết quả model; cần kiểm tra hình dáng và đặc điểm phương tiện trước khi sửa hoặc chấp nhận. | Mục Class: Không mặc định kết quả dự đoán của model là đúng; trường hợp chưa đủ căn cứ cần đưa đi review. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Bổ sung quy tắc tách annotation giữa người và phương tiện; kiểm tra box chồng lấn và duplicate; thống nhất phạm vi box cho xe ba bánh; xử lý object bị che khuất và object nằm sát biên ảnh; yêu cầu xác minh class thay vì phụ thuộc hoàn toàn vào model. | CAL-001 đến CAL-007, tương ứng img_003, img_006, img_013, img_014, img_022, img_028 và img_034. Câu hỏi phân loại xe đạp điện tại CAL-002 vẫn chưa chốt. |