---
title : "Option 3: Skip DMS Lab"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3.2. </b> "
---

### Steps
1. Mở AWS CloudShell
2. Copy data từ staging Amazon S3 bucket đến S3 bucket của chúng ta.
3. Kiểm chứng data

### Kiến trúc.
![DeployCF](/WorkShopTwo/images/3.connect/81.png) 

Cơ sở dữ liệu RDS Postgres được sử dụng làm nguồn bán vé cho các sự kiện thể thao. Nó lưu trữ thông tin giao dịch về giá bán vé cho những người được chọn và chuyển quyền sở hữu vé với các bảng bổ sung để biết chi tiết sự kiện. AWS Database Migration Service (DMS) được sử dụng để tải toàn bộ dữ liệu từ nguồn Amazon RDS sang bộ chứa Amazon S3.

Trước khi Glue lab bắt đầu, ta có thể chọn bỏ qua quá trình DMS data migration. Nếu vậy, hãy sao chép trực tiếp dữ liệu nguồn vào bộ chứa S3 của ta.

Trong bài lab hôm nay, ta sẽ sao chép dữ liệu từ bộ chứa S3 tập trung vào tài khoản AWS của mình, thu thập dữ liệu bằng trình thu thập thông tin AWS Glue để tạo siêu dữ liệu, chuyển đổi dữ liệu bằng AWS Glue, truy vấn dữ liệu và tạo Chế độ xem bằng Athena và cuối cùng là xây dựng dashboard với Amazon QuickSight.

### Open AWS CloudShell

Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/home?region=us-east-1)

### Copy data từ staging Amazon S3 bucket đến S3 bucket của chúng ta.

```
aws s3 cp --recursive --copy-props none s3://aws-dataengineering-day.workshop.aws/data/ s3://<YourBucketName>/tickets/
```

### Data sau đây sẽ được copy đến s3 bucket của chúng ta:
![DeployCF](/WorkShopTwo/images/3.connect/82.jpeg) 

### Xác minh dữ liệu
1. Mở S3 console và xem dữ liệu được copy từ CloudShell
![DeployCF](/WorkShopTwo/images/3.connect/822.png) 
2. Sử dụng S3 select để truy vấn s3 data

![DeployCF](/WorkShopTwo/images/3.connect/83.png) 

![DeployCF](/WorkShopTwo/images/3.connect/84.png) 

![DeployCF](/WorkShopTwo/images/3.connect/85.png) 

### Next Step.
Tiếp theo, Chúng ta sẽ sử  hoàn thành bài lab với AWS Glue
- [Extract, Transform and Load Data Lake with AWS Glue](https://catalog.us-east-1.prod.workshops.aws/workshops/976050cc-0606-4b23-b49f-ca7b8ac4b153/en-US/600.html)