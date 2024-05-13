---
title : "Data preparation with Glue DataBrew"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 7.2 </b> "
---

### Các công việc hoàn thành trong bài lab này
- Tạo Glue DataBrew để khám phá dữ liệu
- Kết nối với S3 sample dataset
- Khám phá dataset trong Glue DataBrew
- Tạo rich data profile trong dataset
- Dọn dẹp và chuẩn hóa dữ liệu


### Tạo project
1. Truy cập AWS Glue DataBrew và nhấn **Create project**
2. Tạo project với các thông tin:
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/2.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/3.png)

3. Tại phần **Connect to a new dataset**, chọn **Amazon S3** bên dưới "Data lake/data store"
Nhập **DatasetS3Path** từ output cloudformation
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/5.png)

4. Trong phần **Sampling** dể mặc định.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/6.png)

5. Trong phần **Permissions** chọn role name từ **OutPut** cloudformation có key là: **DataBrewLabRole**
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/7.png)

6. Nhấn **Create project**
Glue DataBrew sẽ tạo project trong vòng vài phút.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/8.png)

### Khám phá dataset

1. Ghi project đã được tạo, chúng ta sẽ thấy **Grid** view như sau. Đây là view mặc định, dữ liệu được hiển thị dưới dạng bảng
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/9.png)

Grid view hiển thị các thông tin sau:
  - Các cột trong dataset
  - Kiểu dữ liệu của các cột
  - Phạm vi của các giá trị tìm thấy
  - Phân phối thống kê các cột dữ liệu số

2. Chọn Schema Tab
Schema view hiển thị lược đồ đã được suy ra từ tập dữ liệu. Trong Schema view, bạn có thể xem số liệu thống kê về các giá trị dữ liệu trong mỗi cột.

Trong schema view, chúng ta có thể:
- Chọn checkbox  để xem tóm tắt thống kê cho các giá trị cột
- Hiển thị/Ẩn cột
- Đổi tên cột
- Thay đổi kiểu dữ liệu của cột
- Sắp xếp lại thứ tự cột bằng cách kéo thả các cột

3. Click vào **Profile** tab

Trong profile view, chúng ta có thể chạy data profile job để kiểm tra và thu thập các tóm tắt thống kê về dữ liệu. Data profile là sự đánh giá về mặt cấu trúc, nội dung, mối quan hệ và nguồn gốc.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/10.png)

Chọn **Run data profile**
Trong **Job details** và **Job run sample** để các value bằng mặc định. Trong **Job output settings** chọn value của key **DataBrewOutputS3Bucket** và thêm vào cuối /data-profile/ như sau:
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/11.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/12.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/13.png)

Cuối cùng nhấn **Create and run job**

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/142.png)

4. Chọn Jobs và click vào **Profile jobs**  tab để xem danh sách profile job.

Khi profile job successfully completed, chọn **View data profile**

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/14.png)

Sau đó, sẽ hiển thị ra **Data profile overview**

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/15.png)

Bạn cũng có thể truy cập Profile tab trong project
Data profile hiển thị bản tóm tắt về các hàng và cột trong tập dữ liệu, số lượng cột và hàng hợp lệ cũng như mối tương quan giữa các cột.

5. Chọn **Column statistics** tab để xem bảng phân tích từng cột của các giá trị dữ liệu.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/16.png)

### Chuẩn bị tập dữ liệu

Trong phần này, chúng ta sẽ áp dụng các phép biến đổi sau cho tập dữ liệu.

- Chuyển đổi cột ngày từ integer thành string
- Chia cột ngày thành 3 cột mới (năm, tháng, ngày) để phân chia dữ liệu theo các cột này
- Điền các giá trị còn thiếu vào cột có thể xảy ra bằng 0
- Ánh xạ các giá trị của cột dataQualityGrade thành một giá trị số

1. Quay trở lại project covid-states-daily và chọn **PROJECTS**
2. DataBrew đã suy ra cột dữ liệu date là số nguyên. Chúng ta sẽ convert cột date thành string
Chọn # icon và chọn **string**
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/17.png)

Chọn **Apply**
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/18.png)

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/19.png)

3. Chúng ta sẽ copy cột date trước khi split date thành year, month, day.

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/20.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/21.png)

Click apply.

4. Split cột date thành các cột year, month, day và đổi tên các cột tương ứng

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/22.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/23.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/24.png)


![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/25.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/26.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/27.png)

5. Cột probableCases bị thiếu dữ liệu. Chúng ta sẽ điền vào các chỗ bị thiếu giá trị 0.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/28.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/29.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/30.png)

6. Ánh xạ các giá trị của cột dataQualityGrade thành các giá trị số.

Để điều hướng đến cột dataQualityGrade, hãy nhấp vào danh sách cột thả xuống ở trên cùng, nhập dataQualityGrade vào trường tìm kiếm và nhấp vào **View**.

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/31.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/32.png)

Trong Categorically map column dialog

- Chọn **Map all values**
- Chọn **Map values to numeric values**
- Map dataQualityGrade value hiện tại thành new value
| dataQualityGrade      | value |
| ----------- | ----------- |
| N/A	      | 0       |
| A+   | 1        |
| A  | 2        |
| B  | 3        |
| C  | 4        |
| D  | 5        |

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/33.png)

Để tất cả các cài đặt khác làm mặc định. Nhấp vào **Apply**

Sau phép biến đổi này, cột mới dataQualityGrade_mapped có kiểu double, hãy chuyển cột này thành số nguyên. Bằng cách nhấp vào # ở trên cùng bên trái của cột mới dataQualityGrade_mapped. Nhấp vào **Apply** ở phía bên phải để xác nhận thay đổi.

7. Chúng ta đã sẵn sàng xuất bản recipe để sử dụng trong DataBrew jobs. Final recipe sẽ trông như thế này

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/34.png)

Nhấp vào nút **Publish** ở đầu công thức.

Tùy chọn nhập mô tả phiên bản và nhấp vào **Publish**
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/553.png)


### Tạo DataBrew Job
1. Chọn **Jobs** 
Mở **Recipe jobs** tab, và click vào **Create job**

Nhập covid-states-daily-prep là job name

Chọn **Create a recipe job**

Chọn covid-states-daily-stats dataset

Chọn covid-states-daily-recipe

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/36.png)

Tại mục **Job output settings** nhập:
- **Output to** Amazon S3.
- **File type**: CSV.
- **Delimiter** Comma(,).
- **Compression** None.
- **S3 bucket owners's Account** Current AWS account.
- **S3 location**: giá trị của key **DataBrewOutputS3Bucket** và thêm /job-outputs/ vào cuối. Ví du: s3://databrew-lab-databrewoutputs3bucket-xxxxx/job-outputs/.

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/37.png)

Click **Settings**
- Chọn **Create a new folder for each job run.**
- **Custom partition by column values**: Enabled. Tiếp theo tìm kiếm và **Add** year, month và day ở phần **Columns to partition by**. Cuối cùng nhấn save

![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/3838.png)

Tại Permissions chọn role từ Output của CloudFomation:
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/35.png)

Chọn **Create and run job**

2. DataBrew job đã được tạo và running
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/39.png)
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/40.png)

### Xem dòng dữ liệu
1. Tại DataBrew, Truy cập covid-states-daily project.

Chọn **Lineage** 
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/45.png)



Chế độ xem này hiển thị nguồn gốc của dữ liệu và các bước chuyển đổi mà dữ liệu đã trải qua.
![Clean](/WorkShopTwo/images/7.DataBrewDataBrew/46.png)


Chúng ta đã hoàn thành DataBrew Lab.
