# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202603320
- Ngày / CVAT local: 11/11/2026 — CVAT localhost:8080
- Công cụ đã dùng: CVAT, SegFormer-B5 (nvidia/segformer-b5-finetuned-cityscapes-1024-1024)

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- **Ảnh, vị trí và object Medium đầu tiên tự vẽ:** Ảnh 1 (`medium_instance`), chiếc xe ô tô màu trắng ở làn giữa, gần trung tâm khung hình.
- **Class và quy tắc tôi dùng để chọn biên:** Class `car`. Tôi vẽ mask bám sát viền kim loại thân xe — điểm dừng là mép ngoài cùng của thân xe, không bao gộp bóng đổ dưới gầm. Gương chiếu hậu được gộp vào cùng instance vì nó là bộ phận vật lý gắn liền với xe.
- **Nếu dùng gợi ý sau đó:** Sau khi tự vẽ xong object đầu tiên, tôi dùng AI proposal cho các xe còn lại. Proposal thường bắt đúng thân xe nhưng hay bỏ sót gương hoặc bao thêm bóng đổ. Tôi giữ biên thân xe, kéo thêm phần gương còn thiếu và xóa phần bóng bị bao vào.

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** `medium_instance`, chiếc xe ô tô bị một người đi bộ che khuất một phần.
- **Lỗi thuộc loại:** Gộp/tách — một vật thể (một chiếc xe) bị vẽ thành hai mảnh mask rời nhau thay vì một instance duy nhất.
- **Bằng chứng tôi nhìn thấy:** Nhìn lại danh sách object trong CVAT thấy cùng một chiếc xe xuất hiện thành hai shape riêng biệt — phần xe lộ ra bên trái người và phần lộ ra bên phải người được vẽ tách rời do bị người che ở giữa.
- **Quy tắc và hành động sửa:** Một instance phải là một object duy nhất dù bị che khuất một phần — phần bị che vẫn thuộc cùng instance đó. Tôi xóa hai mảnh rời, vẽ lại thành một mask liên tục bao phủ toàn bộ xe (kể cả vùng bị người che) thành một shape duy nhất.
- **Sau sửa đã Save và export lại chưa?** Đã Save trong CVAT và export lại `medium_instance.zip`.

Chưa chạy scoring script sau khi sửa — sẽ kiểm tra sau khi nộp.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `hard_panoptic` — xe máy có người ngồi, người rất nhỏ trong khung hình | (1) Label `motorcycle` bao cả người ngồi vì người và xe dính liền, khó tách; (2) tách riêng `motorcycle` và `person` thành hai instance độc lập | Panoptic yêu cầu mỗi "thing" là một instance riêng; người và xe là hai vật thể khác class | Tôi tách thành 2 instance: `motorcycle` cho thân xe, `person` cho phần thân người nhìn thấy được phía trên. Hỏi coach: phần chân người bị xe che có cần fill tiếp vào mask `person` không? |
| `cp4_curb` — vùng lề đường có màu đất/cát lẫn vào, không rõ là `sidewalk` hay `road` | (1) Label toàn bộ vùng đó là `sidewalk` vì nằm ở rìa đường; (2) label là `road` vì xe có thể đi lên được, bề mặt giống mặt đường | Cityscapes định nghĩa `sidewalk` là vùng đi bộ nằm ngang mức hoặc cao hơn mặt đường; vùng đất/cát không có vạch phân làn rõ | Tôi label là `sidewalk` vì vị trí nằm ngoài rìa, không có vạch đường. Hỏi coach: vùng đất trần sát lề có quy tắc label riêng không? |
| `cp3_thin` — cột đèn đường có phần thanh ngang bắc ngang trên đỉnh để gắn bóng đèn | (1) Chỉ label phần thân đứng là `pole`, bỏ phần thanh ngang vì hình dạng khác với "cột"; (2) gộp cả thanh ngang vào mask `pole` vì nó là bộ phận vật lý gắn liền, không phải vật thể riêng | Cityscapes định nghĩa `pole` là các kết cấu hình trụ nhỏ đứng thẳng; thanh ngang không có hình dạng cột đứng nhưng cũng không thuộc class nào khác trong dataset | Tôi gộp thanh ngang vào cùng mask `pole` vì không có class nào phù hợp hơn. Hỏi coach: theo định nghĩa Cityscapes, thanh ngang của cột đèn đường có được tính là `pole` không, hay bỏ qua? |
