---
title: "[Tùy chọn] Trực quan hóa"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b>4.7. </b>"
---

## 1. Đăng ký Quicksight

Tạo tài khoản trong Quicksight.

Truy cập trang [Amazon Quicksight](https://quicksight.aws.amazon.com/).

Nhấp nút 'SIGN UP FOR QUICKSIGHT'.

Nhập các giá trị sau và nhấp nút 'Finish' để hoàn tất đăng ký:

- **Email for account notification:** \<Workshop Email\>
- **QuickSight region:** Asia Pacific (Seoul) hoặc US East (N. Virginia)
- **QuickSight account name:** \<User ID\> -> Phải là giá trị duy nhất.
- **Allow access and autodiscovery for these resources:** Đánh dấu các tài nguyên sau:
  - IAM
  - Amazon S3: Đánh dấu và cấu hình như sau trước khi nhấp nút 'Finish'
  - Amazon Athena
  - Add Pixel-Perfect Reports: Bỏ đánh dấu

![Kiểm tra S3](/images/workshop/select_s3.webp)

Khi đăng ký hoàn tất, trang hoàn tất tạo sẽ xuất hiện, nhấp nút 'GO TO QUICKSIGHT' để tiếp tục.

![Tạo hoàn tất](/images/workshop/create_complete.webp)

Xem lại trang Welcome và đóng nó.

Đăng ký Quicksight đã hoàn tất.

## 2. Kiểm soát truy cập

Sử dụng AWS Lake Formation để thiết lập kiểm soát truy cập cho các bảng trong Amazon S3 Tables.

Cấu hình quyền truy cập để công cụ trực quan hóa Quicksight có thể truy cập các bảng trong Amazon S3 Tables.

https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-integrating-aws.html#grant-permissions-tables

### 2.1. Lấy User ARN

Nhấp vào thông tin tài khoản ở góc trên bên phải để sao chép Account ID.

![Lấy Account Id](/images/workshop/get_account_id.webp)

Khởi chạy CloudShell bằng cách nhấp vào liên kết CloudShell ở góc dưới bên trái.

![Mở CloudShell](/images/workshop/open_cloudshell.webp)

Chạy CLI sau để lấy Quicksight Admin ARN.

```bash
aws quicksight list-users --aws-account-id <account_id> --namespace default --region <region>
```

- `<account_id>`: Nhập giá trị Account ID đã sao chép ở bước 1.
- `<region>`: Nhập 'ap-northeast-2' hoặc 'us-east-1'.

Sao chép giá trị tương ứng với "Arn" trong JSON phản hồi. Giá trị này là ARN của tài khoản để cấp quyền truy cập trong Lake Formation.

![Lấy User ARN](/images/workshop/cloudshell.png)

Bạn đã lấy được Quicksight user ARN.

### 2.2. Cấp quyền truy cập dữ liệu

Truy cập trang [AWS Lake Formation](https://console.aws.amazon.com/lakeformation/).

Nhấp nút 'Get started'.

Điều hướng đến menu 'Permissions' > 'Data permissions' ở bên trái, và nhấp nút 'Grant'.

![Lake Formation](/images/workshop/lakeformation.webp)

Nhập các giá trị sau:

**Principals**
- Chọn SAML users and groups
- Nhập SAML ARN: Nhập giá trị ARN của tài khoản đã sao chép trước đó

**LF-Tags or catalog resources**
- Chọn Named Data Catalog resources
- Catalogs: Đánh dấu \<account_id\>:s3tablescatalog/workshop-table-bucket
- Databases: Đánh dấu workshop_namespace
- Tables: Đánh dấu All tables

**Table permissions**
- Table permissions: Đánh dấu Super
- Grantable permissions: Đánh dấu Super

Nhấp nút 'Grant'.

Bạn đã thiết lập quyền truy cập cho Quicksight user ARN đến các bảng trong Amazon S3 Tables.

## 3. Trực quan hóa

Sử dụng Quicksight để trực quan hóa dữ liệu trong Amazon S3 Tables.

### 3.1. Thêm Datasets

Truy cập trang [Amazon Quicksight](https://quicksight.aws.amazon.com/).

Chọn 'Datasets' từ menu bên trái.

Nhấp nút 'NEW DATASET' ở góc trên bên phải.

Chọn 'Athena'.

Nhập tên mong muốn cho 'Data source name' và nhấp nút 'Create data source'.

Nhấp nút 'Use custom SQL'.

Xóa văn bản 'New custom SQL' và nhập tên bảng 'patients'.

Dán truy vấn sau vào ô nhập liệu bên dưới.

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.patients
```

Nhấp nút 'Edit/Preview data'.

Trên màn hình tiếp theo, nhấp nút 'Apply' để xem dữ liệu.

Sau khi xem dữ liệu, nhấp nút 'Close'.

Nhấp nút 'Add data' ở góc trên bên phải.

![Thêm dataset](/images/workshop/add-dataset.png)

Chọn 'Data source' trong menu Select, chọn data source bạn đã tạo và nhấp nút 'Select'.

Lặp lại các bước 6-13 cho các Custom SQL sau:

**Custom SQLs**

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.encounters
```

![Thêm encounters](/images/workshop/add-dataset-encounters.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.conditions
```

![Thêm conditions](/images/workshop/add-dataset-conditions.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.observations
```

![Thêm observations](/images/workshop/add-dataset-observations.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.procedures
```

![Thêm procedures](/images/workshop/add-dataset-procedures.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.medications
```

![Thêm medications](/images/workshop/add-dataset-medications.webp)

Sau khi thêm tất cả Custom SQL, bạn sẽ thấy màn hình như bên dưới.

![Thêm datasets](/images/workshop/add-datasets.webp)

Việc thêm dữ liệu đã hoàn tất.

### 3.2. Cấu hình Join

Thiết lập mối quan hệ join giữa các dataset đã thêm.

{{% notice info %}}
- patients - encounters -> Left
- encounters - conditions -> left
- encounters - observations -> left
- encounters - procedures -> left
- encounters - medications -> left
{{% /notice %}}

Thiết lập join giữa 'patients' và 'encounters' như hình bên dưới, và nhấp 'Apply' để áp dụng thay đổi.

- patients Join Column = id
- encounters Join Column = patient
- Join type = Left

![Cấu hình Join](/images/workshop/join-config-1.webp)

Kéo dữ liệu 'conditions' để join với 'encounters'. Thiết lập join và nhấp 'Apply' để áp dụng thay đổi.

- encounters Join Column = id
- conditions Join Column = encounter
- Join type = Left

![Cấu hình Join](/images/workshop/join-config-2.png)

Kéo dữ liệu 'observations' để join với 'encounters'. Thiết lập join và nhấp 'Apply' để áp dụng thay đổi.

- encounters Join Column = id
- observations Join Column = encounter
- Join type = Left

![Cấu hình Join](/images/workshop/join-config-3.png)

Kéo dữ liệu 'procedures' để join với 'encounters'. Thiết lập join và nhấp 'Apply' để áp dụng thay đổi.

- encounters Join Column = id
- procedures Join Column = encounter
- Join type = Left

![Cấu hình Join](/images/workshop/join-config-4.png)

Kéo dữ liệu 'medications' để join với 'encounters'. Thiết lập join và nhấp 'Apply' để áp dụng thay đổi.

- encounters Join Column = id
- medications Join Column = encounter
- Join type = Left

![Cấu hình Join](/images/workshop/join-config-5.png)

Sau khi hoàn tất các bước trên, đợi danh sách cột đầy đủ và xem trước dữ liệu xuất hiện ở bên trái.

![Cấu hình Join](/images/workshop/join-config-final.webp)

Khi 'Query mode' ở góc dưới bên trái và nút 'SAVE & PUBLISH' ở góc trên bên phải được kích hoạt, thiết lập như sau:

- Query mode = SPICE
- Nhấp nút 'SAVE & PUBLISH'.

{{% notice info %}}
**SPICE query mode là gì?**

SPICE là công cụ in-memory của Amazon QuickSight được thiết kế để xử lý và trực quan hóa dữ liệu nhanh chóng. Công cụ này tải dữ liệu vào bộ nhớ in-memory của QuickSight, cải thiện đáng kể hiệu suất truy vấn. Mỗi người dùng được cung cấp 10GB dung lượng SPICE mặc định. Có thể mua thêm dung lượng nếu cần nhiều hơn mức phân bổ mặc định. Phí được tính hàng tháng theo GB. Trong workshop này, chúng ta sử dụng chế độ SPICE để trực quan hóa nhanh hơn.
{{% /notice %}}

Sau thông báo Publish complete, nhấp nút 'CLOSE'.

Nhấp vào dataset đã tạo để xem chi tiết. Bạn có thể thấy trạng thái REFRESH là 'In progress'.

![Chi tiết Dataset](/images/workshop/dataset-details.webp)

REFRESH thường mất 5-10 phút để hoàn tất.

Đợi REFRESH hoàn tất. Sau khi hoàn tất, bạn sẽ thấy chi tiết dataset như bên dưới.

![Chi tiết Dataset](/images/workshop/dataset-details-2.png)

Cấu hình join giữa các dataset và thiết lập SPICE đã hoàn tất.

Tất cả công việc chuẩn bị cho dataset đã hoàn tất.

### 3.3. Trực quan hóa Dashboard

Tự do trực quan hóa dữ liệu.

Nhấp nút 'USE IN ANALYSIS' ở góc trên bên phải.

Trên màn hình New sheet, thực hiện các cài đặt cần thiết và nhấp nút 'CREATE'.

Bây giờ bạn có thể tự do trực quan hóa dữ liệu như hình bên dưới.

![Trực quan hóa](/images/workshop/visualization.webp)

Sau khi xuất bản dashboard, bạn có thể xuất nó ở nhiều định dạng khác nhau:

Nó cung cấp các loại tệp tài liệu như PDF, liên kết và dashboard nhúng. Mã nhúng được cung cấp như bên dưới, và bạn cũng có thể thiết lập quyền truy cập cho từng người dùng.

```html
<iframe
    width="960"
    height="720"
    src="https://<region>.quicksight.aws.amazon.com/sn/embed/share/accounts/<account_id>/dashboards/<dashboard_id>?directory_alias=<quicksight_user_name>">
</iframe>
```

---

Bạn đã trực quan hóa dữ liệu lâm sàng được tải vào Amazon S3 Tables.

Chỉ cần tải dữ liệu vào Amazon S3 Tables, bạn có thể dễ dàng phân tích và trực quan hóa dữ liệu bằng các dịch vụ AWS Analytics khác nhau.
