---
title : "Lab"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

### Giới thiệu

Hướng dẫn này giúp bạn hoàn thành lab "Phát hiện luồng click chuột bất thường bằng Amazon Managed Service for Apache Flink"

Phân tích lưu lượng truy cập web để hiểu rõ hơn nhằm thúc đẩy các quyết định kinh doanh trước đây được thực hiện bằng cách sử dụng xử lý hàng loạt. Mặc dù hiệu quả nhưng cách tiếp cận này dẫn đến phản ứng chậm trễ đối với các xu hướng mới nổi và hoạt động của người dùng. Có những giải pháp xử lý dữ liệu trong thời gian thực bằng cách sử dụng công nghệ streaming và micro-batching, nhưng việc thiết lập và bảo trì có thể phức tạp. Amazon Managed Service dành cho Apache Flink là dịch vụ được quản lý giúp dễ dàng xác định và phản hồi các thay đổi về hành vi dữ liệu trong thời gian thực.

#### Steps:
- Setup Amazon Analytics Studio Application thông qua CloudFormation stack deployment
- Tạo real time website traffic sử dụng Amazon Kinesis Data Generator (KDG)
- Thực hiện real-time Data Analytics
- Dọn dẹp môi trường
- Phụ lục: Các tập lệnh

Trong Kinesis prelab setup, bạn đã hoàn thành các điều kiện tiên quyết cho bài lab này. Trong bài lab, bạn sẽ tạo quy trình Amazon Managed Service cho Apache Flink.

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS5.png) 

### Set up Amazon Analytics Studio Application thông qua CloudFormation stack deployment

1. Click vào đây để deploy CloudFormation Stack:
{{< button href="https://console.aws.amazon.com/cloudformation/home?#/stacks/quickcreate?templateURL=https://aws-bigdata-blog.s3.amazonaws.com/DE-ID-KDA-Lab/1-Lab-Kinesis-Clickstream_CFN.yaml&stackName=kda-flink-pre-lab&param_FlinkVersion=1.15.4&param_Release=master&param_GlueDatabaseName=prelab_kda_db" class="btn btn-white" >}}Deploy To AWS{{< /button >}}

2. Điền các tham số , chọn IAM và check vào box "I acknowledge that AWS CloudFormation might create IAM resources."

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS9.png) 

3. Stack Sex taoj ra 6 Amazon Kinesis Data Streams trong Amazon Kinesis Console

- tickerstream – Stream khởi tạo traffic

- clickstream – Nắm bắt số lượng nhấp chuột

- impressionstream – Số lần hiển thị

- ctrstream – bắt tỷ lệ nhấp được tính toán

- destinationstream – nắm bắt được điểm số bất thường

- anomalydetectionstream – ghi lại các bản ghi có điểm bất thường lớn hơn 2

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS10.png) 

4. CloudFormation Stack cũng sẽ tạo một ứng dụng Amazon Analytics Studio có tên là kda-flink-prelab-RealtimeApplicationNotebook trong tab Amazon Kinesis Application Console. Chúng ta sẽ viết Studio Notebook tương tác trong Apache Zeppelin để phân tích dữ liệu theo thời gian thực.

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS11.png) 

5. Chạy Ứng dụng Studio bằng cách chọn kda-flink-prelab-RealtimeApplicationNotebook trong tab Studio. Chọn “Run” lần nữa trên màn hình tiếp theo.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS12.png) 

### Tạo lưu lượng truy cập trang web theo thời gian thực bằng Amazon Kinesis Data Generator (KDG)
1. Mở output  Amazon CloudFormation console click vào link Amazon Kinesis Data Generator
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS13.png) 

2. Bắt đầu gửi traffic
```  
{"browseraction":"Impression", "site": "https://www.mysite.com"}
```
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS14.png) 

```
  {"browseraction":"Click", "site": "https://www.mysite.com"}
```

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS15.png) 

Bạn có thể xem số lượng được gửi đến data stream

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS16a.png) 

Sau 30 giây thì dừng

