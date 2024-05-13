---
title : "Lab. Streaming ETL with Glue "
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

### Giới thiệu
Trong bài lab này, ta sẽ tìm hiểu cách nhập, xử lý và sử dụng streaming data  bằng các dịch vụ serverless của AWS như Kinesis Data Streams, Glue, S3 và Athena. Để mô phỏng đầu vào truyền dữ liệu, chúng tôi sẽ sử dụng Kinesis Data Generator (KDG).


![DeployCF](/WorkShopTwo/images/3.connect/1.png) 

### Setup môi trường

Sử dụng [DMS Lab Student PreLab CloudFormation](https://catalog.us-east-1.prod.workshops.aws/workshops/976050cc-0606-4b23-b49f-ca7b8ac4b153/en-US/400/401/420-pre-lab-2.html) để thiết lập môi trường cơ sở hạ tầng hội thảo cốt lõi của bạn. Bỏ qua PreLab tương tự trong phần DMS. Nhấp vào biểu tượng Triển khai lên AWS bên dưới:

{{< button href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=dmslab-student&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSlab_student_CFN.yaml" class="btn btn-white" >}}Deploy To AWS{{< /button >}}


### Set up  kinesis stream

1. Mở [AWS Kinesis console](https://console.aws.amazon.com/kinesis/home)
2. Chọn “Create data stream”
![DeployCF](/WorkShopTwo/images/3.connect/2.png) 
3. Nhập số liệu như sau:
- Data stream name: TicketTransactionStreamingData
- Capacity mode: Provisioned
- Provisioned shards: 2

![DeployCF](/WorkShopTwo/images/3.connect/3.png) 

4. Chọn Create data stream

### Create Table for Kinesis Stream Source in Glue Data Catalog

1. Mở tab [AWS Glue console](https://console.aws.amazon.com/glue/home)
2. Tạo database có tên là "tickettransactiondatabase"
![DeployCF](/WorkShopTwo/images/3.connect/4.png) 
3. Tạo tables có tên là "TicketTransactionStreamData" ở trong database "tickettransactiondatabase"
![DeployCF](/WorkShopTwo/images/3.connect/6.png) 
4. Chọn Kinesis làm nguồn, chọn Luồng trong my account để chọn luồng dữ liệu Kinesis, chọn khu vực AWS thích hợp nơi bạn đã tạo luồng, chọn tên luồng là TicketTransactionStreamingData từ danh sách thả xuống, chọn JSON làm định dạng dữ liệu đến, vì chúng ta sẽ gửi JSON payloads từ Kinesis Data Generator theo các bước sau. và nhấp vào Tiếp theo.
![DeployCF](/WorkShopTwo/images/3.connect/5.png) 

5. Để trống schema vì chúng ta sẽ bật tính năng schema detection. Để trống partition indices. Chọn Next
![DeployCF](/WorkShopTwo/images/3.connect/7.png) 
6. Review lại tất cả thông tin và nhấn Create

![DeployCF](/WorkShopTwo/images/3.connect/8.png) 

7. Chọn vào Table để xem các thuộc tính

![DeployCF](/WorkShopTwo/images/3.connect/9.png) 

### Tạo và trigger Glue Streaming job

1. Tại mục Data Integration and ETL chọn Glue Studio

![DeployCF](/WorkShopTwo/images/3.connect/10.png) 

2. Chọn Visual with a blank canvas và nhấn Create
![DeployCF](/WorkShopTwo/images/3.connect/11.png) 

3. Chọn Amazon Kinesis từ Source drop down
![DeployCF](/WorkShopTwo/images/3.connect/12.png) 

4. Trong bảng bên phải phía dưới “Data source properties - Kinesis Stream”, cấu hình như sau:
- Amazon Kinesis Source: Data Catalog table
- Database: tickettransactiondatabase
- Table: tickettransactionstreamdata
- Đảm bảo rằng Detect schema được chọn
- Để tất cả còn lại mặc định

![DeployCF](/WorkShopTwo/images/3.connect/13.png) 

5. Chọn Amazon S3 từ target drop down list

![DeployCF](/WorkShopTwo/images/3.connect/14.png) 

6. Chọn **Data target - S3 bucket** và cấu hình như sau:
- Format: Parquet
- Compression Type: None
- S3 Target Location: Select Browse S3 and select the “mod-xxx-dmslabs3bucket-xxx” bucket
![DeployCF](/WorkShopTwo/images/3.connect/15.png) 

7. Cuối cùng chọn **Job details** tab và cấu hình theo như sau:
- Name: TicketTransactionStreamingJob
- IAM Role: Select the xxx-GlueLabRole-xxx from the drop down list
- Type: Spark Streaming

8. Nhấn Save button để tạo job

9. Khi thấy **Successfully created job** ta nhấn **Run** button để start job

### Trigger streaming data từ Kinesis Data Generator
1. Truy cập Kinesis Data Generator url từ tab setup và đăng nhập.
2. Đảm bảo chọn đúng region. Chọn TicketTransactionStreamingData là target Kinesis stream để  Records per second mặc định (100 records per second). Đối với template, nhập NormalTransaction, copy và dán template payload như sau:

```
{
    "customerId": "{{random.number(50)}}",
    "transactionAmount": {{random.number(
        {
            "min":10,
            "max":150
        }
    )}},
    "sourceIp" : "{{internet.ip}}",
    "status": "{{random.weightedArrayElement({
        "weights" : [0.8,0.1,0.1],
        "data": ["OK","FAIL","PENDING"]
        }        
    )}}",
   "transactionTime": "{{date.now}}"      
}
```

![DeployCF](/WorkShopTwo/images/3.connect/16.png) 

3. Click **Send data** để trigger transaction streaming data.

### Tạo Glue Crawler để transformed data

1. Truy cập [ AWS Glue console ](https://console.aws.amazon.com/glue/home)

2. Tại AWS Glue menu, chọn Crawlers and click Add crawler

![DeployCF](/WorkShopTwo/images/3.connect/17.png) 

3. Nhập tên crawler là  TicketTransactionParquetDataCrawler, nhấn Next

![DeployCF](/WorkShopTwo/images/3.connect/18.png) 

4. Click vào Add a datasource
![DeployCF](/WorkShopTwo/images/3.connect/19.png) 

5. Chọn S3 và chỉ định path
![DeployCF](/WorkShopTwo/images/3.connect/20.png) 

6. Sau khi thêm datasource, nhấn next

![DeployCF](/WorkShopTwo/images/3.connect/21.png) 

7. Chọn IAM Role và nhấn Next

![DeployCF](/WorkShopTwo/images/3.connect/22.png) 

8. Chọn prefix là parquet_ cho tables

![DeployCF](/WorkShopTwo/images/3.connect/23.png) 

9. Đăt Crawler Schedule chạy mỗi giờ.
![DeployCF](/WorkShopTwo/images/3.connect/24.png) 
10. Review lại  Crawler và Click Create để tạo Crawler
![DeployCF](/WorkShopTwo/images/3.connect/25.png) 

11. Sau khi Crawler tạo xong. Nhấn Run crawler để trigger lần đầu.

![DeployCF](/WorkShopTwo/images/3.connect/26.png) 

12. Khi crawler job stop, chuyển đến Glue Data catalog. Đảm bảo rằng parquet_tickettransactionstreamingdata table xuất hiện

![DeployCF](/WorkShopTwo/images/3.connect/27.png) 

13. Click vào parquet_tickettransactionstreamingdata table để xem chi tiết
![DeployCF](/WorkShopTwo/images/3.connect/28.png) 

### Trigger dữ liệu bất thường từ Kinesis Data Generator(KDG)

1. Mở Kinesis Data Generator, chọn đúng region. Chọn TicketTransactionStreamingData là Kinesis stream đích 
![DeployCF](/WorkShopTwo/images/3.connect/29.png) 

2. Template cho record

```
{
    "customerId": "{{random.number(50)}}",
    "transactionAmount": {{random.number(
        {
            "min":10,
            "max":150
        }
    )}},
    "sourceIp" : "221.233.116.256",
    "status": "{{random.weightedArrayElement({
        "weights" : [0.8,0.1,0.1],
        "data": ["OK","FAIL","PENDING"]
        }        
    )}}",
   "transactionTime": "{{date.now}}"      
}
```

![DeployCF](/WorkShopTwo/images/3.connect/30.png) 

3. Click send data

### Sử dụng Athena để truy vấn dữ liệu
1. Mở [ AWS Athena console ](https://console.aws.amazon.com/athena/home)
2. Chọn AwsDataCatalog làm data source và tickettransactiondatabase là database 
![DeployCF](/WorkShopTwo/images/3.connect/31.png) 

3. Sử dụng các truy vấn sau để xem dữ liệu
```
SELECT count(*) as numberOfTransactions, sourceip
FROM "tickettransactiondatabase"."parquet_tickettransactionstreamingdata" 
WHERE ingest_year='2024'
AND cast(ingest_year as bigint)=year(now())
AND cast(ingest_month as bigint)=month(now())
AND cast(ingest_day as bigint)=day_of_month(now())
AND cast(ingest_hour as bigint)=hour(now())
GROUP BY sourceip
Order by numberOfTransactions DESC;

```
