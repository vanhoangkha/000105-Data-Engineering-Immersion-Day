+++
title = "Lab: Data Lake Automation"
date = 2022
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

### Giới thiệu
Trong bài lab này, chúng ta sẽ cùng tìm hiểu về AWS Lake Formation một dịch vụ giúp bạn dễ dàng thiết lập data-lake an toàn trong vài ngày cũng như Athena để truy vấn dữ liệu bạn nhập vào hồ dữ liệu của mình.
![Clean](/WorkShopTwo/images/6.clean/1.png)


### Yêu cầu
Hoàn thành [Ingestion with DMS](../../3-IngestionwithDMS/_index.md) và [Transforming data with Glue ETL](../../4-TransformingdatawithGlue/_index.md) labs.

## Công việc hoàn thành trong bài lab này:

Trong bài này, chúng ta sẽ:
1. Tạo JDBC connection to RDS trong AWS Glue
2. Tạo AWS Lake Formation IAM Role
3. Thêm Administrator trong Lake Formation và bắt đầu tạo workflows sử dụng Blueprints
4. Xem các thành phần của Glue Workflow được tạo bởi Lake Formation
5. Kiểm tra kết quả workflow trong Athena
6. Cấp quyền kiểm soát quyền truy cập chi tiết cho người dùng Data Lake
7. Xác minh quyền sử dụng Athena
