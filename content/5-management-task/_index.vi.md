---
title: "Công việc Bảo trì S3 Tables"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b>7. </b>"
---

## 1. Truy cập CloudShell

Truy cập CloudShell để kiểm tra thông tin liên quan đến công việc bảo trì S3 Tables.

Truy cập trang [AWS Console Home](https://console.aws.amazon.com/).

Nhấp nút **CloudShell** ở góc dưới bên trái.

![Cloud Shell](/images/workshop/open-cloudshell.webp)

Cửa sổ terminal nơi bạn có thể chạy AWS CLI sẽ xuất hiện.

Bạn đã truy cập vào môi trường CloudShell nơi bạn có thể sử dụng AWS CLI để quản lý Amazon S3 Tables Maintenance Jobs.

## 2. CLI Quản lý Bảng

Hãy thực hiện các thao tác cần thiết để quản lý công việc bảo trì S3 Tables, như kiểm tra trạng thái job và cấu hình các công việc bảo trì.

{{% notice info %}}
Nếu workshop tiến hành nhanh, có thể không có lịch sử thực thi job nào sau khi tạo bảng. Trong trường hợp này, lỗi có thể xảy ra khi thực thi lệnh CLI.
{{% /notice %}}

Đầu tiên, hãy đặt các biến liên quan đến S3 Tables.

```bash
table_bucket_arn="<table-bucket-arn>"
namespace="<namespace>"
table="<table>"
```

- **table-bucket-arn**: Nhập giá trị ARN Table Bucket phù hợp với từng môi trường
- **namespace**: workshop_namespace
- **table**: individually_disclosed_building_price

### 2.1. Lấy Cấu hình

#### 2.1.1. Trạng thái Job

Kiểm tra trạng thái các công việc bảo trì của bảng.

```bash
aws s3tables get-table-maintenance-job-status \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Trạng thái job](/images/workshop/get-job-status.webp)

| Job | Mô tả |
|-----|-------|
| icebergCompaction | Job nén và lưu trữ một file dữ liệu đơn lẻ theo kích thước chỉ định |
| icebergUnreferencedFileRemoval | Job xóa các file không còn được quản lý bởi snapshot |
| icebergSnapshotManagement | Job dọn dẹp các snapshot đã hết hạn |

#### 2.1.2. Cấu hình Job

Kiểm tra các giá trị cấu hình của công việc bảo trì bảng.

```bash
aws s3tables get-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Cấu hình job](/images/workshop/get-job-configuration.webp)

| Cấu hình | Mô tả |
|-----------|-------|
| icebergCompaction > targetFileSizeMB | Kích thước file dữ liệu sau xử lý nén |
| icebergSnapshotManagement > minSnapshotsToKeep | Số lượng snapshot tối thiểu |
| icebergSnapshotManagement > maxSnapshotAgeHours | Thời gian lưu giữ snapshot tối đa |

### 2.2. Cập nhật Cấu hình

#### 2.2.1. Compaction

Thay đổi cài đặt liên quan đến Compaction trong các giá trị cấu hình công việc bảo trì bảng.

```bash
aws s3tables put-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --type icebergCompaction \
    --namespace $namespace \
    --name $table \
    --value='{"status":"enabled","settings":{"icebergCompaction":{"targetFileSizeMB":64}}}'
```

| Tham số | Mô tả |
|---------|-------|
| status | Trạng thái của cấu hình bảo trì |
| targetFileSizeMB | Kích thước file mục tiêu của bảng tính bằng MB |

#### 2.2.2. Snapshot

Thay đổi cài đặt liên quan đến Snapshot trong các giá trị cấu hình công việc bảo trì bảng.

```bash
aws s3tables put-table-maintenance-configuration \
   --table-bucket-arn $table_bucket_arn \
   --namespace $namespace \
   --name $table \
   --type icebergSnapshotManagement \
   --value '{"status":"enabled","settings":{"icebergSnapshotManagement":{"minSnapshotsToKeep":10,"maxSnapshotAgeHours":120}}}'
```

| Tham số | Mô tả |
|---------|-------|
| status | Trạng thái của cấu hình bảo trì |
| minSnapshotsToKeep | Số lượng snapshot tối thiểu cần giữ |
| maxSnapshotAgeHours | Tuổi tối đa của snapshot trước khi có thể hết hạn |

#### 2.2.3. Xóa File Mồ côi

Thay đổi cài đặt để xóa các file không còn được quản lý trong snapshot bảng.

```bash
aws s3tables put-table-bucket-maintenance-configuration \
   --table-bucket-arn $table_bucket_arn \
   --type icebergUnreferencedFileRemoval \
   --value '{"status":"enabled","settings":{"icebergUnreferencedFileRemoval":{"unreferencedDays":3,"nonCurrentDays":10}}}'
```

| Tham số | Mô tả |
|---------|-------|
| status | Trạng thái của cấu hình bảo trì |
| unreferencedDays | Số ngày một object phải không được tham chiếu trước khi bị đánh dấu là không hiện tại |
| nonCurrentDays | Số ngày một object phải không hiện tại trước khi bị xóa |

#### 2.2.4. Kiểm tra Cấu hình

Kiểm tra các giá trị cấu hình job đã thay đổi.

```bash
aws s3tables get-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Cấu hình job đã cập nhật](/images/workshop/get-updated-job-configuration.webp)

Bạn có thể xác nhận các giá trị cấu hình đã được cập nhật.
