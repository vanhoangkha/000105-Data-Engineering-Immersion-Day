---
title : "Athena and QuickSight"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

### Step
- Yêu cầu
- Bắt đầu
- Truy vấn data với Amazon Athena
- Dựng Amazon QuickSight Dashboard
- Set up QuickSight
- Tạo QuickSight Charts
- Tạo QuickSight Parameters
- Tạo  QuickSight Filter
- Thêm Calculated Fields
- Amazon QuickSight ML-Insights (Optional)
- Athena Workgroups to Control Query Access and Costs (Optional)
- Workflow setup to separate workloads
- Explore the features of workgroups
- Managing Query Usage and Cost
- Sử dụng tag để phân bổ chi phí

### Yêu cầu
Hoàn thành [Ingestion with DMS](../../3-IngestionwithDMS/_index.md) và [Transforming data with Glue ETL](../../4-TransformingdatawithGlue/_index.md) labs.

### Bắt đẩu
Trong bài lab này, chúng ta sẽ hoàn thành các task sau:
1. Truy vấn dữ liệu và tạo view với Amazon Athena
2. Athena Workgroups để Điều khiển query access và costs
3. Dựng dashboard với Amazon QuickSight

### Truy vấn dữ liệu với Amazon Athena
1. Truy cập Athena

![Athena](/WorkShopTwo/images/5.fwd/1.png) 

2. Chọn database **"ticketdata"**
![Athena](/WorkShopTwo/images/5.fwd/2.png) 

3. Chọn table **"parquet_sporting_event_ticket"** để xem các trường
4. Copy đoạn SQL sau và nhấn **Run**
```
SELECT
e.id AS event_id,
e.sport_type_name AS sport,
e.start_date_time AS event_date_time,
h.name AS home_team,
a.name AS away_team,
l.name AS location,
l.city
FROM parquet_sporting_event e,
parquet_sport_team h,
parquet_sport_team a,
parquet_sport_location l
WHERE
e.home_team_id = h.id
AND e.away_team_id = a.id
AND e.location_id = l.id;
```

Kết quả như sau:

![Athena](/WorkShopTwo/images/5.fwd/3.png) 
![Athena](/WorkShopTwo/images/5.fwd/4.png) 

5. Chọn **Create** và **View from query**

6. Đặt tên view là sporting_event_info vào click **Create**
![Athena](/WorkShopTwo/images/5.fwd/5.png) 

View đã được khởi tạo
![Athena](/WorkShopTwo/images/5.fwd/6.png) 


7. Copy đoạn Sql sau sang một tab mới
```
SELECT t.id AS ticket_id,
e.event_id,
e.sport,
e.event_date_time,
e.home_team,
e.away_team,
e.location,
e.city,
t.seat_level,
t.seat_section,
t.seat_row,
t.seat,
t.ticket_price,
p.full_name AS ticketholder
FROM sporting_event_info e,
parquet_sporting_event_ticket t,
parquet_person p
WHERE
t.sporting_event_id = e.event_id
AND t.ticketholder_id = p.id
```
![Athena](/WorkShopTwo/images/5.fwd/7.png) 

8. Lưu lại với tên: create_view_sporting_event_ticket_info
![Athena](/WorkShopTwo/images/5.fwd/8.png) 

Click **Run**

![Athena](/WorkShopTwo/images/5.fwd/9.png) 


Kết quả xuất hiện như sau:
![Athena](/WorkShopTwo/images/5.fwd/10.png) 

9. Đặt tên view là: sporting_event_ticket_info và click **Create**
![Athena](/WorkShopTwo/images/5.fwd/11.png) 

10. Copy đoạn Sql sau sang một tab mới
```
SELECT
sport,
count(distinct location) as locations,
count(distinct event_id) as events,
count(*) as tickets,
avg(ticket_price) as avg_ticket_price
FROM sporting_event_ticket_info
GROUP BY 1
ORDER BY 1;
```

Lưu với tên là: analytics_sporting_event_ticket_info
![Athena](/WorkShopTwo/images/5.fwd/12.png) 

Click Run

![Athena](/WorkShopTwo/images/5.fwd/13.png) 

