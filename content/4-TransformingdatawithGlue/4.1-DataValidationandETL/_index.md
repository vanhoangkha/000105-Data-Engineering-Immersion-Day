---
title : "Data Validation and ETL"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

### PART A: Tạo Glue Crawler để load full data
1. Truy cập [ AWS Glue Console ](https://console.aws.amazon.com/glue/home)

![DeployCF](/WorkShopTwo/images/4.s3/1.png) 

2. Tại  AWS Glue menu, chọn **Crawlers**
![DeployCF](/WorkShopTwo/images/4.s3/2.png) 

3. Chọn **Create crawler**.

4. Nhập glue-lab-crawler làm crawler name để init load
5. Có thể nhập hoặc không description và nhấn next.
![DeployCF](/WorkShopTwo/images/4.s3/3.png) 

6. Chọn **Not yet** và **Add a data source**

![DeployCF](/WorkShopTwo/images/4.s3/4.png) 

7. Chọn S3 làm data source, **S3 Path** là path chứa CSV files từ bài lab DMS, tất cả các tham số còn lại để default và nhấn **Add an S3 data source**
8. Chọn Next
![DeployCF](/WorkShopTwo/images/4.s3/5.png) 
9. Chọn **Iam Role**


10. Chọn Next
![DeployCF](/WorkShopTwo/images/4.s3/6.png) 

11. Chọn Add database

12. Nhập ticketdata là tên database và nhấn **Create database**

![DeployCF](/WorkShopTwo/images/4.s3/7.png) 

13. Chọn **Target database** là ticketdata vừa tạo và nhấn next

![DeployCF](/WorkShopTwo/images/4.s3/9.png) 

14. Review và nhấn **Create crawler**.
![DeployCF](/WorkShopTwo/images/4.s3/10png) 

15. Thực hiện Crawler bằng cách nhấn **Run crawler** 
![DeployCF](/WorkShopTwo/images/4.s3/11.png) 

16. Tại AWS Glue chọn **Databases -> Tables**

![DeployCF](/WorkShopTwo/images/4.s3/13.png) 

### PART B: Xác thực dữ liệu

1. Chọn **ticketdata** database, **person** tables
Tại table này sẽ có một số cột không thể xác định tên. Chúng ta sẽ khắc phục nó.
2. Chọn **Edit Schema**

![DeployCF](/WorkShopTwo/images/4.s3/14.png) 

3. Chọn **colr0** và nhấn Edit
![DeployCF](/WorkShopTwo/images/4.s3/15.png) 

Nhập id làm column name và nhấn **Save**
![DeployCF](/WorkShopTwo/images/4.s3/16.png) 

Lặp lại các bước trên với từng các cột còn lại: full_name, last_name and first_name.
![DeployCF](/WorkShopTwo/images/4.s3/17.png) 

4. Nhấn **Save as new table version**.

### PART C: Data ETL
1. Chọn ETL jobs.
![DeployCF](/WorkShopTwo/images/4.s3/18.png) 

2. Chọn **Visual ETL**
![DeployCF](/WorkShopTwo/images/4.s3/19.png) 

3. Chọn **Amazon S3** từ Sources list để  thêm **Data source - S3 bucket**
![DeployCF](/WorkShopTwo/images/4.s3/20.png) 

4. Quan sát data source properties.
![DeployCF](/WorkShopTwo/images/4.s3/21.png) 

5. Chọn **ticketdata** database, chọn tables **sport_team**
![DeployCF](/WorkShopTwo/images/4.s3/22.png) 

6. Chọn **Change Schema** để thêm **Transform - Change Schema** node.

![DeployCF](/WorkShopTwo/images/4.s3/23.png) 

7. Quan sát properties của **Transform - Change Schema**  node. Đổi type của id thành double

![DeployCF](/WorkShopTwo/images/4.s3/24.png) 

8. Chọn S3 là target.
![DeployCF](/WorkShopTwo/images/4.s3/25.png) 
9. Chọn **Data target - S3 bucket** để xem thuộc tính. Đổi Format thành **Parquet**. Tại **Compression** Type chọn Uncompressed

10. Chọn **S3 Target Location**, nhấn **Browse S3** và chọn **tickets** item trong "dmslabs3bucket" bucket và nhấn **Choose**

![DeployCF](/WorkShopTwo/images/4.s3/26.png) 

11. Thêm dms_parquet/sport_team/ vào S3 url. 
![DeployCF](/WorkShopTwo/images/4.s3/27.png) 

12. Chọn **Job details** và Nhập tên là Glue-Lab-SportTeamParquet.
13. Chọn **IAM Role**
14. Tại **Job bookmark**, chọn **Disable**. Chúng ta sẽ thực hành bookmark tại bài lab tiếp theo/
![DeployCF](/WorkShopTwo/images/4.s3/28.png) 
15. Nhấn **Save** button để tạo job
16. Khi thấy thông báo **Successfully created job**, chọn Run để bắt đầu job.
17. Chọn Jobs phía panel bên trái để xem list jobs.
18. Chọn Monitoring để xem thống kê trạng thái và số lân run.
![DeployCF](/WorkShopTwo/images/4.s3/29.png) 

19. Chọn Job run để xác định ETL job đã chạy thành công. Mất khoảng tầm 1 phút.

![DeployCF](/WorkShopTwo/images/4.s3/30.png) 

20. Lặp lại các bước trên cho 4 tables **sport_location, sporting_event, sporting_event_ticket and person** tables

### PART D: Tạo Glue Crawler cho Parquet Files

1. Tại Glue navigation menu, chọn **Create crawler**.

![DeployCF](/WorkShopTwo/images/4.s3/32.png) 

2. Nhập glue-lab-parquet-crawler làm Crawler name và nhấn **Next**
![DeployCF](/WorkShopTwo/images/4.s3/33.png) 

3. Chọn **Not yet** và **Add a data source**

4. Chọn S3 làm data source
![DeployCF](/WorkShopTwo/images/4.s3/35.png) 

5. Chọn **Next**

6. Chọn **IAM role**
![DeployCF](/WorkShopTwo/images/4.s3/36.png) 

7. Chọn ticketdata làm  database
![DeployCF](/WorkShopTwo/images/4.s3/37.png) 

8. Review lại và nhấn **Create crawler**

![DeployCF](/WorkShopTwo/images/4.s3/38.png) 

9. Chọn **Run Crawler**
![DeployCF](/WorkShopTwo/images/4.s3/39.png) 

**Quan sát tables**
10. Chọn **Tables**
11. Chọn filter parquet và quan sát

![DeployCF](/WorkShopTwo/images/4.s3/40.png) 

Chúng ta đã hoàn thành bài lab **Data Validation and ETL**