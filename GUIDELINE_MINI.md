# Mini guideline - nhóm: 2A  |  người gán: Hoàng Văn Đạt  |  ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung → `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh → `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm (đã thống nhất)

| Tình huống | Luật nhóm bạn chọn | Vì sao |
| --- | --- | --- |
| Hông của người mặc quần áo dài | `v=1`, đặt chấm ước lượng giữa eo và đùi | Hông là điểm giải phẫu có thể suy luận từ eo và đùi — vẫn còn trong khung |
| Tai bị tóc hoặc mũ bảo hiểm che một phần | `v=1`, đặt chấm ước lượng phía sau tai có thể thấy | Tai vẫn trong khung, tóc/mũ là vật che — dùng viền tai còn thấy để ước lượng |
| Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên) | Khớp còn trong khung → `v=1`; khớp ra ngoài → `v=0` | Dùng mép ảnh làm ranh giới; không bao giờ dùng `v=0` cho khớp còn trong khung |
| Cổ tay nằm sau tay lái / sau thân mình | `v=1`, đặt chấm ước lượng theo hướng cánh tay | Bị che bởi vật nhưng vẫn trong khung — `v=1` theo luật bắt buộc |
| Hai người chồng lên nhau | Làm xong hẳn người phía trước rồi mới sang người sau; khớp bị người kia che → `v=1` | Phân biệt rõ hai skeleton, không để chấm lẫn sang cơ thể người khác |
| Người nhỏ đến mức nào thì không gán nữa | Gán nếu có thể xác định ít nhất vai và hông; bỏ qua nếu cao < 30px | Dưới 30px không đủ thông tin giải phẫu để gán chính xác 17 điểm |

## 3. Ba ca mơ hồ đã gặp

### Ca 1 - ảnh `train_04`, người thứ `2`, khớp `left_knee` và `right_knee`

- Mơ hồ ở chỗ nào: Đầu gối bị thân xe máy che hoàn toàn, không nhìn thấy trực tiếp. Ban đầu phân vân `v=0` hay `v=1`.
- Bạn quyết thế nào: `v=1`, đặt chấm ước lượng tại vị trí giải phẫu giữa đùi và cẳng chân.
- Vì sao: Nhìn phần đùi và cẳng chân vẫn thấy ở hai bên thân xe → đầu gối vẫn còn trong khung, chỉ bị xe che. `v=0` chỉ dành cho khớp ra ngoài mép ảnh.
- Nếu người khác quyết ngược lại (`v=0`): Model không học được vị trí đầu gối của người ngồi xe máy, dẫn đến sai hệ thống ở loại ảnh này.

### Ca 2 - ảnh `train_13`, người thứ `1` (gold_person #1), khớp toàn bộ

- Mơ hồ ở chỗ nào: Có một người nhỏ đứng góc trái ảnh, bị hai người khác che một phần. Phân vân có đủ thông tin để gán không.
- Bạn quyết thế nào: Bỏ qua, không gán skeleton cho người này.
- Vì sao: Không xác định được đủ các điểm mốc giải phẫu (vai, hông) để ước lượng 17 khớp một cách có căn cứ.
- Nếu người khác quyết ngược lại (gán): Gold xác nhận người này cần được gán — đây là lỗi thiếu người, làm OKS của ảnh này bằng 0 cho skeleton đó.

### Ca 3 - ảnh `train_02`, người thứ `1`, khớp `right_hip` và `right_knee`

- Mơ hồ ở chỗ nào: Người nhìn từ sau lưng, ngồi trên xe đạp. Hông và đầu gối phải bị yên xe và khung xe che — phân vân trái/phải và `v=1` hay `v=0`.
- Bạn quyết thế nào: `v=1` cho cả hai, đặt chấm ước lượng theo giải phẫu dựa trên vị trí chân đang đạp pedal.
- Vì sao: Khớp vẫn trong khung ảnh, chỉ bị xe che. Có thể suy ra vị trí từ hướng chân và pedal xe.
- Nếu người khác quyết ngược lại (`v=0`): Model học sai pose người đi xe đạp nhìn từ sau, đặc biệt phần hông và đầu gối.

## 4. Sau khi so visibility report với bạn cùng nhóm

- Khớp lệch `%v=1` nhiều nhất: `left_ear` (68% vs gold COCO thấp hơn nhiều)
- Nguyên nhân là **guideline khác nhau**: Gold COCO dùng `v=0` cho cả "ra ngoài khung" và "không gán", trong khi luật lớp yêu cầu `v=1` cho khớp bị che còn trong khung. `%v=1` cao hơn gold là đúng luật lớp.
- Luật mới bổ sung: Tai bị tóc/mũ che một phần nhưng còn trong khung → `v=1`, đặt chấm ở vị trí viền tai có thể thấy hoặc ước lượng theo cấu trúc đầu.