### Build an Amazon QuickSight Dashboard
#### Set up QuickSight
1. Truy cập **QuickSight**
![Athena](/WorkShopTwo/images/5.fwd/15.png) 

2. **Sign up for QuickSight.**

![Athena](/WorkShopTwo/images/5.fwd/16.png) 

3. Chọn Enterprise Version

4. Continue
![Athena](/WorkShopTwo/images/5.fwd/17.png) 

5. Chọn No, Maybe Later, Ở màn hình mới để mặc định.
6. Chọn region và check vào các box, enable auto discovery, Amazon Athena, and Amazon S3. Nhập tên và email cho account QuickSight
7. Chọn DMS bucket và chọn **Finish**

![Athena](/WorkShopTwo/images/5.fwd/18.png) 

![Athena](/WorkShopTwo/images/5.fwd/19.png) 

Nhấn **Finish**

8. Chọn New analysis
![Athena](/WorkShopTwo/images/5.fwd/20.png) 

9. Chọn **New dataset**
![Athena](/WorkShopTwo/images/5.fwd/21.png) 

10. Tại màn Create a Dataset, chọn Athena

![Athena](/WorkShopTwo/images/5.fwd/22.png) 

11. Data source name: ticketdata-qs, và chọn **Validate connection**

12. Click **Create data source**
![Athena](/WorkShopTwo/images/5.fwd/23.png) 

13. Chọn database **ticketdata**

14. Chọn bảng **sporting_event_ticket_info** và nhấn **Select**

![Athena](/WorkShopTwo/images/5.fwd/24.png) 

15. Chọn option **Import to SPICE for quicker analytics** và nhấn **Visualize**. 

![Athena](/WorkShopTwo/images/5.fwd/25.png) 

Chúng ta có thể quan sát thấy QuickSight Visualize interface và bắt đầu xây dựng dashboard

![Athena](/WorkShopTwo/images/5.fwd/26.png) 

### Tạo QuickSight Charts
Trong phần này chúng ta sẽ cùng tìm hiểu các loại chart

1. Chọn cột ticket_price để tạo chart.
2. Chọn **expand icon** và chọn **Show as Currency**
![Athena](/WorkShopTwo/images/5.fwd/27.png) 

3. Click **Add button** từ Visuals pane
- Visual types: Vertical bar chart
- X-axis Fields list chọn **event_date_time**
- Y-axis chọn **ticket_price** từ Field list.

![Athena](/WorkShopTwo/images/5.fwd/28.png) 

4. Bạn có thể kéo và di chuyển các hình ảnh khác để điều chỉnh không gian trong trang tổng quan. Trong danh sách Trường, nhấp và kéo trường Seat_level vào hộp Nhóm/Màu. Bạn cũng có thể sử dụng thanh trượt bên dưới trục x để vừa với tất cả dữ liệu.

![Athena](/WorkShopTwo/images/5.fwd/29.png) 

5. Trong Visuals pane, chọn **Vertical bar chart** và **Clustered bar combo chart** icon

6. Tại Fields list, chọn **ticketholder** 

7. Tại **Lines** box, chọn **Aggregate: Count Distinct**. 

![Athena](/WorkShopTwo/images/5.fwd/30.png) 

8. Chọn **insight** icon

![Athena](/WorkShopTwo/images/5.fwd/31.png) 


### Tạo QuickSight Parameters

Trong phần này chúng ta sẽ tạo parameters cho dashboard và thêm filter.

1. Chọn Parameters từ tool bar

![Athena](/WorkShopTwo/images/5.fwd/32.png) 

2. Chọn **Add** để tạo parameter
3. Đặt tên là EventFrom
4. Chọn Data type là Datetime
5. Time granularity là Hour
6. Hãy chọn giá trị từ lịch làm ngày bắt đầu có sẵn trong biểu đồ của bạn cho event_date_time

7. Chọn **Create** và đóng dialog
![Athena](/WorkShopTwo/images/5.fwd/33.png) 

8. Tạo thêm parameter với thông số sau:
- Name: EventTo
- Chọn Data type là Datetime
- Time granularity là Hour
- Hãy chọn giá trị từ lịch làm ngày bắt đầu có sẵn trong biểu đồ của bạn cho event_date_time
- Click **Create**

