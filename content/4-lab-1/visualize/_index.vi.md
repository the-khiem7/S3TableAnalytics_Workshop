---
title: "[Tùy chọn] Trực quan hóa"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b>5.5. </b>"
---

## 1. Đăng ký Quicksight

Đăng ký tài khoản Quicksight.

Truy cập trang [Amazon Quicksight](https://quicksight.aws.amazon.com/).

Nhấp nút 'SIGN UP FOR QUICKSIGHT'.

Nhập các giá trị sau và nhấp nút 'Finish' để hoàn tất đăng ký:

- **Email for account notification:** \<Workshop Email\>
- **QuickSight region:** Asia Pacific (Seoul) hoặc US East (N. Virginia)
- **QuickSight account name:** \<User ID\> -> Đây phải là giá trị duy nhất.
- **Allow access and autodiscovery for these resources:** \<Chọn các tài nguyên bên dưới\>
  - IAM
  - Amazon S3: Chọn và thiết lập như sau, sau đó nhấp nút 'Finish'
  - Amazon Athena
  - Add Pixel-Perfect Reports: Bỏ chọn

![Chọn S3](/images/workshop/quicksight-select-s3.webp)

Khi đăng ký hoàn tất, trang hoàn thành tạo sẽ xuất hiện. Nhấp nút 'GO TO QUICKSIGHT' để tiếp tục.

![Tạo hoàn tất](/images/workshop/quicksight-create-complete.webp)

Xem trang Welcome và đóng lại.

Việc đăng ký Quicksight đã hoàn tất.

## 2. Kiểm soát Truy cập

Sử dụng AWS Lake Formation để thiết lập quyền kiểm soát truy cập cho các bảng trong Amazon S3 Tables.

Chúng ta sẽ cấu hình cài đặt để cho phép công cụ trực quan hóa Quicksight truy cập các bảng trong Amazon S3 Tables.

Bước này rất quan trọng để cho phép Quicksight trực quan hóa dữ liệu từ Amazon S3 Tables.

### 2.1. Lấy ARN Người dùng

Nhấp vào thông tin tài khoản ở góc trên bên phải để sao chép Account ID.

![Lấy Account Id](/images/workshop/quicksight-get-account-id.webp)

Mở CloudShell bằng cách nhấp liên kết CloudShell ở góc dưới bên trái.

![Mở CloudShell](/images/workshop/quicksight-open-cloudshell.webp)

Chạy lệnh CLI sau để kiểm tra ARN Admin Quicksight:

```bash
aws quicksight list-users --aws-account-id <account_id> --namespace default --region <region>
```

- `account_id`: Nhập giá trị Account ID đã sao chép ở bước 1.
- `region`: Nhập 'ap-northeast-2' hoặc 'us-east-1'.
- Sao chép giá trị tương ứng với trường "Arn" từ JSON phản hồi.
- Giá trị này là ARN của Tài khoản sẽ được cấp quyền truy cập trong Lake Formation.

![Lấy ARN Người dùng](/images/workshop/quicksight-cloudshell.png)

Bạn đã lấy được giá trị ARN người dùng Quicksight.

### 2.2. Cấp Quyền Dữ liệu

Truy cập trang [AWS Lake Formation](https://console.aws.amazon.com/lakeformation).

Nhấp nút 'Get started'.

Điều hướng đến 'Permissions' > 'Data permissions' trong menu bên trái, sau đó nhấp nút 'Grant'.

Nhập các giá trị sau:

**Principals**
- Chọn SAML users and groups
- Nhập SAML ARN: Nhập giá trị ARN của Tài khoản đã sao chép trước đó

**LF-Tags or catalog resources**
- Chọn Named Data Catalog resources
- Catalogs: Chọn \<account_id\>:s3tablescatalog/workshop-table-bucket
- Databases: Chọn workshop_namespace
- Tables: Chọn All tables

**Table permissions**
- Table permissions: Chọn Super
- Grantable permissions: Chọn Super

![Lake Formation](/images/workshop/lakeformation.webp)

Nhấp nút 'Grant'.

Bạn đã thiết lập quyền truy cập cho ARN người dùng Quicksight vào các bảng trong Amazon S3 Tables.

## 3. Trực quan hóa

Sử dụng Quicksight để trực quan hóa dữ liệu từ Amazon S3 Tables.

### 3.1. Bộ dữ liệu

Truy cập trang [Amazon Quicksight](https://quicksight.aws.amazon.com/).

Chọn 'Datasets' từ menu bên trái.

Nhấp nút 'New Dataset' ở góc trên bên phải.

Chọn 'Athena'.

Nhập tên mong muốn cho 'Data source name' và nhấp nút 'Create data source'.

Nhấp nút 'Use Custom SQL'.

Nhập truy vấn sau và nhấp nút 'Edit/Preview data':

```sql
SELECT * 
FROM "s3tablescatalog"."workshop_namespace"."individually_disclosed_building_price"
```

![Thêm datasource](/images/workshop/quicksight-add-datasource.png)

Bạn sẽ được chuyển đến màn hình như bên dưới. Nhấp nút 'Apply' để xem dữ liệu.

[Lưu ý] 'SPICE' là tính năng liên quan đến bộ nhớ đệm dữ liệu giúp cải thiện tốc độ truy vấn.

![Thêm datasource](/images/workshop/quicksight-add-datasource-2.png)

Khi dữ liệu hiển thị chính xác, nhấp nút 'Close'.

Bạn sẽ được chuyển đến màn hình như bên dưới. Nhấp nút 'PUBLISH & VISUALIZE'.

Nhấp nút 'CREATE' trong New sheet.

![Trực quan hóa](/images/workshop/quicksight-visualize.webp)

Bạn đã xuất bản thành công bảng từ Amazon S3 Tables dưới dạng Dataset trong Quicksight.

### 3.2. Trực quan hóa Dashboard

Hãy tự do tạo các trực quan hóa của riêng bạn.

Khi bạn xuất bản một Dashboard, bạn có thể xuất nó ở nhiều định dạng khác nhau:
- Nó cung cấp các file dạng tài liệu như PDF, Links, Embedded dashboards, v.v.
- Mã Embedded được cung cấp như bên dưới, và quyền truy cập có thể được thiết lập cho từng người dùng.

```html
<iframe
    width="600"
    height="400"
    src="https://us-east-1.quicksight.aws.amazon.com/sn/embed/share/accounts/111122223333/dashboards/1a1ac6ed-39c2-4711-8220-99e6e7fa20b0?directory_alias=yourcompany">
</iframe>
```
