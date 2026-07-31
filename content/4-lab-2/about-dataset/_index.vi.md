---
title: "Giới thiệu về Bộ dữ liệu"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>4.1. </b>"
---

{{% notice info "Thông tin" %}}
https://registry.opendata.aws/synthea-coherent-data/

Trong trường hợp dữ liệu lâm sàng bệnh nhân, rất khó sử dụng cho mục đích thực hành workshop do các vấn đề về quyền riêng tư.

Ví dụ, dữ liệu như MIMIC-III chỉ có thể được sử dụng cho các mục đích hạn chế sau khi được phê duyệt sử dụng.

Do đó, chúng ta sẽ sử dụng dữ liệu lâm sàng bệnh nhân được tạo nhân tạo được cung cấp tại URL trên.
{{% /notice %}}

Dữ liệu này được nén trong tệp ZIP và là một **bộ dữ liệu tổng hợp** bao gồm dữ liệu tệp CSV, dữ liệu định dạng FHIR, dữ liệu hình ảnh DICOM và dữ liệu Genome.

Trong workshop này, chúng ta sẽ tiến hành sử dụng dữ liệu tệp CSV từ các tệp này.

---

Dữ liệu CSV bao gồm 16 tệp CSV sau:

- `allergies.csv`: Hồ sơ dị ứng của bệnh nhân (707 bản ghi)
- `careplans.csv`: Kế hoạch chăm sóc được cung cấp cho bệnh nhân (14.115 bản ghi)
- `conditions.csv`: Bệnh hoặc tình trạng được chẩn đoán (35.874 bản ghi)
- `devices.csv`: Thiết bị y tế được cấy ghép hoặc sử dụng bởi bệnh nhân (552 bản ghi)
- `encounters.csv`: Liên hệ giữa bệnh nhân và cơ sở y tế (khám, nhập viện, v.v.) (285.339 bản ghi)
- `imaging_studies.csv`: Kết quả kiểm tra X-quang hoặc hình ảnh y khoa (6.262 bản ghi)
- `immunizations.csv`: Hồ sơ tiêm chủng của bệnh nhân (35.500 bản ghi)
- `medications.csv`: Hồ sơ thuốc đã dùng hoặc đang dùng bởi bệnh nhân (371.210 bản ghi)
- `observations.csv`: Giá trị quan sát y tế hoặc dữ liệu sinh hiệu (1.480.409 bản ghi)
- `organizations.csv`: Nhà cung cấp dịch vụ y tế (2.535 bản ghi)
- `patients.csv`: Thông tin cơ bản của bệnh nhân (3.539 bản ghi)
- `payer_transitions.csv`: Lịch sử thay đổi trạng thái bảo hiểm của bệnh nhân (16.328 bản ghi)
- `payers.csv`: Thông tin nhà cung cấp bảo hiểm (10 bản ghi)
- `procedures.csv`: Hồ sơ thủ thuật y tế và phẫu thuật (134.385 bản ghi)
- `providers.csv`: Nhà cung cấp dịch vụ y tế (13.920 bản ghi)

Các dữ liệu CSV trên liên quan với nhau thông qua các giá trị id.

Ví dụ, dữ liệu conditions (chẩn đoán) bao gồm giá trị id từ dữ liệu patients dưới dạng một cột để tham chiếu.

Chúng ta sẽ tiến hành Lab-2 sử dụng **dữ liệu tổng hợp** này.

{{% notice info "Thông tin" %}}
**AWS Open Data Program là gì?**

AWS Open Data Program (ODP) giúp dân chủ hóa quyền truy cập vào các bộ dữ liệu công khai có giá trị cao bằng cách lưu trữ chúng trong Amazon S3 và giúp dễ dàng khám phá thông qua Registry of Open Data on AWS (RODA).

Các bộ dữ liệu này có thể được sử dụng bởi bất kỳ ai để xây dựng dịch vụ hoặc thực hiện phân tích bằng các công cụ AWS như Amazon EC2, Amazon Athena, AWS Lambda và Amazon EMR.

Bằng cách lưu trữ các bộ dữ liệu trên AWS, người dùng có thể giảm chi phí và độ phức tạp của việc thu thập dữ liệu và tập trung vào việc rút ra thông tin chi tiết.

Dữ liệu trong ODP được lưu trữ trong các S3 bucket công khai, nghĩa là không có hạn chế truy cập. Bất kỳ ai cũng có thể tải xuống và sử dụng dữ liệu trực tiếp từ các vị trí S3 này.
{{% /notice %}}
