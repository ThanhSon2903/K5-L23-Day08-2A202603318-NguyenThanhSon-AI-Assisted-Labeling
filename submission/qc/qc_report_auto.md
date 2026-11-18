# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 7 ảnh (2 ca khó + 5 ngẫu nhiên), 146 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/146 = 0.0% | 146 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/7 = 0.0% | 7 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 347/446 = 77.8% / 34/446 = 7.6% / 57/446 = 12.8% | 446 box máy |
| Máy: added | 101/482 = 21.0% | 482 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Cảnh báo
- Dòng 4: error_type 'loose' sai (chỉ nhận khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask) - bỏ qua.
- Dòng 6: error_type 'ambiguous' sai (chỉ nhận khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask) - bỏ qua.
- Dòng 8: error_type 'wrong_class' sai (chỉ nhận khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask) - bỏ qua.
- 7 ảnh trong mẫu CHƯA kiểm: img_003.jpg, img_006.jpg, img_013.jpg, img_014.jpg, img_022.jpg, img_028.jpg, img_034.jpg

## Giới hạn thống kê
- 0 ảnh lỗi trong 7 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **35%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**RELEASE**

- ảnh lỗi 0.0% <= 2%, không lỗi hệ thống, bắt bẫy / nhất quán đạt (nếu đã đo)
- LƯU Ý: còn ảnh mẫu chưa kiểm -> kết luận chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
