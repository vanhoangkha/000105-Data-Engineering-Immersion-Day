---
title : "Athena and SageMaker"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

### Giới thiệu

[Amazon SageMaker](https://aws.amazon.com/sagemaker/) là một nền tảng machine learning toàn diện cho phép bạn xây dựng, đào tạo và triển khai các mô hình machine learning trong AWS. Đây là một dịch vụ có tính mô-đun cao cho phép bạn sử dụng từng thành phần này một cách độc lập với nhau.

Ở bài này, chúng ta sẽ thực hành:
- Cách sử dụng Jupyter notebooks trong SageMaker để tích hợp với data lake bằng Athena
- Tạo data frames để thao tác với dữ liệu

![Athena](/WorkShopTwo/images/5.fwd/96.png) 

### Tạo Amazon SageMaker Notebook Instance

1. Truy cập [Amazon Sagemaker](https://console.aws.amazon.com/sagemaker/home), Tại **Notebook instances** chọn **Create notebook instance**
![Athena](/WorkShopTwo/images/5.fwd/97.png) 

2. Nhập các thông tin sau:
- Name: datalake-Sagemaker
- Type: ml.t3.medium
- Elastic Inference: none.
- IAM role: Tạo IAM role Any S3 bucket, gán thêm **AmazonAthenaFullAccess**
![Athena](/WorkShopTwo/images/5.fwd/98.png) 

![Athena](/WorkShopTwo/images/5.fwd/99.png) 

![Athena](/WorkShopTwo/images/5.fwd/100.png) 

Chọn **Create notebook instance**
![Athena](/WorkShopTwo/images/5.fwd/101.png) 

3. Đợi Trạng thái của note book thành **InService**. Chọn **Open Jupyter**
![Athena](/WorkShopTwo/images/5.fwd/02021.png) 

4. Mở notebook interface
![Athena](/WorkShopTwo/images/5.fwd/102.png) 


### Kết nối SageMaker Jupyter notebook tới Athena

1. Tại Jupyter notebook tab, Chọn **New** và chọn **conda_python3**
![Athena](/WorkShopTwo/images/5.fwd/103.png) 

2. Cài đặt PyAthena

```
!pip install PyAthena[SQLAlchemy]
```
![Athena](/WorkShopTwo/images/5.fwd/104.png) 

### Làm việc với panda

1. Chạy Command sau đây bằng notebook cell để lấy region hiện tại
```
!aws configure get region
```

Chạy đoạn code sau trong note book:
```
from sqlalchemy import create_engine
import pandas as pd

s3_staging_dir = "s3://dmslab-student-dmslabs3bucket-xxx/athenaquery/"
connection_string = f"awsathena+rest://:@athena.us-east-1.amazonaws.com:443/ticketdata?s3_staging_dir={s3_staging_dir}"

engine = create_engine(connection_string)

df = pd.read_sql('SELECT * FROM "ticketdata"."nfl_stadium_data" order by stadium limit 10;', engine)
df
```
2. Chọn Run và quan sát dataframe,

![Athena](/WorkShopTwo/images/5.fwd/105.png) 

3. 
```
df = pd.read_sql('SELECT sport, \
       event_date_time, \
       home_team,away_team, \
       city, \
       count(*) as tickets, \
       sum(ticket_price) as total_tickets_amt, \
       avg(ticket_price) as avg_ticket_price, \
       max(ticket_price) as max_ticket_price, \
       min(ticket_price) as min_ticket_price  \
       FROM "ticketdata"."sporting_event_ticket_info" \
       group by 1,2,3,4,5 \
       order by 1,2,3,4,5  limit 1000;', engine)
df
```
![Athena](/WorkShopTwo/images/5.fwd/106.png) 

4. Vẽ biểu đồ
```
import matplotlib.pyplot as plt 
df.plot(x='event_date_time',y='avg_ticket_price')
```
![Athena](/WorkShopTwo/images/5.fwd/107.png) 
