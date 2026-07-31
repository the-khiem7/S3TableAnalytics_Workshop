---
title: "[Lab-1] Truy vấn sử dụng dữ liệu lâm sàng"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>4. </b>"
---

{{% notice info %}}
Với `dữ liệu lâm sàng` được tổng hợp, chúng ta sẽ thực hiện CRUD và truy vấn trên S3 Tables.
{{% /notice %}}

- [Về bộ dữ liệu](about-dataset/): Mô tả dữ liệu lâm sàng tổng hợp của bệnh nhân sẽ được xử lý.
- [Thiết lập môi trường](env-setup/): Thiết lập môi trường cần thiết cho bài lab.
  - Tải lên các tệp Jar để sử dụng dịch vụ Amazon S3 Tables trong EMR Serverless.
- [Tạo S3 Tables](create-s3-tables/): Tạo bảng trong Amazon S3 Tables.
- [Merge, Update, Delete bảng Iceberg](merge-s3-tables/): Thực hiện Merge, Update và Delete trên các bảng đã tạo.
- [Du hành thời gian](time-travel/): Thực hiện truy vấn Du hành thời gian của Iceberg.
- [SQL Playground](query-s3-tables/): Thực hiện các truy vấn phân tích đa dạng.
- [Trực quan hóa](visualize/): Trực quan hóa dữ liệu trong Amazon S3 Tables bằng Quicksight.
