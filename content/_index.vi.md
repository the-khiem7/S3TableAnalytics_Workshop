---
title: "Amazon S3 Tables Hands-on Lab với AWS Analytics Services"
date: 2026-07-31
weight: 0
chapter: true
---

## Tổng quan Workshop

{{% notice info %}}
Sơ đồ bên dưới thể hiện một ví dụ về hệ thống phân tích dựa trên data lake có thể được xây dựng bằng Amazon S3 và S3 Tables.
{{% /notice %}}

![Sơ đồ Workshop](/images/workshop/workshop-diagram.webp)

Như thể hiện trong sơ đồ trên, bằng cách sử dụng dịch vụ Amazon S3 Tables dựa trên Amazon S3 và các dịch vụ AWS Analytics khác nhau có thể truy vấn nó, bạn có thể nhanh chóng xây dựng một môi trường phân tích dữ liệu lớn ổn định với chi phí thấp. Amazon S3 là dịch vụ lưu trữ đối tượng của AWS có độ tin cậy cao và cho phép bạn sử dụng bộ nhớ với giá rất phải chăng. Các tệp dữ liệu được tải lên Amazon S3 có thể được nạp vào Amazon S3 Tables dựa trên Apache Iceberg để xây dựng data lake. Và dữ liệu được nạp vào Amazon S3 Tables có thể được tích hợp dễ dàng với các dịch vụ AWS Analytics, cho phép bạn mở rộng môi trường phân tích dữ liệu một cách an toàn và dễ dàng.

**Trong workshop này, phần khung màu xanh được đề cập, đại diện cho phần quan trọng nhất trong việc xây dựng data lake dựa trên Amazon S3 và S3 Tables.**

{{% notice warning %}}
Workshop này chỉ được hỗ trợ thông qua các phiên workshop được tổ chức bởi các sự kiện AWS,

và không được hỗ trợ cho tài khoản người dùng cá nhân.
{{% /notice %}}

## Bộ dữ liệu

Dữ liệu được đề cập trong workshop này chủ yếu được sử dụng trong lĩnh vực công và bao gồm ba loại dữ liệu:

**Tuy nhiên, việc thực hiện tất cả các thao tác dữ liệu sẽ trùng lặp, và khuyến nghị chọn một bộ dữ liệu hữu ích cho tiến trình workshop.**

**Đối với Lab-1, đây là bộ dữ liệu mới được thêm vào, và các truy vấn phân tích mẫu đã được bổ sung để bạn có thể chạy trực tiếp.**

- [Lab-1] **(Mới thêm)** Dữ liệu phòng khám Synthea (định dạng CSV)
  - Đây là dữ liệu phòng khám bệnh nhân được tổng hợp.
  - Các truy vấn phân tích mẫu đã được thêm vào.
- [Lab-2] Dữ liệu giá đất công bố công khai (định dạng CSV)
  - Đây là dữ liệu công khai về giá đất công bố theo khu vực tại Seoul.
- [Lab-3] **(Tùy chọn)** Dữ liệu biến thể gen (định dạng VCF)
  - Đây là dữ liệu VCF (Variant Call Format) được chia sẻ công khai trên S3.
  - Đây là định dạng tệp để lưu trữ các biến thể được tìm thấy trong bộ gen.
  - Dữ liệu này có thể hữu ích cho lĩnh vực **Healthcare & Lifesciences**.

## Đối tượng mục tiêu

- Các chuyên gia dữ liệu quan tâm đến dịch vụ Amazon S3 Tables và Apache Iceberg.

## Thời lượng

- Dự kiến mất khoảng 2 giờ.
- Có thể thay đổi tùy thuộc vào mức độ hiểu biết của người tham gia về Python và Spark.

## Kỹ năng yêu cầu

- Kiến thức cơ bản về lập trình Python
- Hiểu biết về xử lý dữ liệu bao gồm Spark (Spark SQL, Dataframe, ETL, v.v.)

## Kết quả mong đợi

- Hiểu tổng quan về Apache Iceberg và Amazon S3 Tables.
- Hiểu toàn diện về các phương pháp xử lý dữ liệu sử dụng Spark dựa trên bảng Apache Iceberg.

## Region

- `ap-northeast-2`, `us-east-1`
