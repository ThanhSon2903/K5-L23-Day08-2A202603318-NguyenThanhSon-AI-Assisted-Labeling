# Báo cáo Day 08 — 2A202603318 - Nguyễn Thanh Sơn

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person| YOLO26m | 0.3(mặc định)| r01, r02 thử nghiệm hai mức confidence 0.25 và 0.5. Qua threshold sweep, không có ngưỡng ≤ 0.5 đạt recall mục tiêu 90%. |
| bicycle | YOLO26m| 0.3 (mặc định) | Số đối tượng gold dưới 5, chưa đủ dữ liệu để kết luận ngưỡng riêng. |
| motorbike | YOLO26m| 0.25  | Threshold sweep trên gold cho recall 100%, precision 85%, với 11 đối tượng gold. |
| car | YOLO26m| 0.3 (mặc định) | Không có ngưỡng ≤ 0.5 đạt recall mục tiêu 90%. |
| bus | YOLO26m| 0.3 (mặc định) | Chỉ có 1 đối tượng gold, không đủ mẫu để chọn ngưỡng riêng. |
| truck | YOLO26m| 0.3 (mặc định) | Không có đối tượng truck trong tập gold dùng để sweep.|
| three_wheeler | YOLOE-26s, prompt: three_wheeler| 0.15|Threshold sweep cho recall 100%, precision 78%, với 7 đối tượng gold.|

Các run trong model_log.csv:
- r01: YOLO26m, confidence 0.25, 232 boxes trên 8 ảnh, thời gian 1.29 giây.

- r02: YOLO26m, confidence 0.5, 140 boxes trên 8 ảnh, thời gian 1.13 giây.

- r03: YOLOE-26s, prompt three_wheeler, 9 boxes, thời gian 0.45 giây.

