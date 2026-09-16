# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Hoàng Văn Đạt   Nhóm: 2A   Ngày: 16/09/2026

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 28 |
| v=2 / v=1 / v=0 | 332 / 116 / 28 |
| Thời gian trung bình mỗi ảnh | ~4 phút |

Ba khớp có `%v=1` cao nhất (từ `reports/visibility_report.md`):

1. `left_ear` — 68% (19/28 người)
2. `right_ear` — 43% (12/28 người)
3. `left_eye` / `right_eye` / `left_hip` / `right_wrist` — 32%

Đây đúng là những khớp khó gán nhất. `left_ear` thường bị tóc, mũ bảo hiểm hoặc góc nhìn che hoàn toàn. `right_ear` tương tự nhưng ít hơn vì nhiều người quay mặt nghiêng sang phải. `left_hip` khó vì không có bề mặt giải phẫu nhìn thấy được trên người mặc quần áo — phải ước lượng từ eo và đùi. Điều này khớp với những lần dừng lại phân vân nhiều nhất khi gán.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.956 | 0.956 |
| OKS@0.50 | 0.966 | 0.966 |
| OKS@0.75 | 0.966 | 0.966 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 0 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |
| Thiếu người | 1 | 1 |
| Lệch nhẹ | 1 | 1 |

**Tôi đã sửa gì giữa hai lần chạy:**

Không thực hiện rework vì OKS đã đạt mức Xuất sắc (≥ 0.85) ngay lần đầu. Lỗi duy nhất là `train_13` thiếu 1 skeleton (gold_person #1) — người này nhỏ, đứng ở góc ảnh và bị che một phần. Quyết định không bổ sung vì không đủ thông tin giải phẫu để gán chính xác.

**Lỗi đảo trái/phải:** Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh.

## 3. Kiểm chéo

Bạn cùng nhóm: *(chưa thực hiện kiểm chéo — không có bạn cùng nhóm trong phiên này)*

Khớp lệch `%v=1` nhiều nhất so với gold COCO:

| Khớp | Bạn | Gold COCO | Lệch | Nguyên nhân |
| --- | ---: | ---: | ---: | --- |
| left_ear | 68% | thấp hơn | cao | Luật lớp: bị che trong khung → v=1; COCO dùng v=0 cho cả "không gán" |
| right_ear | 43% | thấp hơn | cao | Tương tự left_ear |
| left_ankle / right_ankle | 14% (v=0: 9/28) | — | — | Khớp ra ngoài khung hoặc bị xe che hoàn toàn |

Luật bổ sung sau khi đối chiếu: khớp bị che nhưng còn trong khung ảnh → `v=1` + đặt chấm ước lượng dựa trên giải phẫu. `%v=1` cao hơn gold là đúng luật lớp, không phải gán ẩu.

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

**1. `pose_mAP50-95` thay đổi bao nhiêu?**

Tăng nhẹ +0.0055 (từ 0.6853 lên 0.6908). 20 ảnh không đủ để thay đổi lớn, nhưng cũng không làm model tệ hơn — đây là tín hiệu nhãn chất lượng tốt. Model gốc đã được train trên toàn bộ COCO nên rất khó cải thiện thêm với chỉ 20 ảnh. Điều 20 ảnh này có thể dạy thêm là phân phối pose đặc thù (người ngồi xe máy, người cầm vật, v.v.) nhưng cũng làm model kém tổng quát hơn trên các pose khác — thể hiện qua `box_mAP50-95` giảm nhẹ -0.0078.

**2. `box_mAP` và `pose_mAP` chênh nhau bao nhiêu?**

`box_mAP50-95` = 0.8041, `pose_mAP50-95` = 0.6908, chênh ~0.113. Model tìm *người* (bounding box) dễ hơn đáng kể so với tìm *khớp* chính xác. Điều này hợp lý: phát hiện người chỉ cần xác định vùng chứa người, còn keypoint đòi hỏi định vị từng khớp đến vài pixel, đặc biệt khó với các khớp bị che.

**3. Một ảnh test model đoán sai:**

`train_06` (OKS model vs nhãn = 0.619 — thấp nhất). Lỗi thuộc loại **lệch nhẹ đến trượt hẳn**: model đặt một số khớp lệch khỏi vị trí đúng, đặc biệt với người đứng từ xa hoặc bị che một phần. Không thấy dấu hiệu đảo trái/phải hay nhầm người.

**4. Ảnh có OKS thấp nhất giữa nhãn và model:**

`train_06` với OKS = 0.619. Nhìn lại ảnh, đây là người đội mũ bảo hiểm che mặt — mặt và các khớp đầu khó xác định. Trong trường hợp này **nhãn của tôi có khả năng đúng hơn** vì tôi có thể dùng suy luận giải phẫu để ước lượng vị trí, còn model phụ thuộc vào đặc trưng thị giác bị che khuất. Gold cũng xác nhận nhãn của tôi không có lỗi `dao_trai_phai` hay `nham_nguoi` ở ảnh này.

**5. Ảnh gán tệ nhất có trùng ảnh model đoán tệ nhất không?**

`train_13` là ảnh tôi gán thiếu người (OKS = 0 cho gold_person #1), và cũng là ảnh model phát hiện nhiều người hơn tôi (model 3 / tôi 2). Điều này cho thấy `train_13` là ảnh **khó về mặt bao phủ người**: có nhiều người, một số bị che, và cả tôi lẫn model đều không đồng thuận về số lượng người cần gán. Ảnh này có thể là ca biên giới về kích thước/che khuất người.

## 5. Một rule evidence đã dùng

`train_04`, người thứ 2 (ID 109, áo xám ngồi xe máy bên trái), khớp `left_knee` và `right_knee`:

Hai đầu gối không nhìn thấy trực tiếp vì bị thân xe máy che. Tuy nhiên, phần đùi và cẳng chân vẫn nhìn thấy một phần ở hai bên thân xe — suy ra đầu gối vẫn nằm trong khung ảnh (không ra ngoài mép). Theo luật lớp: khớp bị che nhưng còn trong khung → `v=1` + đặt chấm ước lượng tại vị trí giải phẫu (chỗ khuỷu chân, giữa đùi và cẳng chân). Ban đầu tôi gán `v=0` nhầm vì không nhìn thấy trực tiếp, nhưng sau khi đọc cảnh báo từ `check_pose_labels.py` đã xác nhận lại và sửa thành `v=1`.
