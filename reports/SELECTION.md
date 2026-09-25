# Vì sao chọn lô này?

Nếu chỉ có ngân sách rà năm ảnh, tôi ưu tiên:

- `frame_0182.jpg` — rank 1, score 0.9591, 72.8 giây; 28 box dự đoán, trong đó 18 box mơ hồ. Đây là điểm cao nhất trong 50 dòng đầu và cần kiểm tra kỹ các xe ở cảnh đêm.
- `frame_0369.jpg` — rank 2, score 0.9324, 147.6 giây; 43 box, 16 box mơ hồ. Nhiều xe khiến việc rà nhãn có thể tốn công, nhưng cũng cho nhiều cơ hội phát hiện box thiếu hoặc chồng lấn.
- `frame_0326.jpg` — rank 4, score 0.9155, 130.4 giây; 39 box, 15 box mơ hồ. Ảnh thuộc một thời điểm khác nhóm 0369 và có nhiều xe/đèn cần phân biệt.
- `frame_0099.jpg` — rank 8, score 0.9063, 39.6 giây; 29 box, 14 box mơ hồ. Chọn để có thêm một đoạn thời gian xa các frame cuối video.
- `frame_0227.jpg` — rank 11, score 0.8915, 90.8 giây; 37 box, 14 box mơ hồ. Dù điểm thấp hơn một số frame lân cận, ảnh giúp phủ thêm thời điểm và có một xe bị AI bỏ sót theo nhật ký sửa nhãn.

Tôi không ưu tiên `frame_0187.jpg` (rank 10, score 0.8995, 74.8 giây) dù điểm cao, vì nó chỉ cách `frame_0182.jpg` 2 giây trong video camera cố định; hai ảnh có nguy cơ gần trùng. Tôi dành chỗ cho `frame_0227.jpg` để tăng độ phủ theo thời gian.

Ba frame trong lô được chọn và bằng chứng từ CSV/contact sheet: `frame_0182.jpg` (rank 1, score 0.9591, 72.8 giây), `frame_0369.jpg` (rank 2, score 0.9324, 147.6 giây) và `frame_0326.jpg` (rank 4, score 0.9155, 130.4 giây) đều có `selected=True`. Contact sheet cho thấy đây là các cảnh đường ban đêm có nhiều xe và đèn; `frame_0369.jpg` và `frame_0326.jpg` có nhiều vùng cần phân biệt giữa thân xe, đèn và phản sáng. Lô được chọn trải trên nhiều thời điểm, không chỉ gom vào một đoạn video.

Một frame điểm cao nhưng không chọn là `frame_0368.jpg`: rank 9, score 0.9003, thời điểm 147.2 giây. Nó cách `frame_0369.jpg` đã chọn chỉ 0.4 giây, nên bị loại bởi khoảng cách tối thiểu 2 giây để tránh rà các ảnh gần như trùng nhau.

Điểm số chỉ xếp hạng mức bất định, số box mơ hồ và độ đa dạng thời gian; nó không chứng minh nhãn của ảnh đúng, cũng không chứng minh huấn luyện trên ảnh đó sẽ cải thiện mô hình. Ảnh khó/nhòe có thể có điểm cao nhưng vẫn khó gán nhãn nhất quán; cần kiểm tra kết quả trên test và xem ảnh so sánh.