- r04: YOLO26m kết hợp SAM 2.1, tạo 17 masks trên 2 ảnh, thời gian trung bình 8.53 giây/ảnh khi chạy CPU.

Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`

**Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: 
Đã thử SAM 2.1 (sam2.1_b.pt) và MobileSAM (mobile_sam.pt, thuộc dòng SAM 1).
- SAM 2.1 tạo mask trên ảnh img_013.jpg với diện tích 317.895 pixel, tương đương 45,5% ảnh và tạo 1 polygon.

- MobileSAM tạo mask trên cùng ảnh với diện tích 246.370 pixel, tương đương 35,3% ảnh và tạo 2 polygon.

- SAM 2.1 được tiếp tục thử trên img_004.jpg và img_017.jpg.
Em tiếp tục sử dụng SAM 2.1 cho hai ảnh sau vì đã có kết quả thử nghiệm trực tiếp. Chưa có phép đo đối chiếu chất lượng mask đầy đủ để kết luận SAM 2.1 tốt hơn MobileSAM trong mọi trường hợp.

Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? 
- Một số lỗi như tuk-tuk bị nhận diện nhầm thành truck/car là lỗi phân loại. Giảm hoặc tăng confidence không đảm bảo sửa được nhãn sai. Khi review, tôi kiểm tra đặc điểm vật thể và sửa class thủ công trên CVAT.

Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
- Ảnh img_0.34.jpg: Cụ thể là qua mắt là xe tải nhưng mà AI gán nhãn lại là xe three_wheeler và điểm conf 0.85

Giới hạn của threshold sweep:
- Threshold được lựa chọn dựa trên chính 4 ảnh gold nên kết quả đánh giá có thể lạc quan hơn thực tế. Số lượng ảnh và đối tượng gold còn ít, đặc biệt ở các lớp bicycle, bus và truck. Vì vậy, các chỉ số chỉ nên xem là ước lượng ban đầu, không phải kết luận tổng quát về chất lượng model.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 347 | 347/446 = 77.8% | 446 box máy |
| edited (sửa) | 34 | 34/446 = 7.6% | 446 box máy |
| deleted (xoá) | 57 | 57/446 = 12.8% | 446 box máy |
| added (vẽ thêm) | 101 | 101/482 = 21.0% | 482 đối tượng sau duyệt |

Lưu ý: Tỉ lệ added sử dụng mẫu số là 482 đối tượng sau duyệt, trong khi accepted, edited và deleted sử dụng 446 box máy.

Trong nhóm deleted và added có 9 trường hợp đổi lớp, tức box máy đặt đúng vị trí nhưng dự đoán sai class. Do đó, số đối tượng máy bỏ sót thực sự được ước tính là 92 (101 added trừ 9 trường hợp đổi lớp).

Kết quả theo lớp
| Lớp | Accepted | Edited | Deleted | Added|
|---|---|---|---|---|
| person | 213 | 20 | 20 | 23
| bicycle | 6 | 0 | 3 |4
| motorbike | 75 | 10| 4 |43
| car | 42 | 0 | 6 |21
| bus | 1 | 0 | 1 |3
| truck | 1 | 0 | 5 |2
| three_wheeler | 9 | 4 | 18 |5

- Lớp `máy` hoạt động chưa tốt
- Motorbike có 43 đối tượng được thêm, nhiều nhất trong các lớp, cho thấy model bỏ sót khá nhiều đối tượng thuộc lớp này trong lô review.
- Three_wheeler có 18 box bị xóa và 4 box phải sửa. Trong quá trình review, tôi cũng gặp trường hợp model nhầm truck thành three_wheeler và dự đoán trùng nhiều box cho cùng một xe.
- Các ví dụ cụ thể cần đối chiếu lại với ảnh trong CVAT trước khi chốt báo cáo.
So với gold: 

Kết quả của máy trên 4 ảnh gold:
- TP: 42.
- FP: 19.
- FN: 10.
- Precision: 68,9% = 42/61.
- Recall: 80,8% = 42/52.
- Tổng số đối tượng gold: 52.

Các chỉ số được lấy từ submission/qc/review_stats.md. Do threshold được chọn trên cùng tập gold này nên kết quả có thể lạc quan.
## 3. Thiên kiến tự động hóa và ảnh bẫy

Kết quả phát hiện bẫy: 10/12 = 83,3%, vượt ngưỡng yêu cầu tối thiểu 80%.
| Loại bẫy | Bắt được | Tổng số|
|---|---|---|
| wrong_class |2 | 3
| loose |2 | 3
| extra |2 | 3
| missing |2 | 3
|Tổng|10|12

Hai bẫy bị lọt:
- img_031.jpg: loose – box chưa được phát hiện/sửa đúng theo bẫy.

- img_014.jpg: wrong_class – nhãn sai chưa được phát hiện.

Trong quá trình review, em gặp các trường hợp:

- Truck bị gán nhầm thành three_wheeler.

- Một chiếc xe bị gán three_wheeler nhiều lần.

- Bounding box vượt giới hạn chiều cao theo guideline.

- Bounding box quá nhỏ, chưa bao phủ hết vật thể.

- Một đối tượng nằm sát mép ảnh, không đủ thông tin để xác định class.

Khi review, em nhận thấy nếu nhìn box AI trước rồi mới quan sát ảnh thì dễ bị ảnh hưởng bởi dự đoán có sẵn. Vì vậy, cần quét ảnh và tự đếm vật thể trước khi kiểm tra annotation.

Accepted cao không đồng nghĩa với việc máy hoàn toàn chính xác. Một box được giữ nguyên có thể vẫn chứa lỗi mà người review không nhận ra. Kết quả bắt bẫy giúp kiểm tra thêm mức độ cẩn thận của người duyệt, không chỉ dựa vào tỷ lệ accepted.

## 4. Máy có đáng dùng không?
So sánh thời gian
- Thời gian  gán tay: 30 phút
- Thời gian review: 1 tiếng 30 phút
- Số ảnh gán tay và số ảnh review cần dùng làm mẫu số: 13
Có một hạn chế là tôi đã gán nhãn gold trước khi review batch nên đã quen với một số đặc điểm của dữ liệu. Điều này có thể khiến tôi nhận diện vật thể nhanh hơn và làm ảnh hưởng đến tính khách quan khi đánh giá AI.

Kết luận

Có thể sử dụng AI để hỗ trợ tạo pre-label, nhưng không nên sử dụng kết quả dự đoán mà không qua bước review.

Lý do:

- 77,8% box máy được giữ nguyên.

- 7,6% box máy cần chỉnh sửa.

- 12,8% box máy bị xóa.

- Máy còn bỏ sót khoảng 92 đối tượng sau khi loại các trường hợp đổi lớp khỏi số added.

- Bắt được 10/12 lỗi bẫy, đạt 83,3%.

**Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
1. Quét toàn bộ ảnh từ trái sang phải và tự đếm đối tượng trước khi nhìn box AI.

2. Kiểm tra từng box về class, vị trí và kích thước; không xác nhận hàng loạt chỉ vì box có confidence cao.

3. Kiểm tra riêng các đối tượng ở mép ảnh, bị che khuất và các class dễ nhầm; đánh dấu can_review khi chưa đủ căn cứ xác định.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 positive, 1 negative) | Cần kiểm tra trực quan | Mask có diện tích 317,895 pixels (45.5% ảnh); cần kiểm tra có lấy thừa người lái, hàng hóa hoặc nền không. | `tuktuk_img013_sam2.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 positive, 1 negative) | Cần kiểm tra trực quan | Mask có diện tích 93,436 pixels (13.4% ảnh); cần kiểm tra ranh giới mask và các phần bị bỏ sót. | `tuktuk_img004_sam2.jpg` |
| img_017.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 positive, 1 negative) | Cần kiểm tra trực quan | Mask có diện tích 84,812 pixels (12.1% ảnh); cần kiểm tra mask có bao phủ đủ phương tiện và tránh vùng nền. | `tuktuk_img017_sam2.jpg` |

