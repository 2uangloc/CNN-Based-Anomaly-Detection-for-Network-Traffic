4.1. Kịch bản thực nghiệm
Bước 1: thực hiện tấn công Ping of Death, SYN FLOOD, UDP FLOOD từ máy Attacker có địa chỉ IP là 192.168.8.10 vào máy Web Server có địa chỉ IP là 192.168.10.30.
![image](https://github.com/user-attachments/assets/946833c0-4885-4f29-a3a5-6c857756ca16)

Hình 4.1: Sơ đồ mô phỏng
Ping of Death là một kiểu tấn công mạng được thực hiện bằng cách sử dụng gói tin ping có kích thước lớn hơn giới hạn cho phép theo chuẩn ICMP.
SYN FLOOD là 1 dạng tấn công DoS hoặc DDoS. Attacker gửi một lượng lớn yêu cầu kết nối (SYN packets) đến một máy Web Server, nhưng không bao giờ hoàn thành quá trình thiết lập kết nối bằng cách không gửi các gói tin ACK để xác nhận kết nối.
UDP Flood là một loại tấn công mạng thuộc danh sách tấn công DoS hoặc DDoS. Attacker gửi một lượng lớn yêu cầu UDP đến máy Web Server, với mục đích làm quá tải và làm gián đoạn hoạt động của máy đó.


Bước 2: Thu thập log từ snort
 
Hình 4.2: Dữ liệu filelog thu thập từ snort
Sau khi Snort đã thu thập log, ta thực hiện tải log bằng giao thức SSH từ máy thật tới tường lửa pfSense.
 
Hình 4.3: Quá trình tải dữ liệu filelog về máy
Bước 3:  Sử dụng mô hình 1D – CNN để huấn luyện dữ liệu:
•	Đọc dữ liệu từ file log, tạo các feature để gán vào từng cột dữ liệu
•	Tiền xử lý dữ liệu
•	Chia dữ liệu thành tập train và test (80-20) tổng dữ liệu từ dataset là 121850
•	Tập train 80% (97480 dòng)
•	Tập test 20% (24370 dòng)
•	Sau khi huấn luyện dùng tập test để đánh giá mô hình dự đoán
•	Dùng mô hình đã huấn luyện để đưa ra dự đoán 
Bước 4: Đưa ra kết luận 
4.2. Triển khai thực nghiệm
4.2.1. Thêm các thư viện quan trọng
Import numpy as np
Import pandas as pd
Import matplotlib.pyplot as plt
From keras.layers Import Conv1D, Activation, MaxPooling1D, Flatten, GRU, AveragePooling1D, Dense, Dropout.
4.2.2. Tạo các thuộc tính cho dữ liệu log
Có tổng cộng 15 trường/thuộc tính: <Timestamp>, <GID>, < SID>, <Event Ref ID>, <Message>, <Protocol>, <Src IP>, <Src Port >, <Dst IP>, <Dst Port>, <Extra Port>, <Description>, <Priority>, <Event>, <Action>
 
Hình 4.4: Tạo thuộc tính giá trị
4.2.3. Đọc dữ liệu từ file log
Dùng thư viện panda để đọc file log dưới dạng CSV và in ra màn hình.
 
Hình 4.5: Đọc dữ liệu
 
Hình 4.6: Dữ liệu sau khi đọc
4.2.4. Chia và xóa các cột của dữ liệu
Chia cột nào có dạng IP thành 4 cột để thuận tiện cho việc nhân tích chập, đồng thời loại bỏ các cột không cần thiết như Timestamp, Src Ip, Dst IP và Action.
 
Hình 4.7: Chia cột và xóa cột dữ liệu
 
Hình 4.8: Dữ liệu sau khi được xử lý
4.2.5. Phân phối số lượng nhãn tấn công
Phân phối lại các lớp nhãn để thuận tiện cho việc mã hóa
 
Hình 4.9: Phân phối các lớp nhãn
4.2.6. Vẽ biểu đồ
Biểu đồ thể hiện số lượng tấn công, phần “UDP Flood” có số lượng nhiều nhất và “Normal” có số lượng ít nhất
 
Hình 4.10: Biểu đồ các loại tấn công
4.2.7. Thay đổi các giá trị NaN trong dữ liệu
Tính giá trị trung bình của hai cột “Src Port” và “Dst Port”. Sau đó gán giá trị cho mỗi dòng của dữ liệu.
 
Hình 4.11: Hàm tính trung bình trong cột dữ liệu “Src Port” và “Dst Port”
 
Hình 4. 12: Dữ liệu sau khi đã thay đổi
4.2.8. Xóa các cột có duy nhất một giá trị
Chỉ giữ các cột có nhiều hơn 1 giá trị do các giá trị giống nhau không cung cấp được các dữ liệu hữu ích trong quá trình huấn luyện.
 
Hình 4.13: Hàm xóa các cột có duy nhất một giá trị
Dữ liệu sau khi đã xóa các cột như: GID, Event Ref ID, Dst Port, Event.
 
Hình 4.14: Dữ liệu sau đã xử lý 
4.2.9. Tạo cột “Intrusion” và mã hóa dữ liệu
Thực hiện mã hóa nhãn cho cột “Message” trong dữ liệu data, sau đó tạo ra cột mới có tên là “Intrusion” để lưu giá trị được mã hóa.
 
Hình 4.15: Hàm mã hóa 
 
Hình 4.16: Dữ liệu sau khi mã hóa
4.2.10. Mã hóa cột có dữ liệu chữ thành số
Mã hóa các cột có định dạng chữ (Protocol) thành số để máy học dễ dàng hiểu được và thuận tiện cho việc nhân tích chập.
 
Hình 4.17: Dữ liệu sau khi đã mã hóa one-hot 
4.2.11. Chia dữ liệu thành 2 phần
Trước tiên ta biến đổi nhãn “Intrusion” thành mảng numpy và gán cho giá trị Y_data. Sau đó ta tạo mảng X_data bằng cách bỏ cột “Intrusion”. Điều này đang chuẩn bị cho các đặc trưng để huấn luyện mô hình. 
 
Hình 4.18: Chia dữ liệu thành hai phần
Tại đây ta thực hiện phân chia dữ liệu là 80% cho phần train và 20% còn lại cho phần test. random_state=42 đảm bảo rằng nếu ta chạy code này nhiều lần sẽ cho ra kết quả chia giống nhau.
 
Hình 4.19: Kết quả sau khi chia dữ liệu
4.2.12. Chuẩn hóa dữ liệu
Chuẩn hóa dữ liệu là bước cuối cùng trong tiền xử lý dữ liệu và cũng rất quan trọng, nhất là khi mô hình này phụ thuộc vào độ lớn của các đặc trưng. Chuẩn hóa đưa các giá trị đặc trưng về cùng một phạm vi, giúp mô hình học tốt hơn và hội tụ nhanh hơn.
 
Hình 4.20: Chuẩn hóa dữ liệu 
4.2.13. Xây dựng mô hình CNN1D
Ta khởi tạo mô hình bằng Sequential (Model = Sequential()), sau đó dùng phương thức add để thêm các layer. Sau khi đưa vào các lớp tạo thành dữ liệu khối, thực hiện duỗi dữ liệu thành vector model.add(Flatten()). Ta tiếp tục thực hiện biến đổi dữ liệu từ nhiều chiều thành 4 chiều model.add(Dense(4)).
 
Hình 4.21: Mô hình thuật toán CNN1D
Các thông số chi tiết của kết quả mô hình CNN1D:
•	conv1d (None, 13, 32): đây là lớp convolution 1D. None là batch size, 13 là chiều dài đầu ra và số lượng bộ lọc là 32. Param là 128.
•	max_pooling1d (None, 6, 32): là lớp max pooling 1D, giúp giảm kích thước của dữ liệu và đồng thời giữ lại thông tin quan trọng. None là batch size, 6 là chiều dài đầu ra và số lượng bộ lọc là 32. Param là 0 vì lớp này chỉ là lớp tuyến tính.
•	conv1d_1 (None, 4, 64): một lớp convolution 1D khác. None là batch size, 4 là chiều dài đầu ra và số lượng bộ lọc là 64. Param là 6208.
•	max_pooling1d_1 (None, 2, 64): một lớp max pooling 1D khác. None là batch size, 2 là chiều dài đầu ra và số lượng bộ lọc là 64. Param là 0.
•	flatten (None, 128: là lớp làm phẳng dữ liệu, chuyển dữ liệu thành vector 1 chiều. None là batch size, 128 là số lượng đơn vị đầu ra. Param là 0.
•	dense (None, 128): là lớp fully connected. None là batch size, 128 là số lượng đơn vị đầu ra. Param là 16512.
•	dropout (None, 128): là lớp dropout, giúp ngăn chặn overfitting. None là batch size, 128 là số lượng đơn vị đầu ra. Param là 0.
•	dense_1 (None, 4): là lớp fully connected với 4 đơn vị đầu ra. None là batch size, 4 là số lượng đơn vị đầu ra (hay còn gọi là nhãn). Param là 516.
•	Total params: là tổng tham số của mô hình, bao gồm cả trọng số và bias là 23364.
•	Trainable params: là tổng tham số có thể được cập nhật trong quá trình huấn luyện. Toàn bộ 23364 tham số đều được huấn luyện
•	Non-trainable params: là tổng số tham số không thể được cập nhật trong quá trình huấn luyện. Trong mô hình này không có tham số nào không được huấn luyện.

 
Hình 4.22: Kết quả của mô hình CNN1D
4.2.14. Biên dịch mô hình 
Quá trình huấn luyện sử dụng gradient descent để điều chỉnh trọng số của mô hình dựa trên giá trị mất mát (categorical_crossentropy), và thuật toán tối ưu hóa (Adam) được sử dụng để thực hiện điều này.
 
Hình 4.23: Biên dịch mô hình
4.2.15. Huấn luyện mô hình
Quá trình huấn luyện sẽ chạy qua 50 epochs, mỗi epoch sẽ được chia thành các batch có kích thước 32 mẫu. Mỗi lần qua một batch, mô hình sẽ điều chỉnh trọng số của nó dựa trên gradient của hàm mất mát (categorical crossentropy) và sử dụng thuật toán tối ưu hóa Adam. Hiệu suất của mô hình trên tập kiểm định sẽ được đánh giá sau mỗi epoch để theo dõi quá trình huấn luyện và tránh overfitting. Kết quả của quá trình huấn luyện được lưu vào biến his.
 
Hình 4.24: Huấn luyện mô hình
 
Hình 4.25: Một phần của kết quả của huấn luyện mô hình
4.2.16. Dự đoán mô hình
•	Kết quả cho thấy số lượng batch (762/762) đã được sử dụng trong quá trình huấn luyện. Trong mỗi epoch, mô hình được cập nhật dựa trên một số lượng batch của dữ liệu huấn luyện. 
•	Thời gian dự kiến hoàn thành mỗi epoch là 2 giây cho mỗi epoch (2s 2ms/step). 
•	Giá trị mất mát (loss) là 0.0019 và độ chính xác (accurary) là 0.9998 trên tập huấn luyện sau khi hoàn thành một epoch cuối cùng của quá trình huấn luyện.
•	Kết quả của tập kiểm thử là Loss: 0.001865588128566742 và Accurary: 99.97948408126831%
 
Hình 4.26: Dự đoán mô hình 









4.2.17. Biểu đồ thể hiện độ chính xác và mất mát của dữ liệu huấn luyện và kiểm thử
Độ chính xác của mô hình sau mỗi lần train có thể khác nhau, nhưng sẽ không có chênh lệch quá nhiều. Biểu đồ bên dưới cho thấy dấu hiệu tích cực về việc mô hình đang học được và có khả năng tổng quát hóa tốt. Điều này đảm bảo mô hình không chỉ nhớ trên tập dữ liệu huấn luyện mà còn có khả năng dự đoán chính xác trên tập dữ liệu mới. Đồng thời giúp tránh bị Overfitting.
 
Hình 4.27 Biểu đồ thể hiện sự chính xác của tập dữ liệu huấn luyện và kiểm thử




Biểu đồ bên dưới thể hiện rằng, ban đầu loss của tập huấn luyện giảm rất nhanh, sau 5 epoch thì không giảm nữa. So với loss của tập kiểm thử thì không chênh lệch quá nhiều cho thấy đó là dấu hiệu tích cực như sau:
•	Khả năng tổng quát tốt hơn: Mô hình không chỉ học được từ dữ liệu huấn luyện mà còn có khả năng áp dụng kiến thức đã học được cho dữ liệu mới.
•	Tránh Overfitting: Overfitting xảy ra khi mô hình quá tinh chỉnh cho dữ liệu huấn luyện và không thể tổng quát tốt cho dữ liệu mới. Nếu loss trên tập kiểm thử tăng lên mạnh so với loss trên tập huấn luyện, có thể là dấu hiệu của overfitting.
 
Hình 4.28 Biểu đồ thể hiện sự mất mát của tập dữ liệu huấn luyện và kiểm thử
 
CHƯƠNG 5. KẾT LUẬN
5.1. KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN
Đề tài “nghiên cứu và triển khai thuật toán máy học ứng dụng dò tìm lưu lượng mạng bất thường” đã gợi ý cho nhóm một mô hình huấn luyện dữ liệu để dòm tìm đó là 1D-CNN. Mô hình 1D-CNN có thể xây dựng được các hệ thống phân loại với độ chính xác cao và được lấy cảm hứng từ tế bào thần kinh trong bộ não con người. Cho thấy 1D-CNN là mô hình được sử dụng để nâng cao hiệu quả dự báo và đã được ứng dụng rộng rãi trong đa lĩnh vực. Do đó, nhóm có đề xuất hướng phát triển dựa trên mô hình 1D-CNN đã thực hiện như sau:

Phần đã làm được	Hướng phát triển
1.	Import các thư viện deep learning cần thiết trong Python để thực hiện huấn luyện dữ liệu bằng mô hình 1D-CNN
2.	Tiền xử lý dữ liệu để tạo bộ dataset
3.	Huấn luyện một dataset 121850 dòng thông qua mô hình 1D-CNN, thực hiện được đánh giá độ chính xác tốt hơn dự đoán.	1.	Cải tiến file log: Lấy thêm dữ liệu cho mô hình học vì hiện tại dữ liệu vẫn còn khá ít chỉ ở mức vài nghìn, nên trong tương lai sẽ cho mô hình học nhiều hơn bằng các lấy thêm file log lên mức vài triệu dữ liệu hoặc nhiều hơn nữa. 
2.	Cải tiến mô hình 1D-CNN: Tiếp tục thử nghiệm chỉnh sửa mô hình bằng cách thiết kế lại các lớp, thêm các lớp Conv, đảm bảo đạt kết quả chính xác hơn trong tương lai.
3.	Phát triển tải trực tiếp log mà không cần phải tải thủ công.
