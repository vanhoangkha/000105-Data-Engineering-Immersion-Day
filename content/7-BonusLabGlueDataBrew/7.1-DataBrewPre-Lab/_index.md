---
title : "DataBrew Pre-Lab"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 7.1 </b> "
---

### Setup
Trong bài lab này, chúng ta sẽ sử dụng AWS Glue DataBrew để khám phá dữ liệu trong S3, làm sạch và chuẩn bị dữ liêu.
Đầu tiên chúng ta phải tạo 1 IAM role để sử dụng trong DataBrew và cho kết quả trong S3 bucket từ DataBrew jobs.

1. Click vào đây để deploy CloudFormation Stack:
{{< button href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?templateURL=https://s3.us-east-1.amazonaws.com/aws-dataengineering-day.workshop.aws/DataBrew_PreLab_CFN.yaml&stackName=databrew-lab&param_SourceBucket=aws-dataengineering-day.workshop.aws&param_SourceKey=states_daily.csv.gz" class="btn btn-white" >}}Deploy To AWS{{< /button >}}

2. Sau khi deploy stack thành công. Nhấn **Output** để xem các thông tin

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/1.png)

Ta sẽ sử dụng các tham số  **DatasetS3Path, DataBrewLabRole and DataBrewOutputS3Bucket** trong bài lab này.