Ngoài ra, tôi thử so sánh MobileSAM (`mobile_sam.pt`) trên `img_013.jpg` với cùng 3 click:
- SAM2.1: 317,895 pixels (45.5% ảnh), 1 polygon.
- MobileSAM: 246,370 pixels (35.3% ảnh), 2 polygons.

Kết quả cho thấy hai model tạo ra mask có diện tích và số polygon khác nhau. Tuy nhiên, không thể kết luận model nào tốt hơn chỉ dựa vào diện tích mask; cần đối chiếu với ranh giới vật thể thực tế.

### Đối chiếu guideline mục 4.2

- Quy tắc hữu ích: Sử dụng positive points để chỉ định vùng thuộc vật thể và negative points để loại trừ vùng không thuộc vật thể. Điều này giúp định hướng SAM tạo mask đúng đối tượng hơn.
- Quy tắc cần làm rõ: Cách xử lý người lái, hàng hóa hoặc các bộ phận gắn liền với phương tiện khi xác định phạm vi mask. Cần thống nhất mask chỉ bao phủ phương tiện hay bao gồm cả các phần này theo định nghĩa trong guideline.
- Cần kiểm tra trực quan kết quả mask trước khi sử dụng, không đánh giá chất lượng chỉ dựa trên số pixel hoặc số polygon.

### Interactor SAM trong CVAT

Chưa có dữ liệu xác nhận đã thử Interactor SAM trực tiếp trong CVAT nên chưa kết luận được trạng thái hoạt động.

Khác biệt về quy trình:
- SAM offline: Chạy script Python riêng, truyền ảnh cùng positive/negative points, sau đó xuất kết quả mask ra thư mục `submission/masks/`.
- SAM Interactor trong CVAT: Thao tác trực tiếp trên giao diện CVAT để chọn điểm và tạo mask, thuận tiện cho việc chỉnh sửa annotation trong quá trình gán nhãn.

## 6. Calibration (Part 6)

- **Số quyết định:** 7 quyết định (`calibration_decisions.csv`), từ `CAL-001` đến `CAL-007`.

