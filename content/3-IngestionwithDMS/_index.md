---
title : "Nhập data với DMS"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

### Giới thiệu

AWS Database Migration Service (AWS DMS) giúp bạn di chuyển cơ sở dữ liệu sang AWS một cách nhanh chóng và an toàn. Cơ sở dữ liệu nguồn vẫn hoạt động đầy đủ trong quá trình di chuyển, giảm thiểu thời gian ngừng hoạt động đối với các ứng dụng dựa trên cơ sở dữ liệu. AWS Database Migration Service có thể di chuyển dữ liệu của bạn đến và đi từ các cơ sở dữ liệu thương mại và nguồn mở được sử dụng rộng rãi nhất.

AWS Database Migration Service hỗ trợ di chuyển đồng nhất như Oracle sang Oracle cũng như di chuyển không đồng nhất giữa các nền tảng cơ sở dữ liệu khác nhau, chẳng hạn như Oracle hoặc Microsoft SQL Server sang Amazon Aurora. Với AWS Database Migration Service, bạn cũng có thể liên tục sao chép dữ liệu với độ trễ thấp từ bất kỳ nguồn được hỗ trợ nào sang bất kỳ mục tiêu được hỗ trợ nào. Ví dụ: bạn có thể sao chép từ nhiều nguồn sang Amazon Simple Storage Service (Amazon S3) để xây dựng giải pháp hồ dữ liệu có tính sẵn sàng cao và có khả năng mở rộng. Bạn cũng có thể hợp nhất cơ sở dữ liệu vào kho dữ liệu quy mô petabyte bằng cách truyền dữ liệu tới Amazon Redshift.

**Các điều hướng của task**

DMS lab có 3 lựa chọn:
1. Nếu bạn muốn có trải nghiệm thực hành chuyên sâu về DMS (Dịch vụ di chuyển dữ liệu), trước tiên hãy chạy phòng thí nghiệm của người hướng dẫn để mô phỏng môi trường cơ sở dữ liệu quan hệ tại chỗ, sau đó là phòng thí nghiệm dành cho sinh viên để tạo cơ sở hạ tầng di chuyển dữ liệu cần thiết trong AWS . Phòng thí nghiệm chính sẽ giúp bạn thực hiện di chuyển dữ liệu thực tế từ cơ sở dữ liệu quan hệ sang kho dữ liệu trong AWS.
2. Nếu bạn muốn bắt đầu với Glue ETL và bỏ qua phần thực hành DMS hoàn toàn, vui lòng chạy phòng thí nghiệm tự động hoàn thành DMS. Tự động hoàn thành có thể mất từ 15 đến 20 phút để cung cấp tất cả dữ liệu trong lớp thô của hồ dữ liệu S3 tập trung và bạn sẽ sẵn sàng tìm hiểu sâu về chuyển đổi dữ liệu bằng Glue ETL.
3. Nếu bạn không muốn lab dịch vụ DMS, có thể sao chép trực tiếp dữ liệu thô sang S3 bằng Tùy chọn 3: Bỏ qua DMS Lab. Hạn chế của tùy chọn này là bạn không thể sử dụng DMS để tạo dữ liệu gia tăng nhằm kiểm tra tính năng bookmark của Glue.

