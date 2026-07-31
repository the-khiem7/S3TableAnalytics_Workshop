---
title: "Thiết lập Môi trường"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>5.2. </b>"
---

## 1. Tải lên file CSV và Apache Iceberg jar vào S3

Chúng ta sẽ tải lên file dữ liệu CSV, file Apache Iceberg Runtime Jar và file Glow Jar cần thiết cho workshop này lên S3 bucket.

Tải xuống file CSV của workshop,
- AL_D151_11_20240125_UTF8.csv
- AL_D151_41_20250120_200000_UTF8.csv

Các file này là dữ liệu công khai từ Hàn Quốc.

- **AL_D151_11_20240125_UTF8.csv:** File dữ liệu chứa giá đất công bố công khai theo khu vực tại Seoul.
- **AL_D151_41_20250120_200000_UTF8.csv:** File dữ liệu chứa 100.000 giá đất công bố công khai từ Seoul và 100.000 giá đất công bố công khai từ Gyeonggi-do. (Sẽ được sử dụng cho bài thực hành truy vấn Merge.)

Tải xuống file Apache Iceberg runtime jar cho Amazon S3 Tables, **s3-tables-catalog-for-iceberg-runtime-0.1.5.jar**.

Truy cập trang [S3 Console](https://console.aws.amazon.com/s3).

Chọn 'General purpose buckets' từ menu bên trái.

Nhấp vào bucket 'workshop-bucket-xxxxxx'.

Nhấp 'Create folder' và tạo thư mục có tên 'lib'.

Nhấp 'Create folder' lần nữa và tạo thư mục có tên 'data'.

Nhấp 'Upload', sau đó sử dụng nút 'Add files' để tải lên các file đã tải xuống vào các đường dẫn tương ứng.

{{% notice info %}}
Tải lên file Jar (s3-tables-catalog-for-iceberg-runtime-0.1.5.jar) vào đường dẫn 's3://${bucket}/lib/'.

Tải lên hai file CSV (AL_D151_11_20240125_UTF8.csv và AL_D151_41_20250120_200000_UTF8.csv) vào đường dẫn 's3://${bucket}/data/'.
{{% /notice %}}

![Tải lên file dữ liệu](/images/workshop/upload-data.webp)

![Tải lên file jar](/images/workshop/upload-jar.webp)

Bạn đã hoàn thành việc tải lên các file cần thiết cho workshop lên S3.
