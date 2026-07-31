---
title: "[Lab-3] Truy vấn sử dụng Dữ liệu VCF"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b>6. </b>"
---

{{% notice info %}}
Khi chạy các job PySpark thông qua Amazon EMR Serverless Application, các thư viện Python khác nhau có thể được đóng gói làm phụ thuộc. Để thực hiện điều này, bạn có thể sử dụng chức năng Python cơ bản, xây dựng môi trường ảo, hoặc cấu hình trực tiếp các tác vụ PySpark để sử dụng thư viện Python. Trang này trình bày từng phương pháp.
{{% /notice %}}

Lab này liên quan đến việc thực hiện các thao tác CRUD và truy vấn trên S3 Tables sử dụng dữ liệu biến thể gen (dữ liệu VCF).

Nội dung trùng lặp với [Lab-1] Truy vấn với Dữ liệu Công khai, với sự khác biệt duy nhất là loại dữ liệu được sử dụng.

Đối với những người tham gia không thuộc lĩnh vực Y tế & Khoa học Đời sống, khuyến nghị tiến hành Lab-1.

**Về Bộ dữ liệu:** Mô tả dữ liệu VCF.

**Thiết lập Môi trường:** Thiết lập Môi trường

Tải lên các file jar để sử dụng dịch vụ Amazon S3 Tables trong EMR Serverless.

**Tùy chọn 1:** Sử dụng thư viện Glow với dữ liệu VCF và Amazon S3 Tables.

- Tải dữ liệu VCF sử dụng thư viện Glow, sau đó nhập dữ liệu vào S3 Tables và thực hiện các thao tác CRUD (Tạo/Đọc/Cập nhật/Xóa).
- Mặc dù có thể sử dụng nhiều công cụ truy vấn khác nhau, đối với dữ liệu VCF, do phụ thuộc vào thư viện Glow, Lab này phải được thực hiện thông qua EMR Studio. (Không thể sử dụng Athena)

**Tùy chọn 2:** Xử lý dữ liệu VCF chỉ sử dụng mã Pyspark mà không cần thư viện Glow, và sử dụng với Amazon S3 Tables.
