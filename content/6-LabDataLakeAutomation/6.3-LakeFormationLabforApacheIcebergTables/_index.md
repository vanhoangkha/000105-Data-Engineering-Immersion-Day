---
title : "Lake Formation Lab for Apache Iceberg Tables"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

### Giới thiệu Apache Iceberg
Apache Iceberg là một định dạng bảng mở dành cho các bộ dữ liệu phân tích khổng lồ. Iceberg thêm các bảng vào các công cụ điện toán bao gồm Spark, Trino, PrestoDB, Flink và Hive bằng định dạng bảng hiệu suất cao hoạt động giống như bảng SQL. Iceberg đã được thiết kế và phát triển để trở thành một tiêu chuẩn cộng đồng mở với đặc điểm kỹ thuật nhằm đảm bảo khả năng tương thích giữa các ngôn ngữ và cách triển khai.

Tìm hiểu thêm về Apache Iceberg:
1. [Apache Iceberg documentation ](https://iceberg.apache.org/docs/latest/)
2. [How Iceberg works ](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-iceberg-how-it-works.html)
3. [Using the Iceberg framework in AWS Glue](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format-iceberg.html)

Trong bài lab này chúng ta sẽ: Tạo Apache Iceberg tables

### Bước 1 Tạo glue job và iceberg database và tables
1. Truy cập: [Athena Console](https://console.aws.amazon.com/athena/home) và chạy truy vấn sau:
```
CREATE DATABASE ticketdata_iceberg LOCATION 's3://<BUCKET_NAME>/data/ticketdata_iceberg/';
```
2. Tạo Glue job

![Clean](/WorkShopTwo/images/6.clean/40.png)

![Clean](/WorkShopTwo/images/6.clean/41.png)

3. Copy đoạn code sau và dán vào Glue script editor
```

import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkConf, SparkContext
from pyspark.sql import SparkSession
from awsglue.context import GlueContext
from awsglue.job import Job
from pyspark.sql.functions import col

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'RAWZONE_BUCKET','ICEBERG_DB_NAME'])

AWS_BUCKET = args['RAWZONE_BUCKET']
GLUE_DB_NAME = args['ICEBERG_DB_NAME'] # Update with your GlueDB
TABLE_NAME = "ticket_purchase_hist"
conf_list = [
    #General Spark configs
    ("hive.metastore.client.factory.class", "com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory"),
    ("spark.sql.parquet.writeLegacyFormat", "true"),
    ("spark.sql.parquet.writeLegacyFormat", "true"),
    ("hive.exec.dynamic.partition.mode", "nonstrict"),
    ("spark.sql.hive.caseSensitiveInferenceMode", "INFER_ONLY"),
    ("spark.sql.source.partitionOverviewMode", "dynamic"),
    #Configs needed for Iceberg
    ("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions"),
    ("spark.sql.catalog.iceberg_catalog", "org.apache.iceberg.spark.SparkCatalog"),
    ("spark.sql.catalog.iceberg_catalog.warehouse", f"s3://{AWS_BUCKET}/iceberg_catalog/"),
    ("spark.sql.catalog.iceberg_catalog.catalog-impl", "org.apache.iceberg.aws.glue.GlueCatalog")
]
 
spark_conf = SparkConf().setAll(conf_list)
spark = SparkSession.builder.config(conf=spark_conf).enableHiveSupport().getOrCreate()
glue_context = GlueContext(spark.sparkContext)
job = Job(glue_context)
job.init(args['JOB_NAME'], args)
 
df = spark.read.format("csv").option("header", "true").load(f"s3://{AWS_BUCKET}/tickets/dms_sample/ticket_purchase_hist/LOAD00000001.csv")
df = df.withColumn("purchase_price",col("purchase_price").cast('double'))
df.registerTempTable("input_data")
 
## CREATE TABLE
 
sql_stmnt = f"""
CREATE OR REPLACE TABLE iceberg_catalog.{GLUE_DB_NAME}.{TABLE_NAME}
USING iceberg
TBLPROPERTIES ('table_type'='ICEBERG', 'format-version'='2')
LOCATION 's3://{AWS_BUCKET}/data/{GLUE_DB_NAME}/{TABLE_NAME}'
AS SELECT * FROM input_data
"""
spark.sql(sql_stmnt).show()

job.commit()

```
4. Chọn Job Details và nhập các thông số sau:
- Name: glue-iceberg-lf-job
- IAM: *-GlueLabRole-*
- Type: Spark
- Glue version: Glue 3.0 - Supports spark 3.1, Scala 2, Python 3.
- Language: Python 3
- Disable the Job bookmark
- Tất cả option còn lại để mặc định

5. Nhập **Job parameters** như sau:
- Key: --ICEBERG_DB_NAME, Value: ticketdata_iceberg
- Key: --RAWZONE_BUCKET, Value: Enter the Bucket name from s3-target-endpoint.

- Key: --conf, Value: spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.glue_catalog=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.glue_catalog.warehouse=s3://<BUCKET_NAME>/data/ticketdata_iceberg --conf spark.sql.catalog.glue_catalog.catalog-impl=org.apache.iceberg.aws.glue.GlueCatalog --conf spark.sql.catalog.glue_catalog.io-impl=org.apache.iceberg.aws.s3.S3FileIO
- Key: --datalake-formats, Value: iceberg

6. Nhấn Save và click run. Đợi đến khi Run status là Succeeded
![Clean](/WorkShopTwo/images/6.clean/42.png)

7. Glue job đã tạo tables ticket_purchase_hist

### Bước 2 Truy vấn Iceberg table trong Athena
1. Xác định  Iceberg table được tạo thành công.
2. Ta thấy tables và data mới ticket_purchase_hist ticketdata_iceberg.
![Clean](/WorkShopTwo/images/6.clean/43.png)

3. Truy vấn ticket_purchase_hist từ ticketdata_iceberg database
![Clean](/WorkShopTwo/images/6.clean/44.png)
![Clean](/WorkShopTwo/images/6.clean/45.png)


### Phân quyền truy vấn
Phần này tương tự như lab: Bước 3 [Lake Formation Lab for Apache Hudi Tables](../6.3-LakeFormationLabforApacheIcebergTables/_index.md)