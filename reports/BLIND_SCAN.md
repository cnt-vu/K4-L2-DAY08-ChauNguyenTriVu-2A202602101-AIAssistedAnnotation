# Quét độc lập trước khi xem pre-label

Frame: frame_0270.jpg

Số xe nhìn thấy bằng mắt: 27 xe thuộc nhóm từ 4 bánh trở lên.

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe:
1. Ở giữa ảnh, hơi chếch sang trái: một xe mô tô khá khó nhìn. Mô tô không thuộc lớp `car` của bài (chỉ gán nhãn xe từ 4 bánh trở lên), nên không tính vào số xe cần gán nhãn.
2. Gần cạnh trái ảnh: một ô tô bị một xe khác che khuất một phần; AI có thể bỏ sót hoặc chỉ vẽ khung quanh phần thân xe nhìn thấy.

Chạy `python tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
