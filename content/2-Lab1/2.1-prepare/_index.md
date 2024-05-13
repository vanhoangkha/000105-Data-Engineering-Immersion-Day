---
title : "Phát hiện luồng click chuột bất thường bằng Amazon Managed Service for Apache Flink"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

### Các bước thực hiện
- Giới thiệu
- Deploy CloudFormation Stack
- Thiết lập Amazon Kinesis Data Generator
- Thiết lập Email và SMS Subscription
- Xem xét AWS Lambda Anomaly function
- Phụ lục: CloudFormation Template


### Giới thiệu
Phần này sẽ là phần thiết lập môi trường cho bài lab.

Sau khi deploy CloudFormation template Kinesis_PreLab.yaml ta sẽ được kiến trúc sau:

![VisualizationData](/WorkShopTwo/images/1.png)

CloudFormation template sẽ tạo các resource sau:
- 2 S3 buckets: dùng để lưu trữ dữ liệu thô và dữ liệu đã được xử lý
- 1 Lambda function: Function này sẽ được trigger khi phát hiện bất thường.
- Amazon SNS topic: Lamda function sẽ publish tới topic này khi phát hiện click bất thường.
- Amazon Cognito User credentials: Sử  dụng để login vào Kinesis Data Generator để gửi bản ghi đến Kinesis Data Firehose.

### CloudFormation Stack Deployment
1. Click vào đây để deploy CloudFormation Stack:
{{< button href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=kinesis-pre-lab&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/Kinesis_PreLab.yaml" class="btn btn-white" >}}Deploy To AWS{{< /button >}}

Nhập các tham số vào trong form:

![DeployCF](/WorkShopTwo/images/2.prerequisite/20-workshop2.png) 

2. Điền các thông số trong form:
- **Username**: Username để đăng nhập vào Kinesis Data Generator
- **Password**: Mật khẩu để đăng nhập Kinesis Data Generator. 
- **Email**: Email để nhận thông báo. SNS topic sẽ gửi mail xác nhận.
- **SMS**: Số điện thoại nhận thông báo từ SNS.
- Ở phần cuối, check vào box marked "I acknowledge that AWS CloudFormation might create IAM resources".

3. Chọn Create. CloudFormation chuyển hướng bạn đến ngăn xếp hiện có của bạn. Sau vài phút, kinesis-pre-lab hiển thị trạng thái CREATE_COMPLETE.

![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/20-workshop3.png) 

4. Trong khi stack runs, ta sẽ nhân được email như sau:

![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-4.jpg) 

5. Xác nhập subscription
![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-5.jpg) 

6. Khi stack deployed, click Outputs để xem thêm thông tin:
- **KinesisDataGeneratorUrl**: Đây là Kinesis Data Generator (KDG) URL
- **RawBucketName**: Tên bucket lưu raw data từ Kinesis Data Generator (KDG)
- **ProcessedBucketName**: Tên bucket lưu transformed data

![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-6.jpg) 

**Bạn đã hoàn thành việc triển khai CloudFormation.**

### Khởi tạo Amazon Kinesis Data Generator (KDG)
Tại tab Outputs. Click vào Kinesis Data Generator URL.

KDG đơn giản hóa nhiệm vụ tạo dữ liệu và gửi dữ liệu tới Amazon Kinesis. Công cụ này cung cấp giao diện người dùng thân thiện với người dùng chạy trực tiếp trong trình duyệt của bạn. Với KDG, bạn có thể thực hiện các tác vụ sau:

- Tạo template cho bản ghi trong các trường hợp cụ thể .
- Tạo dữ liệu cho template với dữ liệu cố định hoặc dữ liệu ngẫu nhiên.
- Lưu lại template tương lai.
- Liên tục gửi hàng nghìn bản ghi mỗi giây tới Amazon Kinesis data stream hoặc Firehose delivery stream.

Kiểm tra Cognito User trong Kinesis Data Generator.

1. Click KinesisDataGeneratorUrl trong Outputs tab

![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-7.jpg) 

2. Đăng nhập bằng username và password nhập vào CloudFormation console

![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-81.png)

3. Sau khi đăng nhập, bạn sẽ thấy bảng điều khiển Kinesis Data Generator. Cần thiết lập một số  template để  giả lập dòng nhấp chuột

Tạo các template sau nhưng chưa nhấp vào Gửi dữ liệu, chúng ta sẽ thực hiện việc đó sau. Sao chép phần tô sáng tên tab bằng chữ in đậm và giá trị dưới dạng chuỗi JSON, tham khảo ảnh chụp màn hình:

```
Schema Discovery Payload
```

```
{"browseraction":"DiscoveryKinesisTest", "site": "yourwebsiteurl.domain.com"}
```

```
Click Payload
```

```
{"browseraction":"Click", "site": "yourwebsiteurl.domain.com"}
```

```
Impression Payload
```

```
{"browseraction":"Impression", "site": "yourwebsiteurl.domain.com"}
```

Amazon Kinesis Data Generator sẽ trông như thế này
![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/Workshop2-9.png)

### Xác thực email và SMS Subscription
1. Tại Amazon SNS, chọn Topics để xem.
![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/WS1.png)

2. Click vào topic để xem chi tiết:
![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/WS2.png)
### AWS Lambda function

3. Chọn vào Amazon Lamba để xem các lambda function mà CloudFormation đã tạo cho ta:
 ![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/WS3.png)

1. Chọn vào lambda function để xem chi tiết:
 ![DeployCFComplete](/WorkShopTwo/images/2.prerequisite/WS4.png)


Tại thời điểm này, chúng ta đã có tất cả các thành phần cần thiết để làm lab.