### Thực hiện phân tích dữ liệu thời gian thực
1. Mở   Amazon Kinesis Application Console, chọn kda-prelab-template-RealtimeApplicationNotebook. Chọn “Open in Apache Zeppelin”.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS16.png) 

2. Tại Apache Zeppelin Console, chọn Create new note. Tên notebook là kda_Interactive_notebook
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS18.png) 
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS19.png) 

3. Thực hiện phân tích tương tác theo thời gian thực với luồng dữ liệu Kinesis.
- Tạo bảng Flink bằng Truy vấn SQL Flink

- Sử dụng truy vấn Flink SQL để chuyển đổi và tạo luồng dữ liệu mới trong thời gian thực

- Thực hiện phát hiện bất thường bằng Chức năng do người dùng xác định Flink và kích hoạt email thông báo bất thường trong thời gian thực.

- Các script [ở đây](https://aws-bigdata-blog.s3.amazonaws.com/DE-ID-KDA-Lab/kda_notebook_steps.docx).

- Notebook cũng có [tại đây](https://aws-bigdata-blog.s3.amazonaws.com/DE-ID-KDA-Lab/kda_realtime_inetractive_streaming_notebook_2J9H888TM.zpln). Bạn có thể tải về và imported thông qua Apache Zeppelin console.

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS19aa.png) 

- Sau đó, bạn có thể mở notebook và chạy từng đoạn một.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS19bb.png) 

- Sơ đồ luồng
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS19c.png) 

### Chạy Apache Zeppelin

1. Chạy các scripts tạo bảng

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS20.png) 

2. User Defined Function (UDF) thực hiện Phát hiện bất thường trong thời gian thực bằng thuật toán Random Cut Forest algorithm

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS22.png) 

3. Bạn có thể xem dữ liệu thời gian thực từ các lượt truy cập trang web bằng cách chạy truy vấn ở Bước #3
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS23.png) 

4. Tạo impressionstream bằng cách lọc messages từ tickerstream

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS24.png)

5. Tạo clickstream bằng cách lọc messages từ tickerstream

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS25.png)

6. Tính toán Tỷ lệ nhấp (CTR) và điền vào ctrstream.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS26.png)

7. Bạn có thể xem Tỷ lệ nhấp trong thời gian thực bằng cách thực hiện Bước 7.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS27.png)

8. Sử dụng UDF (RRandom Cut Forest) để tạo điểm bất thường.
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS28.png)
9. Populate anomalydetectionstream bằng cách thực hiện step 9
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS29.png) 
10. Bây giờ hãy kiểm tra điểm bất thường từ thuật toán Random Cut Forest trong thời gian thực:
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS30.png)

11. Bạn sẽ bắt đầu nhận được thông báo trong email của mình khi phát hiện thấy sự bất thường:
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS31.png)

12. Nếu bạn không nhận được email thông báo bất thường trong lần thử đầu tiên

- Mở lại hai phiên đồng thời của KDG UI trong trình duyệt của bạn.

- Trong phiên đầu tiên, gửi tin nhắn hiển thị với tốc độ một tin nhắn mỗi giây đến dòng mã đánh dấu, nội dung tin nhắn là
```
{"browseraction":"Impression", "trang web":https://www.mysite.com"}
```

- Trong phiên thứ hai, gửi tin nhắn nhấp chuột với tốc độ năm tin nhắn mỗi giây đến dòng mã đánh dấu, nội dung tin nhắn là
```
{"browseraction">Click", "trang web":https://www.mysite.com"}
```

- Dừng gửi tin nhắn sau 30-40 giây.

- Bây giờ trên Sổ ghi chép Apache Zeppelin, hãy lặp lại các bước từ 3 đến 10 và bạn sẽ bắt đầu nhận được thông báo qua email từ lần thử thứ hai.

### Dọn dẹp tài nguyên

1. Xong khi hoàn thành bài lab. Delete kda-flink-pre-lab

![DeployCF](/WorkShopTwo/images/2.prerequisite/WS32.png)
![DeployCF](/WorkShopTwo/images/2.prerequisite/WS33.png)