- **Quan trọng nhất:**
  - `CAL-003` (`img_013.jpg`): Thống nhất cách vẽ bounding box cho xe tuk-tuk. Một phương tiện liền khối chỉ có một bounding box bao phủ toàn bộ phần phương tiện nhìn thấy, không tách riêng từng bộ phận và không lấy thừa vùng nền.
  - `CAL-006` (`img_028.jpg`): Phân biệt các phương tiện độc lập trong ảnh đông xe. Mỗi phương tiện có annotation riêng; không gộp object và cần kiểm tra box duplicate.
  - `CAL-007` (`img_034.jpg`): Không mặc định class do model dự đoán là chính xác. Cần đối chiếu ảnh gốc và xác minh class trước khi chấp nhận pre-label.

- **Quy tắc mới đã đưa vào `guideline_v1.1_calibrated.md`:**
  1. Người và phương tiện phải được gán annotation riêng, không gộp người vào bounding box của phương tiện (`CAL-001`).
  2. Các object khác nhau vẫn có annotation riêng dù bounding box có vùng chồng lấn; cần kiểm tra để tránh duplicate (`CAL-002`, `CAL-006`).
  3. Bounding box của phương tiện phải bao phủ toàn bộ phần phương tiện nhìn thấy, không lấy thừa nền hoặc tách các bộ phận của cùng phương tiện thành nhiều object (`CAL-003`).
  4. Object bị che khuất chỉ được gán class khi phần nhìn thấy đủ căn cứ nhận diện (`CAL-004`).
  5. Object bị cắt bởi biên ảnh chỉ được bao phủ trong phạm vi thực sự nhìn thấy (`CAL-005`).
  6. Không chấp nhận class pre-label một cách máy móc; cần đối chiếu ảnh gốc và đưa trường hợp chưa rõ vào review (`CAL-007`).

- **Nguyên nhân gốc:** Chủ yếu do guideline chưa cụ thể ở một số tình huống thực tế và người gán nhãn chưa thống nhất cách xử lý các object gần nhau, bị che khuất hoặc bị cắt ở biên ảnh. Ngoài ra, model vẫn có thể dự đoán sai class, cần human review để xác nhận (`CAL-007`).

## 7. QC và quyết định (Part 6)
- **Mẫu:** 7 ảnh (2 ảnh ca khó + 5 ảnh ngẫu nhiên, seed 42), gồm 146 đối tượng. Người kiểm: Nguyễn Thanh Sơn (hạn chế: người kiểm cũng là người thực hiện gán nhãn và review trước đó, có thể tồn tại thiên kiến chủ quan).
- **Lỗi theo đối tượng:** 0/146 = 0.0% · **Lỗi theo ảnh:** 0/7 = 0.0% (ngưỡng Release <= 2%).
- **Top lỗi:** Báo cáo QC tự động chưa ghi nhận lỗi nào trong các nhóm bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính và sai mask. Tuy nhiên, kết quả này chưa đáng tin cậy vì các dòng có error_type không hợp lệ đã bị script bỏ qua.
- **Các trường hợp cần kiểm tra lại:**
  - `img_013.jpg`: kiểm tra box phương tiện có bị rộng hoặc lấy thừa vùng nền không.
  - `img_014.jpg`: kiểm tra class của phương tiện, đặc biệt trường hợp nghi ngờ sai lớp.
  - `img_034.jpg`: xác minh class truck theo ảnh gốc, không phụ thuộc hoàn toàn vào pre-label.
- **Giới hạn mẫu nhỏ:** Với 0 ảnh lỗi trên 7 ảnh kiểm tra, cận trên 95% theo Wilson của tỷ lệ ảnh lỗi thật là 35%. Mẫu hiện tại quá nhỏ để chứng minh tỷ lệ lỗi thực tế của toàn bộ dataset thấp hơn ngưỡng 2%.
- **Script kiểm được:**
  - Đếm số lỗi theo đối tượng và số ảnh có lỗi dựa trên phiếu kiểm hợp lệ.
  - Tổng hợp số lượng machine boxes accepted, edited, deleted và số object được added.
  - Thống kê khả năng phát hiện lỗi bẫy.
