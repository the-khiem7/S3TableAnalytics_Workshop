---
title: "Tạo Namespace"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>3.2. </b>"
---

## Cài đặt S3 cho kết quả truy vấn Athena

{{% notice info %}}
Chỉ định vị trí S3 để lưu trữ kết quả truy vấn Athena. Nếu không được chỉ định, các truy vấn không thể được thực thi.
{{% /notice %}}

1. Truy cập trang [Amazon Athena Console](https://console.aws.amazon.com/athena/home).

2. Nhấp nút 'Explore the query editor' ở bên phải.

3. Nhấp nút 'Edit settings' để chỉ định vị trí S3 cho việc lưu trữ kết quả truy vấn Athena. (Nếu không được chỉ định, các truy vấn Athena không thể được thực thi.)
   - ![Cài đặt S3 cho kết quả truy vấn](/images/workshop/athena_query_result_s3_setting.webp)

   1. Nhấp nút 'Browse S3'.
   2. Chọn một S3 bucket hiện có và nhấp nút 'Choose'.
   3. Xác nhận rằng bucket đã chọn được nhập trong 'Location of query result'.
      - Bạn có thể thêm đường dẫn con sau tên bucket.
      - Ví dụ: s3://workshop-bucket-2ybfnmiyhg8a/query_result
   4. Nhấp nút 'Save' để hoàn tất cài đặt.
   5. Sau khi cài đặt, nhấp tab 'Editor' để di chuyển đến trang trình soạn thảo truy vấn.

{{% notice warning %}}
Cài đặt vị trí lưu trữ S3 cho kết quả truy vấn Athena đã hoàn tất.
{{% /notice %}}

---

## Tạo Namespace

{{% notice info %}}
Tạo Namespace sẽ được sử dụng trong S3 Tables.
{{% /notice %}}

1. Thiết lập cài đặt bên trái trên trang Editor như sau:
   - ![Tạo namespace](/images/workshop/athena_create_namespace.webp)
   - Data source: `AwsDataCatalog`
   - Catalog: `s3tablescatalog/workshop-table-bucket`

2. Nhập truy vấn tạo Namespace sau trong Query Editor:

```sql
CREATE DATABASE IF NOT EXISTS workshop_namespace
```

3. Nhấp nút 'Run' để thực thi truy vấn trên.

4. Nếu truy vấn được thực thi thành công, bạn sẽ thấy 'workshop_namespace' đã được tạo trong phần 'Database' của bảng cài đặt bên trái.

{{% notice warning %}}
Việc tạo Namespace đã hoàn tất.
{{% /notice %}}
