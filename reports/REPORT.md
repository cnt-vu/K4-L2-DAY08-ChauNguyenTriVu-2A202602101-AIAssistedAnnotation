# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Châu Nguyễn Tri Vũ

Công cụ gán nhãn đã dùng: CVAT v2.76.0 chạy bằng Docker trên máy cá nhân

## 1. Dữ liệu và cách chia tập

Video được lấy mẫu 2.5 ảnh/giây từ một camera cố định nên hai frame cách nhau 0.4 giây và cùng một xe có thể xuất hiện ở nhiều frame liền nhau. Dữ liệu được chia theo thời gian: 20 ảnh test nằm trong bốn đoạn quanh giây 20, 60, 100 và 140; 112 ảnh ở vùng đệm bị loại bỏ; 268 ảnh còn lại thuộc pool. Frame pool gần test nhất cách 4.4 giây. Cách chia này hạn chế ảnh gần như giống nhau hoặc cùng chiếc xe lọt vào cả train và test. Nếu chia ngẫu nhiên, test có thể chứa frame sát frame train; mô hình đã thấy gần cùng cảnh/xe nên số đo thường bị cao hơn khả năng tổng quát trên cảnh mới (rò rỉ dữ liệu).

## 2. Mô hình khởi đầu lạnh (cold start)

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

Ở `frame_050`, ảnh tham chiếu có nhiều xe trên đường đêm; cột cold start chỉ khớp một số box, bỏ sót nhiều xe và có box dự đoán không khớp. Sai lệch dễ thấy ở các xe nhỏ/xa và vùng đèn lóa. Recall xác nhận khó khăn này: xe nhỏ 0.182, thấp hơn xe cỡ vừa 0.547 và xe lớn 0.561. Trước khi kết luận mô hình sai ở một box sát đường chân trời, cần rà lại tham chiếu: có box rất nhỏ chỉ còn hai chấm đèn, khó xác định đó là xe từ bốn bánh trở lên hay đối tượng khác. Tổng tham chiếu gồm 403 box được chấm và 14 box cao dưới 16 px bị bỏ qua. Nhãn test do mô hình tạo, chưa được người rà từng box; tôi không sửa nhãn test.

## 3. Chiến lược chọn mẫu

Điểm `score = W_U·U + W_A·A + W_D·D` cộng ba thành phần: `U` là trung bình độ bất định của tối đa năm box khó nhất; `A` là số box có confidence từ 0.15 đến dưới 0.50, chuẩn hóa theo số lớn nhất trong pool; `D` là khoảng cách thời gian tới frame đã gán nhãn gần nhất, chặn ở 10 giây rồi chuẩn hóa. Trọng số lần lượt là 0.5, 0.3 và 0.2. Ở vòng chọn đầu chưa có frame đã gán nên `D=1` cho mọi frame. `MIN_GAP_S=2.0` loại các ảnh cách frame đã chọn dưới hai giây để giảm ảnh gần trùng; nếu chưa đủ 12 ảnh thì thuật toán giảm dần khoảng cách.

Ba frame được chọn làm bằng chứng là `frame_0182.jpg` (rank 1, 0.9591), `frame_0369.jpg` (rank 2, 0.9324) và `frame_0326.jpg` (rank 4, 0.9155). Tôi cũng cân nhắc `frame_0187.jpg` (rank 10, 0.8995) nhưng ưu tiên `frame_0227.jpg` (rank 11, 0.8915) để tránh dành ngân sách cho frame chỉ cách `frame_0182.jpg` hai giây và tăng độ phủ thời gian. `frame_0368.jpg` là ví dụ điểm cao nhưng không được chọn: score 0.9003, nhưng chỉ cách frame đã chọn `frame_0369.jpg` 0.4 giây. Điểm bất định giúp tìm nơi cần người xem; nó không bảo đảm ảnh có nhãn dễ sửa hoặc giúp AP50 tăng.

## 4. Các vòng học chủ động (active learning)

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vòng 1 | 12 | 327 | 0.474 | -0.298 | 1.000 | 0.030 | 0.058 | 0.000 | 0.037 | 0.024 |

