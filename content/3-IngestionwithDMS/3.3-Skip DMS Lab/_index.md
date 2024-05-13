---
title : "Option 2: AutoComplete DMS Lab"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

### Giới thiệu

Lab trong Data Engineering workshop được thiết kế theo tuần tự. Bài lab này tự động deploy AWS Database Migration Service (AWS DMS) để chúng ta có thể nhanh chóng đến Glue Lab.
Nếu muốn hands-on với AWS DMS service. Hãy chọn [Option 1: DMS Main Lab](../3.1-DMS-Migration-Lab/_index.md)

### AutoComplete DMS

1. Chọn "Deploy to AWS" để deploy stack
{{< button href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=auto-dmslab&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/SkipDMSlab_student_CFN.yaml" class="btn btn-white" >}}Deploy To AWS{{< /button >}}

Stack sẽ hoàn thành các task sau:
- Khởi tạo môi trường cho workshop
- Tạo DMS subnet group trong VPC
- Tạo DMS replication instance
- Tạo source endpoint cho RDS source database
- Tạo target endpoint cho full data load
- Tạo target endpoint cho CDC
- Tạo task thực hiện full migration
- Tạo task hỗ trợ replication thay đổi.
2. Chọn Parameters:
- DMSCWRoleCreated: Nếu có role dms-cloudwatch-logs-role thì chọn **true**, nếu không thì chọn **false**
- DMSVPCRoleCreated: Nếu có dms-vpc-role thì chọn true, còn không thì chọn **false**
- ServerName: RDS Database Server Name
![DeployCF](/WorkShopTwo/images/3.connect/79.png) 

3. Phía dưới Capabilities, tích checkbox acknowledge the policy và chọn Create stack để tạo.

4. Stack cần 5-6 phút để hoàn thành. Đợi đến khi "CREATE_COMPLETE"

5. Tại thời điểm này, dữ liệu nguồn đã được tải đầy đủ từ cơ sở dữ liệu RDS sang S3 bucket thông qua DMS. Truy cập [ AWS DMS console ](https://console.aws.amazon.com/dms/v2/home#tasks), bạn sẽ thấy hai Database migration tasks đã hoàn thành 100%. Nếu không, vui lòng đợi cho đến khi hoàn thành rồi chuyển sang Glue lab
![DeployCF](/WorkShopTwo/images/3.connect/80.png) 