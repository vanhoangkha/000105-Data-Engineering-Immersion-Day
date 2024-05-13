---
title : "Athena Federated query"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

[Federated query](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html) là một tính năng mới của Amazon Athena cho phép các nhà phân tích dữ liệu, kỹ sư và nhà khoa học dữ liệu thực hiện các truy vấn SQL trên dữ liệu được lưu trữ trong các nguồn dữ liệu quan hệ, không quan hệ, đối tượng và tùy chỉnh. Với Athena federated query, khách hàng có thể gửi một truy vấn SQL duy nhất và phân tích dữ liệu từ nhiều nguồn chạy tại chỗ hoặc được lưu trữ trên đám mây. Athena thực hiện federated queries bằng Data Source Connectors chạy trên [AWS Lambda](http://aws.amazon.com/lambda).

Trong bài này, chúng ta sẽ khám phá cách truy vấn các nguồn dữ liệu khác nhau từ Athena. Chúng tôi sẽ sử dụng cơ sở dữ liệu RDS hiện có được tạo như một phần của thiết lập ban đầu dưới dạng datasource-1 và data lake data (được lưu trữ trong S3) dưới dạng datasource-2.

![Athena](/WorkShopTwo/images/5.fwd/83.png) 


### Yêu cầu
Hoàn thành [Ingestion with DMS](../../3-IngestionwithDMS/_index.md) và [Transforming data with Glue ETL](../../4-TransformingdatawithGlue/_index.md) labs.

### Step

1. Tạo S3 endpoint để cho phép S3 truy cập vào Athena connector for Lambda, hãy làm theo các bước được đề cập [ở đây](https://docs.aws.amazon.com/glue/latest/dg/vpc-endpoints-s3.html) để tạo S3 endpoint. Đảm bảo sử dụng cùng subnet sẽ được sử dụng cho hàm Lambda trong bước tiếp theo.
2. Tại Athena console, chọn **"Data sources"**, **Create data source** button

![Athena](/WorkShopTwo/images/5.fwd/84.png) 

3. Chọn **PostgreSQL** làm data source, và nhấn **Next**
![Athena](/WorkShopTwo/images/5.fwd/85.png) 

4. Nhập **Data source name**: Postgres_DB

![Athena](/WorkShopTwo/images/5.fwd/86.png) 

5. Tại Connection detail, Chọn **‘Create Lambda function’** 

![Athena](/WorkShopTwo/images/5.fwd/87.png) 

6. Điền các thông tin như sau
|Field	|Value|
-------|:-------------:|
|Application Name|	AthenaJdbcConnector|
|SecretNamePrefix|	AthenaFed_|
|SpillBucket|	Get the BucketName from Workshop Studio Event Dashboard (e.g. dmslab-student-dmslabs3bucket-dpxexymdkhve)
|DefaultConnectionString	|postgres://jdbc:postgresql://<DATABASE_ENDPOINT>:5432/sportstickets?user=adminuser&password=admin123 → replace <DATABASE_EDNPOINT> with the correct database endpoint (e.g. dmslabinstance.abcdshic87yz.eu-west-1.rds.amazonaws.com)
|DisableSpillEncryption	|false
|LambdaFunctionName |	postgresqlconnector
|LambdaMemory |	3008
|LambdaTimeout |	900
|SecurityGroupIds |	Use the SecurityGroupId noted in prerequisites
|SpillPrefix |	athena-spill
|SubnetIds |	Use the SubnetId noted in prerequisites

![Athena](/WorkShopTwo/images/5.fwd/88.png) 
![Athena](/WorkShopTwo/images/5.fwd/89.png) 

8. Đợi function deploy, Chọn function ở ô **Select or enter a Lambda function**. Nhấn **Next**

![Athena](/WorkShopTwo/images/5.fwd/90.png) 

Nhấn **Create data source**

9. Cấu hình biến môi trường cho Lamda function.
Thêm một biến môi trường có tên là: Postgres_DB_connection_string và copy dữ liệu của biến **default** 

![Athena](/WorkShopTwo/images/5.fwd/91.png) 

![Athena](/WorkShopTwo/images/5.fwd/92.png) 

10. Xác thực data source đã được tạo ở Athena Console.
![Athena](/WorkShopTwo/images/5.fwd/93.png) 

11. Truy cập [query editor](https://console.aws.amazon.com/athena/home#/query-editor) chọn new datasource

![Athena](/WorkShopTwo/images/5.fwd/94.png) 

12. Truy vấn kết hợp sử dụng, **“sport_location”** tables của Postgres data source và **“parquet_sporting_event”** table từ data lake

Copy đoạn code sau và chạy:
```
SELECT loc.city, count(distinct evt.id) AS events
FROM "Postgres_DB"."dms_sample"."sport_location" AS loc
JOIN "AwsDataCatalog"."ticketdata"."parquet_sporting_event" AS evt
ON loc.id = evt.location_id
GROUP BY loc.city
ORDER BY loc.city ASC;
```

![Athena](/WorkShopTwo/images/5.fwd/95.png) 