Ở vòng 1, trong 169 box AI đề xuất có 148 box giữ nguyên, 6 box chỉnh sửa và 15 box xóa; tôi thêm 173 box. Sau fine-tune AP50 giảm 0.298 so với cold start (0.7714 xuống 0.4736). Precision tại conf 0.25 tăng từ 0.925 lên 1.000 nhưng recall giảm từ 0.489 xuống 0.030 và F1 từ 0.640 xuống 0.058. Recall giảm ở cả ba nhóm kích thước: small 0.182 xuống 0, medium 0.547 xuống 0.037, large 0.561 xuống 0.024. Như vậy vòng này không cho thấy nhóm kích thước nào tốt lên theo recall.

Trong `compare_round0.jpg` và `compare_round1.jpg`, `frame_250` giảm từ TP 6 / FP 2 / FN 9 ở cold start xuống TP 0 / FP 0 / FN 15 ở vòng 1 (ngưỡng conf 0.25). Đây là dấu hiệu cần kiểm tra vì mô hình vòng 1 bỏ sót nhiều box mà cold start tìm được. Chưa thể kết luận nguyên nhân chỉ từ ảnh; cần kiểm tra file nhãn/`data.yaml`, class id, dữ liệu train và confidence/training curve trước khi train tiếp.

Blind Scan là quan sát độc lập của tôi trên `frame_0270.jpg`: tôi ghi nhận 27 xe từ bốn bánh trở lên và nêu mô tô khó nhìn cùng một ô tô bị che một phần; mô tô không thuộc lớp `car`. Nhật ký `REVIEW_LOG.csv` ghi ba ví dụ pre-label bị bỏ sót ở `frame_0227.jpg`, `frame_0312.jpg` và `frame_0326.jpg`. Đây là các quan sát/sửa nhãn của người làm; `round1_diff.md` là phép đếm máy trên cả lô 12 ảnh (173 box thêm, 15 xóa, 6 sửa), không phải số AP. Theo guideline, với xe bị cắt mép dưới như `frame_0312.jpg` chỉ vẽ phần xe nằm trong ảnh; với xe bị cây che như `frame_0326.jpg` chỉ vẽ phần thân xe nhìn thấy.

## 5. Kết luận và giới hạn

Vòng 1 kém cold start trên bộ tham chiếu: AP50 giảm 0.298, recall giảm mạnh và ảnh so sánh cho thấy nhiều dự đoán biến mất. Tôi dừng trước vòng 2 để rà pipeline và nhãn thay vì tiếp tục huấn luyện trên kết quả đang suy giảm. Hai ứng viên để xem xét sau khi kiểm tra mô hình là `frame_0070.jpg` (rank 1 trong `selection_round2.csv`, score 0.601, 5 box/3 box mơ hồ) và `frame_0377.jpg` (rank 4, score 0.5571, 9 box/7 box mơ hồ). Frame thứ hai có nhiều box mơ hồ nên dự kiến tốn công rà hơn; mỗi frame cần kiểm tra từng xe và box. Không nên lấy thêm `frame_0069.jpg`/`frame_0074.jpg` sát `frame_0070.jpg`, hoặc `frame_0375.jpg`/`frame_0376.jpg` sát `frame_0377.jpg` nếu ảnh gần trùng; chọn một ảnh đại diện mỗi cụm trước.

Kết luận có giới hạn: test chỉ có 20 ảnh, 14 box rất nhỏ bị bỏ qua khi chấm, và 403 box tham chiếu được dùng chưa được người rà lại. Vì vậy AP50 là mức khớp với bộ tham chiếu, không phải chân lý tuyệt đối về chất lượng ngoài thực tế. Nếu AP50 giảm, trước tiên tôi sẽ xác nhận đúng gói `day8_data.zip`, kiểm tra 12 ảnh/nhãn YOLO và class id 0, bảo đảm test không bị đưa vào train và nhãn test không đổi; sau đó xem confidence và ảnh dự đoán để xác định mô hình bỏ sót hay tham chiếu có vấn đề. Chỉ cân nhắc vòng 2 sau khi các kiểm tra này giải thích được mức giảm.
