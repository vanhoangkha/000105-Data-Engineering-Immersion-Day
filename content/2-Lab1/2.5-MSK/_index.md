---
title : "Clickstream Analytics using MSK"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 2.5 </b> "
---

### Clickstream Analytics using MSK

Trong workshop này, mục tiêu chung của chúng ta là trực quan hóa và phân tích hiệu suất của các sản phẩm khác nhau trong trang web thương mại điện tử bằng cách nhập, chuyển đổi và phân tích dữ liệu luồng nhấp chuột theo thời gian thực bằng cách sử dụng dịch vụ AWS cho Apache Kafka (Amazon MSK), Apache Flink (Dữ liệu Kinesis Analytics cho các ứng dụng Java) và Elaticsearch (Amazon Elaticsearch). [Clickstream Analytics using MSK](https://amazonmsk-labs.workshop.aws/en/mskkdaflinklab/overview.html)  để tìm hiểu thêm.

Kiến trúc cấp cao sẽ trông như thế này:

![DeployCF](/WorkShopTwo/images/3.connect/32.png) 

Trước tiên, chúng ta sẽ sử dụng trình tạo dữ liệu để tạo thông báo Clickstream tới một topic (ExampleTopic) trong cụm Amazon MSK lưu trữ Apache kafka. Sau đó, chúng ta sẽ cấu hình và khởi động Kinesis Data Analytics cho Ứng dụng Java bằng cách sử dụng công cụ Apache Flink, một dịch vụ Apache Flink được quản lý từ AWS. Công cụ này sẽ đọc các sự kiện từ chủ đề exampleTopic trong Amazon MSK, xử lý và tổng hợp nó, sau đó gửi các sự kiện tổng hợp ( Analytics) cho cả hai topic trong Amazon MSK và Amazon Elasticsearch. Sau đó, chúng tôi sẽ sử dụng các thông báo đó từ Amazon MSK để minh họa cách người tiêu dùng có thể nhận được phân tích luồng nhấp chuột theo thời gian thực. Ngoài ra, chúng tôi sẽ tạo trực quan hóa Kibana và bảng điều khiển Kibana để trực quan hóa phân tích luồng nhấp chuột theo thời gian thực.

Chúng tôi sẽ cung cấp các tài nguyên cần thiết cho bài lab thông qua CloudFormation. 

Template sẽ trông như sau:
![DeployCF](/WorkShopTwo/images/3.connect/33.png) 

The stack sẽ gồm:
- A VPC với 1 Public subnet và 3 Private subnets
- 1 Public instance that hosts a schema registry service, a producer and consumer.
- 1 Amazon Elasticsearch cluster.
- 1 Amazon KDA for Java application.
- 1 Amazon MSK cluster.

Trong bài lab này, chúng ta sẽ ssh vào EC2 instance 

**KafkaClientEC2Instance**

[Clickstream Analytics using MSK](https://amazonmsk-labs.workshop.aws/en/mskkdaflinklab/overview.html) to follow complete lab.