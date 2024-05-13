---
title : "Lab. Phát hiện luồng click chuột bất thường bằng Amazon Managed Service for Apache Flink"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2 </b> "
---

### Giới thiệu
Dữ liệu truyền trực tuyến là dữ liệu được tạo ra liên tục bởi hàng nghìn nguồn dữ liệu, thường gửi các bản ghi dữ liệu đồng thời và ở kích thước nhỏ (Kilobyte). Dữ liệu truyền trực tuyến bao gồm nhiều loại dữ liệu như tệp nhật ký do khách hàng tạo bằng ứng dụng web hoặc thiết bị di động của bạn, mua hàng thương mại điện tử, hoạt động của người chơi trong trò chơi, thông tin từ mạng xã hội, sàn giao dịch tài chính hoặc dịch vụ không gian địa lý và đo từ xa từ các thiết bị được kết nối hoặc thiết bị trong trung tâm dữ liệu.

Dữ liệu này cần được xử lý tuần tự và tăng dần trên cơ sở từng bản ghi hoặc trên các cửa sổ thời gian trượt và được sử dụng cho nhiều loại phân tích bao gồm tương quan, tổng hợp, lọc và lấy mẫu. Thông tin thu được từ phân tích như vậy mang lại cho các công ty cái nhìn sâu sắc về nhiều khía cạnh trong hoạt động kinh doanh và khách hàng của họ, chẳng hạn như –việc sử dụng dịch vụ (để đo lường/thanh toán), hoạt động của máy chủ, số lần nhấp vào trang web và vị trí địa lý của thiết bị, con người và hàng hóa vật lý –và cho phép để họ có thể ứng phó kịp thời với những tình huống mới nổi. Ví dụ: doanh nghiệp có thể theo dõi những thay đổi trong cảm nhận của công chúng đối với thương hiệu và sản phẩm của họ bằng cách liên tục phân tích các luồng truyền thông xã hội và phản hồi kịp thời khi cần thiết.

AWS có rất nhiều công nghệ mạnh mẽ như Kinesis Data Streams, Kinesis Data Firehose, Amazon Managed Service cho Apache Flink và Managed Streaming cho Kafka khi làm việc với dữ liệu truyền trực tuyến. Trong phòng thí nghiệm này, chúng tôi sẽ đề cập đến một số dịch vụ chính này bằng các bài tập thực hành.

