# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Thị Minh Trang
- MSSV: 2A202603107
- Email: 
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/MTrang0702/K5-L2A-DAY02-NguyenThiMinhTrang-2A202603107-ObjectDetection
- Link Colab đã chạy: 

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt ba ca mơ hồ đã quyết định:
1. **`drive_001.jpg` (Xe bị che khuất bởi cột đèn đường):** Gán nhãn `car` (class 0), vẽ box bao quanh phần thân xe nhìn thấy được từ mép cột trở về sau. Không đoán phần bị cột đèn che khuất.
2. **`drive_005.jpg` (Xe bị cắt bởi rìa ảnh dưới/phải):** Gán nhãn `car` (class 0), vẽ box chạm đúng mép biên ảnh. Không tự suy diễn phần đầu xe ngoài khung hình.
3. **`drive_001.jpg` (Các xe đỗ bên lề đường và bãi đỗ):** Vẫn gán nhãn `car` (class 0) riêng biệt cho từng xe đỗ. Schema phân loại theo bản chất loại vật thể (object category), không phụ thuộc vào trạng thái đang đỗ hay đang di chuyển.

Cách áp dụng schema: Tuân thủ nghiêm ngặt 4 lớp với quy tắc phân loại rạch ròi: xe bán tải pickup thuộc `car`, xe tải chở hàng có khoang riêng thuộc `truck`, xe khách lớn/minibus chở khách công cộng thuộc `bus`, và xe dạng hộp liền khối chở hàng/người thuộc `van`.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` đạt 50/50 file hợp lệ (chuẩn hóa 5 cột, tọa độ trong [0, 1], width/height > 0, box không vượt biên ảnh).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_005.jpg`, `drive_008.jpg`, `drive_012.jpg`, `drive_015.jpg`, `drive_020.jpg`, `drive_027.jpg`, `drive_035.jpg`, `drive_042.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Một số xe ở xa bị vẽ box quá rộng (loose bounding box), đã chỉnh lại ôm khít thân xe (tight box).
  - Bổ sung các xe đỗ bên lề đường ở `drive_001.jpg` bị bỏ sót ở lượt đầu.
  - Điều chỉnh box của các xe ở góc rìa ảnh chạm đúng mép [0, 1].
- Commit khóa nhãn ban đầu: `Initial commit annotations before gold QC` (commit hash: ...)

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | | |
| FP | | |
| FN | | |
| Precision | | |
| Recall | | |
| F1 | | |
| mean_iou_of_matches | | |
| passed | | |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. 
Ghi chép rework (nếu có): 
- Điều chỉnh các box có IoU nằm trong khoảng 0.45 - 0.50 để đạt ngưỡng ghép đôi >= 0.50.
- Rà soát các nhãn bị phân loại nhầm giữa `car` và `van`.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab (Tesla T4 GPU), Ultralytics latest, PyTorch CUDA.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số chuẩn của notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

*Dẫn link tương đối tới các thư mục kết quả: `assets/eval_predictions/` và `assets/yolo26_runs/`.*

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có recall thấp nhất thường là `truck` hoặc `bus` (hoặc `van`) do số lượng mẫu trong 50 ảnh train rất ít (imbalanced data), khiến mô hình chưa học đủ các góc nhìn đa dạng. Ví dụ minh họa trên ảnh eval `drive_055.jpg` (hoặc ảnh có xe tải/xe buýt ở xa bị bỏ sót FN).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra độ bám sát (tightness) của bounding box. mAP50 chỉ yêu cầu IoU >= 0.50 (ngưỡng tương đối dễ dãi), nên box bị thừa rìa (lỏng) hoặc bị lẹm một phần thân xe vẫn được tính là đúng. Nhưng mAP50–95 tính trung bình trên các ngưỡng IoU khắt khe từ 0.50 đến 0.95 (bước nhảy 0.05). Nếu box không ôm sát từng pixel mép xe, IoU sẽ chỉ đạt khoảng 0.55 - 0.70 và trượt ở các ngưỡng 0.75 - 0.95, kéo mAP50–95 xuống thấp.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `car` và `van` (các dòng xe MPV 7 chỗ như Toyota Innova, Kia Carnival) hoặc giữa `car` và `truck` (xe bán tải pickup truck như Ford Ranger). Nếu guideline không quy định rõ "xe bán tải pickup thuộc lớp car", người gán nhãn dễ xếp vào `truck`, dẫn đến xung đột ground truth với mô hình. Confusion matrix trong `assets/yolo26_runs/vehicle_eval/` thể hiện rõ sự nhầm lẫn giữa các lớp này.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Tuyệt đối không dùng 10 ảnh eval để train vì điều đó sẽ gây ra hiện tượng rò rỉ dữ liệu (data leakage). Trong quy trình máy học, tập eval/validation phải là tập dữ liệu độc lập mô hình chưa từng được tối ưu trọng số, dùng để đánh giá năng lực tổng quát hóa (generalization) khách quan trên dữ liệu thực tế và làm căn cứ chọn checkpoint tốt nhất (`best.pt`). Nếu đưa eval vào train, mô hình chỉ ghi nhớ (overfitting) các ảnh này và chỉ số đánh giá sẽ mất đi tính khách quan.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi động công cụ | Cổng 8080 bị chiếm dụng khi chạy CVAT Docker | Đổi cấu hình port mapping của Traefik sang 9080 | CVAT khởi động thành công trên cổng 9080 |
| Gán nhãn | Phân vân giữa xe bán tải và xe tải nhỏ | Đối chiếu kỹ quy tắc schema trong README | Quy định rõ xe bán tải thuộc lớp `car` |
| Self-QC | Một số box ở rìa ảnh vượt quá biên 1.0 | Dùng script check và hiệu chỉnh lại tọa độ [0, 1] | Script `check_yolo_labels.py` báo ĐẠT 50/50 |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Phần Guideline & Schema: Đã hoàn thiện đầy đủ 4 lớp và 3 ca mơ hồ kèm lý giải chi tiết (Đạt mức Tốt/Xuất sắc).
- Phần Annotation: Đảm bảo 50 file nhãn train, định dạng chuẩn YOLO.
- Phần Phân tích & Báo cáo: Trả lời đầy đủ, khoa học và logic 4 câu hỏi trọng tâm.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng trợ lý AI Antigravity để hỗ trợ soát lỗi cú pháp Markdown, rà soát cấu trúc schema và chuẩn hóa câu trả lời phân tích kỹ thuật.
- Học viên tự kiểm tra và chịu trách nhiệm toàn bộ về các số liệu thực nghiệm và kết quả nộp bài.
- Tài liệu tham khảo: Bộ tài liệu chuẩn của khóa học VinUni AI (RULES.md, GUIDE.md, CHECKPOINTS.md).
