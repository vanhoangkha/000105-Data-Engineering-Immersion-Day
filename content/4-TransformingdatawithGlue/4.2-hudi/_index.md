---
title : "Incremental Data Processing with Hudi"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

### Giới thiệu Apache HUDI
Apache Hudi là một framework trừu tượng hóa lưu trữ giúp các tổ chức phân tán xây dựng và quản lý các data lakes có quy mô petabyte. Bằng cách sử dụng các phương pháp nguyên thủy như upsert và incremental pulls, Hudi mang đến khả năng xử lý kiểu luồng cho dữ liệu lớn batch-like big data. Hudi cho phép Atomicity, Consistency, Isolation & Durability (ACID) trên data lake.

Dưới đây là một số tài liệu tham khảo:
1. [Apache Hudi concepts](https://hudi.apache.org/docs/concepts/)
2. [How Hudi Works](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi-how-it-works.html)

Trong bài lab này, chúng ta sẽ học được:
1. How to create HUDI tables.
2. Process incremental updates.
3. Perform upsert operations.
4. Run incremental queries in a Glue job.

### Step
- Step 0 - Các yêu cầu
- Step 1 - Tạo glue job và HUDI tables
- Step 2 - Truy vấn HUDI tables bằng Athena
- Step 3 - HUDI configurations
- Step 4 - Upsert CDC data.
- Step 5 - Incremental Queries sử dụng Spark SQL.

### Step 0 - Prerequisites

1. Hoàn thành  [Main Lab](../../3-IngestionwithDMS/3.1-DMS-Migration-Lab/_index.md) or [Autocomplete DMS Lab](../../3-IngestionwithDMS/3.2-Private-instance/_index.md)

2. Source RDS database và trạng thái cdctask là ‘Replication ongoing’
![DeployCF](/WorkShopTwo/images/4.s3/41.png) 

3. Tên của S3 bucket trong s3-target-endpoint DMS
![DeployCF](/WorkShopTwo/images/4.s3/42.png) 

### Step 1 - Tạo Glue job và HUDI tables
1. Truy cập **AWS Glue Console** và chọn Jobs

![DeployCF](/WorkShopTwo/images/4.s3/43.png) 

2. Chọn **Spark script editor**, nhấn **Create a new script with boilerplate code** option và click **Create**
![DeployCF](/WorkShopTwo/images/4.s3/44.png) 

3. Copy đoạn code sau và past vào Glue script editor

```
from pyspark.sql.types import StringType
import sys
import os
import json

from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
from pyspark.sql.functions import concat, col, lit, to_timestamp, dense_rank, desc
from pyspark.sql.window import Window

from awsglue.utils import getResolvedOptions
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame

import boto3
from botocore.exceptions import ClientError

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'RAWZONE_BUCKET', 'CURATED_BUCKET'])

spark = SparkSession.builder.config('spark.serializer','org.apache.spark.serializer.KryoSerializer').config('spark.sql.hive.convertMetastoreParquet', 'false').getOrCreate()
glueContext = GlueContext(spark.sparkContext)
job = Job(glueContext)
job.init(args['JOB_NAME'], args)
logger = glueContext.get_logger()

logger.info('Initialization.')
glueClient = boto3.client('glue')

logger.info('Fetching configuration.')
region = os.environ['AWS_DEFAULT_REGION']

rawBucketName = args['RAWZONE_BUCKET']
curatedBucketName = args['CURATED_BUCKET']

if rawBucketName == None or curatedBucketName == None:
    raise Exception("Please input the bucket names in job parameters. Refer: https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-crawler-pyspark-extensions-get-resolved-options.html")

rawS3TablePath = 's3://' + rawBucketName + '/tickets/dms_sample/ticket_purchase_hist/'
cdcRawS3TablePath = 's3://' + rawBucketName + '/cdc/dms_sample/ticket_purchase_hist/'
curatedS3TablePathPrefix = 's3://' + curatedBucketName + '/hudi/'

sourceDBName = 'dms_sample'
sourceTableName = 'ticket_purchase_hist'

targetDBName = 'hudi_sample'
targetTableName = 'ticket_purchase_hist'

hudiStorageType = 'CoW'

dropColumnList = ['db','table_name','Op']
      
logger.info('Processing starts.')

keys = {
    "dms_sample.ticket_purchase_hist": {"primaryKey": "sporting_event_ticket_id"}
    }

spark.sql('CREATE DATABASE IF NOT EXISTS ' + targetDBName)

isTableExists = False
isPrimaryKey = False
isPartitionKey = False
primaryKey = ''
partitionKey = ''
try:
    glueClient.get_table(DatabaseName=targetDBName,Name=targetTableName)
    isTableExists = True
    logger.info(targetDBName + '.' + targetTableName + ' exists.')
except ClientError as e:
    if e.response['Error']['Code'] == 'EntityNotFoundException':
        isTableExists = False
        logger.info(targetDBName + '.' + targetTableName + ' does not exist. Table will be created.')

# lookup primary key and partition keys from keys json declared above
try:
    keyName = sourceDBName +"." + sourceTableName
    logger.info(keyName)
    table_config = ''
    
    for key in keys:
        if key == keyName:
            table_config = keys[key]
    
    try:
        primaryKey = table_config['primaryKey']
        isPrimaryKey = True
        logger.info('Primary key:' + primaryKey)
    except KeyError as e:
        isPrimaryKey = False
        logger.info('Primary key not found. An append only glueparquet table will be created.')
    try:
        partitionKey = table_config['partitionKey']
        isPartitionKey = True
        logger.info('Partition key:' + partitionKey)
    except KeyError as e:
        isPartitionKey = False
        logger.info('Partition key not found. Partitions will not be created.')
except ClientError as e:    
    if e.response['Error']['Code'] == 'ParameterNotFound':
        isPrimaryKey = False
        isPartitionKey = False
        logger.info('Config for ' + sourceDBName + '.' + sourceTableName + ' not found in parameter store. Non partitioned append only table will be created.')

# Reads the raw zone table and writes to HUDI table
try:
    inputDyf = glueContext.create_dynamic_frame_from_options(connection_type = 's3', connection_options = {'paths': [rawS3TablePath], 'groupFiles': 'none', 'recurse':True}, format = 'csv', format_options={'withHeader':True}, transformation_ctx = targetTableName)
    
    # Ensure timestamp is in HUDI timestamp format
    inputDf = inputDyf.toDF().withColumn('transaction_date_time', to_timestamp(col('transaction_date_time'))).withColumn(primaryKey, col(primaryKey).cast(StringType()))

    logger.info('Total record count in raw table = ' + str(inputDyf.count()))

    targetPath = curatedS3TablePathPrefix + '/' + targetDBName + '/' + targetTableName

    morConfig = {
        'hoodie.datasource.write.table.type': 'MERGE_ON_READ', 
        'hoodie.compact.inline': 'false', 
        'hoodie.compact.inline.max.delta.commits': 20, 
        'hoodie.parquet.small.file.limit': 0
    }

    commonConfig = {
        'className' : 'org.apache.hudi', 
        'hoodie.datasource.hive_sync.use_jdbc':'false', 
        'hoodie.datasource.write.precombine.field': 'transaction_date_time', 
        'hoodie.datasource.write.recordkey.field': primaryKey, 
        'hoodie.table.name': targetTableName, 
        'hoodie.datasource.hive_sync.database': targetDBName, 
        'hoodie.datasource.hive_sync.table': targetTableName, 
        'hoodie.datasource.hive_sync.enable': 'true'
    }

    partitionDataConfig = {
        'hoodie.datasource.write.partitionpath.field': partitionKey, 
        'hoodie.datasource.hive_sync.partition_extractor_class': 'org.apache.hudi.hive.MultiPartKeysValueExtractor', 
        'hoodie.datasource.hive_sync.partition_fields': partitionKey
    }
                    
    unpartitionDataConfig = {
        'hoodie.datasource.hive_sync.partition_extractor_class': 'org.apache.hudi.hive.NonPartitionedExtractor', 
        'hoodie.datasource.write.keygenerator.class': 'org.apache.hudi.keygen.NonpartitionedKeyGenerator'
    }
    
    incrementalConfig = {
        'hoodie.upsert.shuffle.parallelism': 20, 
        'hoodie.datasource.write.operation': 'upsert', 
        'hoodie.cleaner.policy': 'KEEP_LATEST_COMMITS', 
        'hoodie.cleaner.commits.retained': 10
    }
    
    initLoadConfig = {
        'hoodie.bulkinsert.shuffle.parallelism': 100, 
        'hoodie.datasource.write.operation': 'bulk_insert'
    }
    
    deleteDataConfig = {
        'hoodie.datasource.write.payload.class': 'org.apache.hudi.common.model.EmptyHoodieRecordPayload'
    }

    if(hudiStorageType == 'MoR'):
        commonConfig = {**commonConfig, **morConfig}
        logger.info('MoR config appended to commonConfig.')
    
    combinedConf = {}

    # HUDI require us to provide a primaryKey. If no primaryKey defined, we will fallback to 'glueparquet' format
    if(isPrimaryKey):
        logger.info('Going the Hudi way.')
        if(isTableExists):
            logger.info('Incremental load.')
            inputDyf = glueContext.create_dynamic_frame_from_options(connection_type = 's3', connection_options = {'paths': [cdcRawS3TablePath], 'groupFiles': 'none', 'recurse':True}, format = 'csv', format_options={'withHeader':True}, transformation_ctx = targetTableName)
            
            # Ensure timestamp is in HUDI timestamp format
            inputDf = inputDyf.toDF().withColumn('transaction_date_time', to_timestamp(col('transaction_date_time'))).withColumn(primaryKey, col(primaryKey).cast(StringType()))

            # ensure only latest updates are kept for each record using descending timestamp order
            w = Window.partitionBy(primaryKey).orderBy(desc('transaction_date_time'))
            inputDf = inputDf.withColumn('Rank',dense_rank().over(w))
            inputDf = inputDf.filter(inputDf.Rank == 1).drop(inputDf.Rank)

            outputDf = inputDf.filter("Op != 'D'").drop(*dropColumnList)

            if outputDf.count() > 0:
                logger.info('Upserting data. Updated row count = ' + str(outputDf.count()))
                if (isPartitionKey):
                    logger.info('Writing to partitioned Hudi table.')
                    outputDf = outputDf.withColumn(partitionKey,concat(lit(partitionKey+'='),col(partitionKey)))
                    combinedConf = {**commonConfig, **partitionDataConfig, **incrementalConfig}
                    outputDf.write.format('hudi').options(**combinedConf).mode('Append').save(targetPath)
                else:
                    logger.info('Writing to unpartitioned Hudi table.')
                    combinedConf = {**commonConfig, **unpartitionDataConfig, **incrementalConfig}
                    outputDf.write.format('hudi').options(**combinedConf).mode('Append').save(targetPath)
            
            outputDf_deleted = inputDf.filter("Op = 'D'").drop(*dropColumnList)

            if outputDf_deleted.count() > 0:
                logger.info('Some data got deleted.')
                if (isPartitionKey):
                    logger.info('Deleting from partitioned Hudi table.')
                    outputDf_deleted = outputDf_deleted.withColumn(partitionKey,concat(lit(partitionKey+'='),col(partitionKey)))
                    combinedConf = {**commonConfig, **partitionDataConfig, **incrementalConfig, **deleteDataConfig}
                    outputDf_deleted.write.format('hudi').options(**combinedConf).mode('Append').save(targetPath)
                else:
                    logger.info('Deleting from unpartitioned Hudi table.')
                    combinedConf = {**commonConfig, **unpartitionDataConfig, **incrementalConfig, **deleteDataConfig}
                    outputDf_deleted.write.format('hudi').options(**combinedConf).mode('Append').save(targetPath)
        else:
            outputDf = inputDf.drop(*dropColumnList)
            if outputDf.count() > 0:
                logger.info('Inital load.')
                if (isPartitionKey):
                    logger.info('Writing to partitioned Hudi table.')
                    outputDf = outputDf.withColumn(partitionKey,concat(lit(partitionKey+'='),col(partitionKey)))
                    combinedConf = {**commonConfig, **partitionDataConfig, **initLoadConfig}
                    outputDf.write.format('hudi').options(**combinedConf).mode('Overwrite').save(targetPath)
                else:
                    logger.info('Writing to unpartitioned Hudi table.')
                    combinedConf = {**commonConfig, **unpartitionDataConfig, **initLoadConfig}
                    outputDf.write.format('hudi').options(**combinedConf).mode('Overwrite').save(targetPath)
    else:
        if (isPartitionKey):
            logger.info('Writing to partitioned glueparquet table.')
            sink = glueContext.getSink(connection_type = 's3', path= targetPath, enableUpdateCatalog = True, updateBehavior = 'UPDATE_IN_DATABASE', partitionKeys=[partitionKey])
        else:
            logger.info('Writing to unpartitioned glueparquet table.')
            sink = glueContext.getSink(connection_type = 's3', path= targetPath, enableUpdateCatalog = True, updateBehavior = 'UPDATE_IN_DATABASE')
        sink.setFormat('glueparquet')
        sink.setCatalogInfo(catalogDatabase = targetDBName, catalogTableName = targetTableName)
        outputDyf = DynamicFrame.fromDF(inputDf.drop(*dropColumnList), glueContext, 'outputDyf')
        sink.writeFrame(outputDyf)
except BaseException as e:
    logger.info('An error occurred while processing table ' + targetDBName + '.' + targetTableName + '. Please check the error logs...')
    print(e)

job.commit()

```
4. Chọn vào Job Detail:
- Nhập Name là: glue-hudi-job
- Chọn Iam Role tương tự như thế này: *-GlueLabRole-*

5. Chọn Type là **Spark**

6. Glue version: **Glue 3.0 - Supports spark 3.1, Scala 2, Python 3.**
7. Chọn **Language** là **Python 3**
8. Disable **Job bookmark**
9. Tất cả còn lại để mặc định.
10. Mở rộng Advanced Properties
11. Nhập **Job parameters**
12. Nhấn Save và click **Run**
13. Đợi tới khi Succeeded
![DeployCF](/WorkShopTwo/images/4.s3/45.png) 
14. Glue job sẽ tạo HUDI tables ticket_purchase_hist

### Step 2 – Truy vấn HUDI table trong Athena

1. Đảm báo rằng HUDI table được tạo thành công. Click vào Tables 

2. Bạn sẽ thấy new tables là ticket_purchase_hist và database là hudi_sample
![DeployCF](/WorkShopTwo/images/4.s3/46.png) 

3. Truy cập Amazon Athena Console và chọn hudi_sample Database
4. Setup nơi lưu kết quả.

![DeployCF](/WorkShopTwo/images/4.s3/47.png) 

![DeployCF](/WorkShopTwo/images/4.s3/48.png) 

5. Chọn ellipsis và **Preview table**, ticket_purchase_hist
6. Count ticket_purchase_hist bằng cách chạy query sau
```
select count(1) from hudi_sample.ticket_purchase_hist
```

![DeployCF](/WorkShopTwo/images/4.s3/49.png) 

7. Kết quả HUDI tables
![DeployCF](/WorkShopTwo/images/4.s3/50.png) 

### Step 3 – HUDI HUDI configurations

Một số  configurations:
| Attribute   |      Description      |
|----------|:-------------:|
| hoodie.datasource.write.table.type |  Chọn COPY_ON_WRITE hoặc MERGE_ON_READ table type. | 
| hoodie.datasource.write.recordkey.field | Tương tự primary key ở cơ sở dữ liệu quan hệ      |  
| hoodie.datasource.hive_sync.partition_fields | Trường trong bảng dùng để xác định các hive partition columns |  
| hoodie.datasource.write.operation | Chọn upsert, insert hoặc bulkinsert cho các hành động write |  
| hoodie.datasource.read.end.instanttime | Thời gian tức thì nạp data |  

Xem thêm các configurations khác [Apache Hudi documentation](https://hudi.apache.org/docs/configurations)

### Step 4 – Upsert Incremental Changes
Ở các bước trước, chúng ta đã tạo một bảng HUDI có tên ticket_purchase_hist, bao gồm toàn bộ bảng nguồn. Bây giờ, hãy tạo một số dữ liệu CDC để có thể cập nhật các thay đổi gia tăng vào bảng HUDI.

1. Làm theo các bước ở đây để tạo CDC data -  [Generate CDC Data](https://catalog.us-east-1.prod.workshops.aws/workshops/976050cc-0606-4b23-b49f-ca7b8ac4b153/en-US/400/401/410-pre-lab-1#generate-and-replicate-the-cdc-data-(optional))

2. Sau khi CDC data được tạo, đảm bảo rằng **cdctask** trong DMS cập nhập bản ghi như bên dưới:
![DeployCF](/WorkShopTwo/images/4.s3/51.png) 

3. Chạy Glue job glue-hudi-job để upsert the CDC changestới HUDI table ticket_purchase_hist. Sau khi Glue job chạy thành công. Bạn có thể truy cập [ Amazon Athena Console ](https://console.aws.amazon.com/athena) để xác minh việc tăng số lượng bản ghi.
![DeployCF](/WorkShopTwo/images/4.s3/52.png) 

### Step 5 – Chạy Incremental Queries sử dụng Spark SQL

1. Truy cập AWS Glue Console, click **Job** 

![DeployCF](/WorkShopTwo/images/4.s3/53.png) 

2. Chọn gluehudijob. Nhấn **Clone job** ở **Action**
![DeployCF](/WorkShopTwo/images/4.s3/54.png) 

3. Copy đoạn code sau.
```
import os
import sys

import boto3
from awsglue.context import GlueContext
from awsglue.dynamicframe import DynamicFrame
from awsglue.job import Job
from awsglue.utils import getResolvedOptions
from botocore.exceptions import ClientError
from pyspark.sql.session import SparkSession
from pyspark.sql.types import *

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'CURATED_BUCKET'])

spark = SparkSession.builder.config(
    'spark.serializer',
    'org.apache.spark.serializer.KryoSerializer').config('spark.sql.hive.convertMetastoreParquet','false').getOrCreate()

glueContext = GlueContext(spark.sparkContext)
job = Job(glueContext)
job.init(args['JOB_NAME'], args)
logger = glueContext.get_logger()

logger.info('Initialization.')
glueClient = boto3.client('glue')

logger.info('Fetching configuration.')
region = os.environ['AWS_DEFAULT_REGION']

curatedBucketName = args['CURATED_BUCKET']

if curatedBucketName == None:
    raise Exception(
        "Please input the bucket names in job parameters. Refer: https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-crawler-pyspark-extensions-get-resolved-options.html"
    )

keys = {
    "dms_sample.ticket_purchase_hist": {"primaryKey": "sporting_event_ticket_id"}
    }

curatedS3TablePathPrefix = 's3://' + curatedBucketName + '/hudi/'

hudiDBName = 'hudi_sample'
hudiTableName = 'ticket_purchase_hist'
primaryKey = 'sporting_event_ticket_id'

unpartitionDataConfig = {
    'hoodie.datasource.hive_sync.partition_extractor_class': 'org.apache.hudi.hive.NonPartitionedExtractor',
    'hoodie.datasource.write.keygenerator.class': 'org.apache.hudi.keygen.NonpartitionedKeyGenerator'
}

incrementalConfig = {
    'hoodie.upsert.shuffle.parallelism': 68,
    'hoodie.datasource.write.operation': 'upsert',
    'hoodie.cleaner.policy': 'KEEP_LATEST_COMMITS',
    'hoodie.cleaner.commits.retained': 10
}

all_commits = list(
    map(
        lambda row: row[0],
        spark.sql(
            "select distinct(_hoodie_commit_time) as commitTime from " + hudiDBName + ".ticket_purchase_hist order by commitTime"
        ).limit(50).collect()))

logger.info('Total number of commits are: ' + str(len(all_commits)))
beginTime = all_commits[len(all_commits) - 2]  # commit time we are interested in

# incrementally query data
incremental_read_options = {
    'hoodie.datasource.query.type': 'incremental',
    'hoodie.datasource.read.begin.instanttime': beginTime,
}

incrementalDF = spark.read.format("hudi").options(**incremental_read_options). \
  load(curatedS3TablePathPrefix + hudiDBName + '/' + hudiTableName)

commonConfig = {
    'className': 'org.apache.hudi',
    'hoodie.datasource.hive_sync.use_jdbc': 'false',
    'hoodie.datasource.write.precombine.field': 'transaction_date_time',
    'hoodie.datasource.write.recordkey.field': primaryKey,
    'hoodie.table.name': hudiTableName + '_incremental',
    'hoodie.datasource.hive_sync.database': hudiDBName,
    'hoodie.datasource.hive_sync.table': hudiTableName + '_incremental',
    'hoodie.datasource.hive_sync.enable': 'true'
}

combinedConf = { **commonConfig, **unpartitionDataConfig, **incrementalConfig }

incrementalDF.write.format('hudi').options(**combinedConf).mode('Overwrite').save(curatedS3TablePathPrefix + '/' + hudiDBName + '/' + hudiTableName + '_incremental')
job.commit()

```

4. Chọn **Job Details**
- Name: incremental-hudi-job
- IAM role: *-GlueLabRole-*
5. Click **Save** and **Run**
6. Đợi status chuyển Succeeded
7. Truy cập Amazon Athena console và truy vấn table ticket_purchase_hist_incremental

![DeployCF](/WorkShopTwo/images/4.s3/55.png) 

### Tổng kết 
Chúng ta đã thực hiện được:
1. Glue job để tạo HUDI tables tương ứng với source tables ticket_purchase_hist cho CDC changes
2. Glue job khác để nhận incremental changes và lưu kết quả vào tables ticket_purchase_hist_incremental
3. Truy vấn HUDI tables bằng Amazon Athena