---
title: "Tạo Table Bucket"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>3.1. </b>"
---

{{% notice info %}}
Tạo một Table Bucket trong bảng điều khiển AWS.
{{% /notice %}}

1. Truy cập trang [S3 Console](https://console.aws.amazon.com/s3/home).
2. Chọn 'Table buckets' từ menu bên trái.
3. Nhấp 'Create table bucket'.
4. Nhập tên bucket phù hợp trong 'Table bucket name'. Ví dụ: workshop-table-bucket
5. Đánh dấu 'Enable integration' trong phần 'Integration with AWS analytics services'.
   - ![Tạo table bucket](/images/workshop/create_table_bucket.webp)
   - Khi được bật, một Catalog có tên 's3tablescatalog' sẽ được tạo trong AWS Glue Catalog.
   - Catalog này sẽ cho phép tích hợp với các dịch vụ phân tích AWS (Athena, Redshift, EMR, v.v.) sau này.
6. Nhấp nút 'Create table bucket' để tạo Table bucket.
   - ![Tạo table bucket](/images/workshop/create_table_bucket_complete.webp)

{{% notice warning %}}
Việc tạo Table Bucket đã hoàn tất.
{{% /notice %}}
