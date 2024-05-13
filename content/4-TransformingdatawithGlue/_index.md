---
title : "Lab: Transforming data with Glue"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
### Giới thiệu

Trong bài lab này, chúng ta sẽ tìm hiểu về AWS Glue, một dịch vụ tích hợp dữ liệu phi máy chủ giúp khám phá, chuẩn bị, di chuyển và tích hợp dữ liệu từ nhiều nguồn để phân tích, học máy (ML) và phát triển ứng dụng dễ dàng hơn. Bạn có thể sử dụng trình thu thập thông tin để điền vào Danh mục dữ liệu AWS Glue các bảng. Đây là phương pháp chính được hầu hết người dùng AWS Glue sử dụng. Trình thu thập thông tin có thể thu thập dữ liệu nhiều kho dữ liệu trong một lần chạy. Sau khi hoàn tất, trình thu thập thông tin sẽ tạo hoặc cập nhật một hoặc nhiều bảng trong Danh mục dữ liệu của bạn. Các công việc trích xuất, chuyển đổi và tải (ETL) mà bạn xác định trong AWS Glue sử dụng các bảng Danh mục dữ liệu này làm nguồn và mục tiêu. Công việc ETL đọc từ và ghi vào kho dữ liệu được chỉ định trong bảng Danh mục dữ liệu nguồn và đích.

### Yêu cầu:
> Note: Bạn cần hoàn thành [DMS Lab](../3-IngestionwithDMS/_index.md) để thực hiện bài lab này

### Tổng kết
Trong bài lab này, bạn sẽ thực hiện các task sau. Bạn có thể chọn chỉ hoàn thành Data Validation and ETL để chuyển sang bài lab tiếp theo nơi có thể truy vấn các bảng bằng Amazon Athena và Visualize bằng Amazon Quciksight.
- Data Validation and ETL
- Incremental Data Processing with Hudi
- Glue Job Bookmark (Optional)
- Glue Workflows (Optional)