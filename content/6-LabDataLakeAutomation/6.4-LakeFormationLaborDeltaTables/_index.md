---
title : "Lake Formation Lab for Delta Tables"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 6.4 </b> "
---

### Yêu cầu.

- Tải dữ liệu vào S3 bằng các hoàn thành Lab [MainLab](../../3-IngestionwithDMS/3.1-DMS-Migration-Lab/_index.md)

### Các bước thực hiện.
- Tạo Glue Job để load dữ liệu vào Delta tables
- Chạy Glue Crawler
- Xem kết quả workflow sử dụng Athena

### Tạo Glue Job để load dữ liệu vào delta table
1. Tạo glue job và past đoạn code sau đây vào.
![Clean](/WorkShopTwo/images/6.clean/46.png)
![Clean](/WorkShopTwo/images/6.clean/47.png)

```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from delta.tables import * #added to support delta tables

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# Script generated for node S3 bucket
S3bucket_node1 = glueContext.create_dynamic_frame.from_options(
    format_options={"quoteChar": '"', "withHeader": True, "separator": ","},
    connection_type="s3",
    format="csv",
    connection_options={
        "paths": ["s3://<BUCKET NAME>/tickets/dms_sample/mlb_data/"],
        "recurse": True,
    },
    transformation_ctx="S3bucket_node1",
)

# Script generated for node ApplyMapping
ApplyMapping_node2 = ApplyMapping.apply(
    frame=S3bucket_node1,
    mappings=[
        ("mlb_id", "string", "mlb_id", "int"),
        ("mlb_name", "string", "mlb_name", "string"),
        ("mlb_pos", "string", "mlb_pos", "string"),
        ("mlb_team", "string", "mlb_team", "string"),
        ("mlb_team_long", "string", "mlb_team_long", "string"),
        ("bats", "string", "bats", "string"),
        ("throws", "string", "throws", "string"),
        ("birth_year", "string", "birth_year", "int"),
        ("bp_id", "string", "bp_id", "int"),
        ("bref_id", "string", "bref_id", "int"),
        ("bref_name", "string", "bref_name", "string"),
        ("cbs_id", "string", "cbs_id", "int"),
        ("cbs_name", "string", "cbs_name", "string"),
        ("cbs_pos", "string", "cbs_pos", "string"),
        ("espn_id", "string", "espn_id", "int"),
        ("espn_name", "string", "espn_name", "string"),
        ("espn_pos", "string", "espn_pos", "string"),
        ("fg_id", "string", "fg_id", "int"),
        ("fg_name", "string", "fg_name", "string"),
        ("lahman_id", "string", "lahman_id", "int"),
        ("nfbc_id", "string", "nfbc_id", "int"),
        ("nfbc_name", "string", "nfbc_name", "string"),
        ("nfbc_pos", "string", "nfbc_pos", "string"),
        ("retro_id", "string", "retro_id", "int"),
        ("retro_name", "string", "retro_name", "string"),
        ("debut", "string", "debut", "string"),
        ("yahoo_id", "string", "yahoo_id", "int"),
        ("yahoo_name", "string", "yahoo_name", "string"),
        ("yahoo_pos", "string", "yahoo_pos", "string"),
        ("mlb_depth", "string", "mlb_depth", "string"),
    ],
    transformation_ctx="ApplyMapping_node2",
)

# Script generated for node S3 bucket
additional_options = {
    "path": "s3://<BUCKET NAME>/tickets/dms_sample_deltalake/mlb_data/",
    "write.parquet.compression-codec": "snappy",
}
S3bucket_node3_df = ApplyMapping_node2.toDF()
S3bucket_node3_df.write.format("delta").options(**additional_options).mode("append").save()

job.commit()
```

2. Nhập các thông số sau ở tab Job Details:
- Name: job-WorkshopConvertToDeltaTable
- Type: Spark
- Glue version: Glue 3.0 - Supports spark 3.1, Scala 2, Python 3.
- Language: Python 3
- Disable Job bookmark
- Để các option còn lại default
3. Nhập **Job parameters** như sau:
- Key: --conf, Value: spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension --conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog
- Key: --datalake-formats, Value: delta

4. Nhấn Save, Run và đợi đến khi trạng thái run là Succeeded

![Clean](/WorkShopTwo/images/6.clean/48.png)

### Glue Crawler to register delta table with Glue Catalog

1. Mở [ AWS Glue Console ](https://console.aws.amazon.com/glue/home)
2. Chọn **Crawlers**
3. Nhấn: **Create crawler**
4. Nhập tên là: Create crawler
5. Chọn Data Source,   Include delta lake table paths và Create Symlink tables
![Clean](/WorkShopTwo/images/6.clean/49.png)

6. Chọn IAM và Chọn Database đích
![Clean](/WorkShopTwo/images/6.clean/50.png)
![Clean](/WorkShopTwo/images/6.clean/51.png)
![Clean](/WorkShopTwo/images/6.clean/52.png)
![Clean](/WorkShopTwo/images/6.clean/53.png)

7. Tạo và Nhấn **Run crawler** để chạy job
![Clean](/WorkShopTwo/images/6.clean/54.png)
![Clean](/WorkShopTwo/images/6.clean/55.png)
![Clean](/WorkShopTwo/images/6.clean/56.png)

### Truy vấn Delta table sử dụng Athena
1. Truy cập Amazon Athena Console và chọn ticketdata_delta database
2. Click Preview table để xem kết quả như sau:
![Clean](/WorkShopTwo/images/6.clean/57.png)

### Phân quyền truy vấn
Phần này tương tự như lab: Bước 3 [Lake Formation Lab for Apache Hudi Tables](../6.3-LakeFormationLabforApacheIcebergTables/_index.md)