![Athena](/WorkShopTwo/images/5.fwd/35.png) 

9. Trong màn hình tiếp theo. Click vào 3 chấm của **EventFrom** parameter và chọn **Add control**

![Athena](/WorkShopTwo/images/5.fwd/36.png) 

10. Nhập name là Event From và nhấn **Add**
![Athena](/WorkShopTwo/images/5.fwd/37.png) 

11. Tương tự với **EventTo** 
![Athena](/WorkShopTwo/images/5.fwd/38.png) 

Bây giờ bạn có thể xem và mở rộng phần Kiểm soát phía trên biểu đồ.

![Athena](/WorkShopTwo/images/5.fwd/39.png) 

### Tạo QuickSight Filter

1. Chọn **Filter**
2. Click (+) để thêm filter event_date_time

![Athena](/WorkShopTwo/images/5.fwd/40.png) 

3. Chọn edit
![Athena](/WorkShopTwo/images/5.fwd/41.png) 

4. Chọn **All applicable visuals** và **Applied To** dropdown

5. Filter type chọn **Date & Time range** và Condition là **Between**
6. Chọn option **Use Parameter**, chọn **Yes**

7. Start date parameter: **EventFrom**
8. End date parameter: **EventTo**
![Athena](/WorkShopTwo/images/5.fwd/42.png) 

9. Click **Apply**.

### Thêm Calculated Fields

1. Chọn **Add Calculated Field** 
![Athena](/WorkShopTwo/images/5.fwd/43.png) 

2. Nhập name là: event_day_of_week

3. Formula: extract('WD',{event_date_time})

4. Click **Save**
![Athena](/WorkShopTwo/images/5.fwd/44.png) 

5. Thêm một trường được tính toán khác với các thuộc tính sau:
- Calculated field name: event_hour_of_day
- Formula: extract('HH',{event_date_time})

6. Chọn Visualize icon từ the Tool bar và nhấn **Add visual**

![Athena](/WorkShopTwo/images/5.fwd/45.png) 

7. Field type" scatter plot

8. Tại Fields list, Chọn các thuộc tính sau: 
- X-axis: "event_hour_of_day"
- Y-axis: "event_day_of_week"
- Size: "ticket_price"
![Athena](/WorkShopTwo/images/5.fwd/46.png) 

Pushlish hoặc Share
![Athena](/WorkShopTwo/images/5.fwd/47.png) 

Dashboard là ảnh chụp nhanh phân tích chỉ đọc mà bạn có thể chia sẻ với những người dùng Amazon QuickSight khác nhằm mục đích báo cáo. Trong Dashboard, những người dùng khác vẫn có thể xem hình ảnh và dữ liệu nhưng điều đó sẽ không sửa đổi tập dữ liệu.

Bạn có thể chia sẻ phân tích với một hoặc nhiều người dùng khác mà bạn muốn cộng tác để tạo hình ảnh. Phân tích cung cấp các cách sử dụng khác để ghi và sửa đổi tập dữ liệu.

### Cost Allocation Tags

Khi bạn tạo hai nhóm làm việc: nhóm làm việcA và nhóm làm việcB, bạn cũng đã tạo tên dưới dạng thẻ. Các thẻ này có thể được sử dụng trong bảng điều khiển Quản lý chi phí và thanh toán để xác định mức sử dụng cho mỗi nhóm làm việc.

Ví dụ: bạn có thể tạo một bộ thẻ cho nhóm làm việc trong tài khoản của mình để giúp bạn theo dõi chủ sở hữu nhóm làm việc hoặc xác định nhóm làm việc theo mục đích của họ. Bạn có thể xem các thẻ cho một nhóm làm việc trong trang “Xem chi tiết” cho nhóm làm việc đang được xem xét.

Bạn có thể thêm tag sau khi đã tạo workgroup. Để tạo tag:

1. [Athena console](https://console.aws.amazon.com/athena/home), chọn tab Workgroups và chọn workgroup.
2. Chọn tab tags.
3. Trên tab tags, bấm vào Manage tags, sau đó chỉ định key và value cho tag.
4. Khi bạn hoàn tất, hãy nhấp vào **Save Changes**.

![Athena](/WorkShopTwo/images/5.fwd/82.png) 
