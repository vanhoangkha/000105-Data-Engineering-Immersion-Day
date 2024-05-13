---
title : " Option1: DMS Migration Lab"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

## DMS Migration Lab

### Steps

1. Giới thiệu
2. Tạo Subnet Group
3. Tạo Replication Instance
4. Tạo DMS Source Endpoint
5. Tạo Target Endpoint
6. Tạo 1 task để bắt đầu full copy
7. Tạo CDC endpoint để replicate thay đổi
8. Tạo task để replicate liên tục


### Giới thiệu
Bài lab này sẽ giúp bạn hiểu rõ hơn về  AWS Database Migration Service(AWS DMS). Bạn sẽ di chuyển dữ liệu từ cơ sở dữ liệu Amazon Relational Database Service (Amazon RDS) Postgres hiện có sang Amazon Simple Storage Service (Amazon S3) bucket.

![DeployCF](/WorkShopTwo/images/3.connect/34.png) 

Link GitHub của lab - [https://github.com/aws-samples/data-engineering-for-aws-immersion-day](https://github.com/aws-samples/data-engineering-for-aws-immersion-day)

### Tạo Subnet group

1. Tại [ DMS console ](https://console.aws.amazon.com/dms/v2/home#createSubnetGroup), chọn  Subnet Groups và Create subnet group.
![DeployCF](/WorkShopTwo/images/3.connect/35.png) 
- Tại Name textbox: dms-lab-subnet-grp
- Description textbox: Replication instance for production data system
- VPC: Chọn *-dmslstudv1
- Chọn subnets và click Add

![DeployCF](/WorkShopTwo/images/3.connect/36.png) 

2. Chọn Create subnet group

3. Tại DMS console, subnet group displays **Complete**
![DeployCF](/WorkShopTwo/images/3.connect/37.png) 

### Tạo Replication Instance

1. Tại DMS console, chọn [ Replication instances ](https://console.aws.amazon.com/dms/v2/home#createReplicationInstance) để tạo  replication instance mới.
![DeployCF](/WorkShopTwo/images/3.connect/38.png) 
- Name: DMS-Replication-Instance
- Description: DMS Replication Instance
- Instance class: dms.t3.medium
- Chọn engine version mới nhất
- High Availability: Dev or test workload (Single-AZ)
- VPC: dmslstudv1
![DeployCF](/WorkShopTwo/images/3.connect/39.png) 

- Chọn Advanced để mở rộng
- Chọn security group là sgdefault
![DeployCF](/WorkShopTwo/images/3.connect/40.png) 

2. Tất cả các trường còn lại mặc định

3. DMS console hiển thị trạng thái tạo instance
![DeployCF](/WorkShopTwo/images/3.connect/41.png) 


### Tạo MDS Source Endpoint

1. Tại DMS console, chọn Endpoints để tạo source [Endpoint](https://console.aws.amazon.com/dms/v2/home#createNewEndpoint)
![DeployCF](/WorkShopTwo/images/3.connect/42.png) 

- Chọn Source Endpoint
- Endpoint identifier: rds-source-endpoint
- Source engine: PostgreSQL
- Access to Endpoint database: Provide access information manually
- Server name: RDS-Server-Name
![DeployCF](/WorkShopTwo/images/3.connect/43.png) 

- Port: 5432
- SSL mode: none
- User name: adminuser
- Password: admin123
- Database name: sportstickets

2. Tất cả còn lại để mặc định, rồi click tạo endpoint. Khi sẵn sàng, trạng thái sẽ chuyển sang active
3. Kiểm tra lại replication instance

![DeployCF](/WorkShopTwo/images/3.connect/45.png) 

4. Chọn source endpoint và nhấn Test connection

![DeployCF](/WorkShopTwo/images/3.connect/46.png) 

5. Click Run test. Nếu thành công sẽ có thông báo "Connection tested successfully" xuất hiện.

![DeployCF](/WorkShopTwo/images/3.connect/47.png) 

### Tạo Target Endpoint

1. Tại DMS console, Chọn Endpoint để tạo target [Endpoint](https://console.aws.amazon.com/dms/v2/home#createNewEndpoint)
![DeployCF](/WorkShopTwo/images/3.connect/48.png) 
- Endpoint type: Target endpoint
- Endpoint identifier: s3-target-endpoint
- Target engine: Amazon S3
- Service access role ARN: Copy và Past DMSLabRoleS3 ARN
- Bucket name: paste S3 Bucket Name
- Bucket folder: tickets

![DeployCF](/WorkShopTwo/images/3.connect/49.png) 

- Mở rộng phần: Endpoint settings
- Chọn Use endpoint connection checkbox, điền addColumnName=true trong Extra connection attributes box
![DeployCF](/WorkShopTwo/images/3.connect/50.png) 

- Mở rộng Test endpoint connection (optional). chọn VPC.

- Chọn Run test. Bước này kết nối với source database. Nếu thành công sẽ hiển thị thông báo "Connection tested successfully"
![DeployCF](/WorkShopTwo/images/3.connect/51.png) 

2. Chọn Create Endpoint. Khi đã sẵn sàng, trạng thái endpoint sẽ chuyển thành active

### Tạo task initial full copy
1. Tại DMS console, chọn Database Migration Tasks.
![DeployCF](/WorkShopTwo/images/3.connect/52.png) 

2. Chọn Create Task.

- Task name: dms-full-dump-task
- Chọn Replication instance
- Chọn Source endpoint
- Chọn Target endpoint
- Migration type: Migrate existing data.

![DeployCF](/WorkShopTwo/images/3.connect/53.png) 

- Mở rộng **Task Settings**
- Chọn **Turn on CloudWatch logs** checkbox

![DeployCF](/WorkShopTwo/images/3.connect/54.png) 

- Tại Table Mappings
- Chọn **Add new selection rule** và chọn **Enter a Schema** tại Schema field
- Tại **Source name**: dms_sample
- Để tất cả field còn lại mặc định.
![DeployCF](/WorkShopTwo/images/3.connect/55.png) 

3. Chọn **Create task**. Task sẽ được tạo và tự động start

4. Chọn task và xem chi tiết.
![DeployCF](/WorkShopTwo/images/3.connect/56.png) 

![DeployCF](/WorkShopTwo/images/3.connect/57.png) 

5. Khi hoàn thành, task console hiển thị 100% progress
![DeployCF](/WorkShopTwo/images/3.connect/58.png) 

6. Mở  S3 console và xem data được copy bởi DMS
![DeployCF](/WorkShopTwo/images/3.connect/59.png) 

7. Review data bằng S3 select

![DeployCF](/WorkShopTwo/images/3.connect/60.png) 

![DeployCF](/WorkShopTwo/images/3.connect/61.png) 


![DeployCF](/WorkShopTwo/images/3.connect/62.png) 

### Tạo CDC endpoint để replicate các thay đổi diễn ra

1. Tại DMS console, chọn [Endpoints](https://console.aws.amazon.com/dms/v2/home#createNewEndpoint)
![DeployCF](/WorkShopTwo/images/3.connect/64.png) 
 
2. Nhấn **Create endpoint**
- Endpoint type: Target
- Endpoint identifier: rds-cdc-endpoint
- Target engine: Amazon S3
- Service Access Role ARN: DMSLabRoleS3
- Bucket name: Chọn S3 Bucketname
- Bucket folder: cdc
![DeployCF](/WorkShopTwo/images/3.connect/65.png) 

- Mở rộng phần **Endpoint settings**

- Tích vào checkbox **Use endpoint connection attributes** và nhập addColumnName=true. Thuộc tính này bao gồm tên cột từ dữ liệu nguồn.
![DeployCF](/WorkShopTwo/images/3.connect/66.png) 

- Mở rộng phần **Test endpoint connection (optional)**, chọn VPC name.
- Click **Run test**. Nếu thành công sẽ hiển thị thông báo "Connection tested successfully".


3. Chọn **Create endpoint**
![DeployCF](/WorkShopTwo/images/3.connect/67.png) 

4. Khi sẵn sàng, endpoint status chuyển sang active.

![DeployCF](/WorkShopTwo/images/3.connect/68.png) 

### Tạo task replication liên tục.
1. Tại DMS console, chọn **Database Migration Tasks**
![DeployCF](/WorkShopTwo/images/3.connect/69.png) 
2. Chọn **Create Task**
- Task Identifier: cdctask
- Chọn Replication instance
- Chọn Source endpoint
- Chọn Target endpoint: rds-cdc-endpoint
- Chọn Migration type: **Replicate data changes only.**

![DeployCF](/WorkShopTwo/images/3.connect/70.png) 

- Trong **Task Settings**, Chọn **Turn on CloudWatch logs** checkbox

![DeployCF](/WorkShopTwo/images/3.connect/71.png) 

- Chuyển đến **Table Mappings**

- Chọn ** Add new selection rule ** và Chọn **Enter a Schema** tại Schema field

- Tại **Source name**, chọn dms_sample. Tất cả còn lại để mặc định
![DeployCF](/WorkShopTwo/images/3.connect/72.png) 

3. Chọn Create task. Task sẽ được tạo và tự động chạy. Chúng ta có thể thấy trạng thái là Replication ongoing.
![DeployCF](/WorkShopTwo/images/3.connect/73.png) 

4. Đợi 5 đến 10 phút để CDC data ánh xạ RDS postgres database

5. Chọn CDC task để xem chi tiết, xem phần **Table statistics**:
![DeployCF](/WorkShopTwo/images/3.connect/74.png) 

6. Mở S3 console và xem CDC data được copied từ DMS

![DeployCF](/WorkShopTwo/images/3.connect/75.png) 

7. Chọn 1 file và sử dụng [ S3 Select ](https://docs.aws.amazon.com/AmazonS3/latest/userguide/selecting-content-from-objects.html)

![DeployCF](/WorkShopTwo/images/3.connect/76.png) 

![DeployCF](/WorkShopTwo/images/3.connect/77.png) 

![DeployCF](/WorkShopTwo/images/3.connect/78.png) 