- **Script không tự kiểm được:**
  - Không tự xác định bounding box có ôm sát vật thể hay không.
  - Không tự phát hiện các object bị bỏ sót nếu người kiểm không ghi nhận.
  - Không xác minh class đúng hay sai nếu phiếu kiểm không có quyết định chính xác.
  - Không đánh giá chất lượng mask và các lỗi theo checklist bằng mắt nếu chưa được ghi vào phiếu kiểm.
  - Không loại bỏ hoàn toàn thiên kiến của người vừa gán nhãn vừa kiểm tra.

### Quyết định: HOLD, ngày 2026-11-18
**Lý do:**

1. **Hold:** Chưa thể xác nhận chất lượng cuối cùng vì còn 7 ảnh trong mẫu chưa được kiểm tra và 3 dòng `error_type` không hợp lệ (`loose`, `ambiguous`, `wrong_class`). Ngoài ra, guideline và một số trường hợp phân loại vẫn cần được xác nhận.
2. **Rework:** Chưa đủ căn cứ để kết luận dataset cần làm lại toàn bộ. Cần sửa phiếu kiểm, kiểm tra lại các trường hợp nghi vấn và cập nhật báo cáo QC trước.
3. **Release:** Chưa thể Release chính thức. Mặc dù kết quả hiện tại là 0/7 ảnh lỗi và khả năng bắt bẫy đạt 10/12 = 83.3%, mẫu nhỏ chưa đủ chứng minh tỷ lệ lỗi dưới 2%.

**Việc cần làm tiếp theo:**
- Kiểm tra đầy đủ 7 ảnh: `img_003.jpg`, `img_006.jpg`, `img_013.jpg`, `img_014.jpg`, `img_022.jpg`, `img_028.jpg`, `img_034.jpg`.
- Sửa các giá trị `error_type` theo taxonomy hợp lệ:
  - `loose` → `sai_vi_tri_kich_thuoc`.
  - `wrong_class` → `sai_lop`.
  - `ambiguous` → đánh giá lại theo lỗi thực tế; nếu chưa xác định được lỗi thì không tự gán loại lỗi.
- Cập nhật phiếu kiểm và chạy lại `review_stats.py`/script QC.
- Đánh giá lại kết quả và quyết định Release hoặc Rework sau khi hoàn tất kiểm tra.

## 8. Ghi chú cá nhân
- Dùng AI để làm bài

Em sử dụng các công cụ AI hỗ trợ trong quá trình tìm hiểu model, giải thích lỗi annotation và hỗ trợ đọc kết quả. Các quyết định gán nhãn và chỉnh sửa cuối cùng được kiểm tra trực tiếp trên ảnh và CVAT.

- Gold có bị nhiễm không?

[CẦN XÁC NHẬN: Trong quá trình gán gold ban đầu, có nhìn pre-label hoặc hỏi AI trước khi hoàn tất hay không.]

Máy / GPU / CVAT

Hệ điều hành: Windows, sử dụng PowerShell.

Môi trường Linux: Ubuntu 24.04 trên WSL2.

Docker: Docker Desktop sử dụng WSL2.

CVAT: triển khai bằng Docker Compose, phiên bản v2.81.0.

Model inference và SAM: có các lần chạy bằng CPU; log SAM ghi nhận trung bình 8,53 giây/ảnh ở run r04.

- Cache

[CẦN XÁC NHẬN: Có sử dụng dữ liệu trong data/cache/ hay tự chạy lại các bước tạo dữ liệu.]

- Lỗi / bug script

Trong quá trình làm bài, tôi gặp một số vấn đề về cấu hình CVAT và import annotation. Cụ thể, khi import COCO, CVAT báo label person chưa được đăng ký cho Task. Nguyên nhân là Task được tạo độc lập, chưa nằm trong Project có đầy đủ labels. Sau đó tôi chuyển Task vào Project Day08 và thực hiện lại thao tác import.

- Một điều em sẽ làm khác lần sau

Em sẽ tạo Project và cấu hình đầy đủ labels trước khi tạo Task, đồng thời kiểm tra kỹ cấu trúc annotation và danh sách class trước khi import. Khi review, tôi sẽ chủ động quan sát ảnh gốc trước khi nhìn box AI để giảm thiên kiến tự động hóa.