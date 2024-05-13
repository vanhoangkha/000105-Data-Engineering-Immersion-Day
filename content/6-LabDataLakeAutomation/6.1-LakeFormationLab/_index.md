---
title : "Lake Formation Lab"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

### Tạo Glue JDBC connection for RDS
1. Truy cập [AWS Glue console](https://console.aws.amazon.com/glue/home).
2. Chọn **Connections** 

![Clean](/WorkShopTwo/images/6.clean/2.png)

3. Chọn **Create connection**.
4. Nhập tên là glue-rds-connection
5. Chọn **JDBC** connection type
6. Nhập miêu tả
7. Chọn JDBC URL

![Clean](/WorkShopTwo/images/6.clean/3.png)

8. Nhập username và password của RDS
9. Mở rộng **Network options - optinoal**
10. Chọn Subnet private
11. Chọn **security group** là **sgdefault**
![Clean](/WorkShopTwo/images/6.clean/4.png)
12. Chọn **Create Connection** để hoàn thành **glue-rds-connection** setup

![Clean](/WorkShopTwo/images/6.clean/5.png)

### Thêm Administrator và start workflows sử dụng Blueprints

Truy cập [AWS Lake Formation](https://console.aws.amazon.com/lakeformation/home): 
![Clean](/WorkShopTwo/images/6.clean/6.png)

1. Nếu là lần đầu đăng nhập vào Lake Formation. Đầu tiên chúng ta cần thêm administrators.
2. Mở **Welcome to Lake Formation pop-up**. Chọn **Add myself** checkbox và nhấn **Get started**
![Clean](/WorkShopTwo/images/6.clean/77.png)

3. Click chọn **Databases**. Chọn **ticketdata** và nhấn **Actions** -> **Grant**.
![Clean](/WorkShopTwo/images/6.clean/8.png)
4. Trao quyền **super** cho cả 2 **Database permissions** and **Grantable permissions**.
![Clean](/WorkShopTwo/images/6.clean/9.png)

5. Chọn Edit database **ticketdata**

![Clean](/WorkShopTwo/images/6.clean/10.png)

6. Uncheck **Use only IAM access control** và nhấn **Save**
![Clean](/WorkShopTwo/images/6.clean/1111.jpeg)

7. Chọn **Blueprints** và nhấn **Use blueprint**

![Clean](/WorkShopTwo/images/6.clean/11.png)

- **Blueprint Type**, Chọn **Database snapshot**
- **Import Source**: 
    - Database Connection Chọn glue-rds-connection
    - Source Data Path: sportstickets/dms_sample/mlb_data
![Clean](/WorkShopTwo/images/6.clean/12.png)

- **Import Target**
    - Target Database: ticketdata
    - Target storage location: xxx-dmslabS3bucket-xxx
    - Thêm /lakeformation vào buckets url, Ví dụ: s3://mod-08b80667356c4f8a-dmslabs3bucketnh54wqg771lk/lakeformation
    - Data Format: Parquet

![Clean](/WorkShopTwo/images/6.clean/1313.png)

- Các mục còn lại nhập như sau:

![Clean](/WorkShopTwo/images/6.clean/144.png)

8. Các options còn lại để mặc định. Nhấn **Create**
9. Khi blueprint được tạo thành công. Nhấn **Action → Start.**. 
![Clean](/WorkShopTwo/images/6.clean/14.png)
10. Bạn sẽ thấy trạng thái thay đổi như sau: **Running → Discovering → Importing → Completed**
![Clean](/WorkShopTwo/images/6.clean/15.png)

## Xem các thành phần bên dưới của một BLueprint
Lake Formation blueprin sẽ tạo Glue Workflow điều phối các công việc của Glue ETL (cả python shell và pyspark), Glue crawlers và triggers. Sẽ mất tầm 20-30 phút để thực thi lần đầu tiên. Chúng ta hãy cùng xem các thành phần mà nó tạo ra:
1. Tại **Lake Formation console**, Chọn Blueprints
2. Tại  Workflow section, chọn tên Workflow. CLick vào Run Id.
![Clean](/WorkShopTwo/images/6.clean/16.png)

3. Tại đây bạn có thể thấy Workflow Run details
![Clean](/WorkShopTwo/images/6.clean/17.png)

4. Để xem các jobs được tạo. Nhấn **AWS Glue** và Chọn **ETL Jobs.**
![Clean](/WorkShopTwo/images/6.clean/18.png)

5. Xem lịch sử và các thông số liên quan.
![Clean](/WorkShopTwo/images/6.clean/19.png)


### Xem kết quả sử dụng Athena

1. Chọn Database **ticketdata** tại Lake Formation console và nhấn view Tables.
2. Chọn tables lakeformation__sportstickets_dms_sample_mlb_data, **Action → View Data**
![Clean](/WorkShopTwo/images/6.clean/20.png)
3. Tại Athena console. chạy query:
![Clean](/WorkShopTwo/images/6.clean/21.png)

```
SELECT * FROM "ticketdata"."lakeformation__sportstickets_dms_sample_mlb_data" limit 10;
```

```
SELECT count(*) as recordcount FROM "ticketdata"."lakeformation__sportstickets_dms_sample_mlb_data";
```

### Cấp quyền truy cập ở mức column

1. Thêm IAM User
![Clean](/WorkShopTwo/images/6.clean/22.png)

2. Tạo user với tên là: **datalake_user**
![Clean](/WorkShopTwo/images/6.clean/23.png)
3. Gán cho User policy **AthenaFullAccess**
4. Tạo user

5. Chọn User **datalake_user** vừa tạo, Nhấn add inline policy:

![Clean](/WorkShopTwo/images/6.clean/24.png)

Sử dụng đoạn json sau đây.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
        "Effect": "Allow",
        "Action": [
        "s3:Put*",
        "s3:Get*",
        "s3:List*"
        ],
        "Resource": [ "arn:aws:s3:::<your_dmslabs3bucket_unique_name>/*" ]
        }
    ]
}  
```
6. Thêm tiếp policy sau và user
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lakeformation:StartQueryPlanning",
                "lakeformation:GetQueryState",
                "lakeformation:GetWorkUnits",
                "lakeformation:GetWorkUnitResults"
            ],
            "Resource": "*"
        }
    ]
}
```

7. Tại **Lake Formation console**, Permissions, chọn **Data lake permissions**
8. chọn Grant
9. Nhập các thông số sau vào popup:
- IAM user and roles: datalake_user.
- Chọn Named data catalog resources
- Database, choose ticketdata
- Table: lakeformation__sportstickets_dms_sample_mlb_data.
- Table permissions, Chọn Select.
- Data permissions, Chọn Column-based access
- Columns, Chọn Include Columns và chọn mlb_name, mlb_team_long, mlb_team and mlb_pos
- Click Grant

### Xác minh quyền truy cập dữ liệu theo column sử dụng Athena

1. Đăng nhập bằng user datalake_user

![Clean](/WorkShopTwo/images/6.clean/25.jpeg)

2. Truy cập athena, chọn **ticketdata** database và chạy query sau:
```
SELECT * FROM "ticketdata"."lakeformation__sportstickets_dms_sample_mlb_data" limit 10;
```

3. Kết quả chỉ hiển thị những column được gán quyền như bên trên: mlb_name, mlb_team_long, mlb_team and mlb_pos

![Clean](/WorkShopTwo/images/6.clean/27.png)
