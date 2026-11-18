# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 446; đối tượng sau duyệt: 482; ngưỡng accepted: IoU >= 0.9
- Đã **loại 29 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 347 | 347/446 = 77.8% | 446 box máy |
| edited (sửa) | 34 | 34/446 = 7.6% | 446 box máy |
| deleted (xoá) | 57 | 57/446 = 12.8% | 446 box máy |
| added (vẽ thêm) | 101 | 101/482 = 21.0% | 482 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 92.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 213 | 20 | 20 | 23 |
| bicycle | 6 | 0 | 3 | 4 |
| motorbike | 75 | 10 | 4 | 43 |
| car | 42 | 0 | 6 | 21 |
| bus | 1 | 0 | 1 | 3 |
| truck | 1 | 0 | 5 | 2 |
| three_wheeler | 9 | 4 | 18 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 52 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 19 | 10 | 68.9% (KTC 95%: 56–79%) | 80.8% (KTC 95%: 68–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - loose: 2/3
  - extra: 3/3
  - missing: 3/3
  - lọt: img_031.jpg (loose), img_014.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
