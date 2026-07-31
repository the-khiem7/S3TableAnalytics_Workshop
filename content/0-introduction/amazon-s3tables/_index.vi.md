---
title: "Amazon S3 Tables"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>1.1. </b>"
---

{{% notice info %}}
Amazon S3 Tables là một dịch vụ mới được AWS chính thức ra mắt vào cuối năm 2023. Đây là dịch vụ dữ liệu dạng bảng dựa trên lưu trữ đối tượng Amazon S3.

Nó đơn giản hóa việc xây dựng và quản lý data lake. Được xây dựng trên định dạng bảng mã nguồn mở Apache Iceberg, cho phép các nhà phân tích dữ liệu và kỹ sư dễ dàng quản lý dữ liệu được lưu trữ trong S3.
{{% /notice %}}

Trước khi đi sâu vào Amazon S3 Tables, hãy cùng tìm hiểu về Apache Iceberg trước.

## 1. Apache Iceberg

Apache Iceberg là một định dạng bảng mở để quản lý các tập dữ liệu phân tích quy mô lớn.

Nó được khởi xướng bởi Netflix và hiện đang được quản lý bởi Apache Software Foundation.

Iceberg giải quyết các hạn chế của định dạng bảng Hive hiện tại và cung cấp giao dịch ACID, tiến hóa schema, tối ưu hóa phân vùng và quản lý metadata cho các tập dữ liệu quy mô lớn.

https://github.com/apache/iceberg

### 1.1. Các tính năng chính

- Hỗ trợ giao dịch ACID: Duy trì tính nhất quán dữ liệu ngay cả trong các thao tác song song.
- Tiến hóa Schema: Hỗ trợ thêm, xóa và đổi tên cột mà không cần ngừng hoạt động.
- Phân vùng ẩn: Cho phép tối ưu hóa truy vấn mà không cần hiển thị các cột phân vùng.
- Du hành thời gian: Khả năng quay lại hoặc truy vấn các thời điểm cụ thể hoặc các snapshot cụ thể.
- Đọc tăng dần: Hiệu quả vì chỉ cần đọc dữ liệu đã thay đổi.
- Hỗ trợ Catalog có thể cắm: Có thể tích hợp với nhiều metastore khác nhau như Hadoop, AWS Glue, Hive Metastore, v.v.

### 1.2. Kiến trúc

![Kiến trúc Apache Iceberg](/images/workshop/apache-iceberg-architecture.webp)

#### 1.2.1. Các thành phần chính

- Iceberg Catalog
  Chứa con trỏ đến tệp metadata hiện tại của bảng.
  Mỗi khi dữ liệu thay đổi, một tệp metadata mới được tạo ra, và con trỏ trỏ đến tệp metadata gần nhất.
  Khía cạnh này cho phép ACID trong các bảng Iceberg.
- Lớp Metadata
  Table Metadata (tệp JSON): Lưu trữ thông tin như schema bảng, vị trí, thông tin phân vùng, dấu thời gian snapshot, v.v.
  Manifest List (tệp Avro): Tệp metadata trỏ đến danh sách tệp của một snapshot đơn lẻ.
  Manifest (tệp Avro): Danh sách các tệp dữ liệu thực tế và thống kê phân vùng/hàng cho mỗi tệp.
- Lớp Dữ liệu
  Data Files (Parquet, ORC, Avro): Các tệp dữ liệu thực tế.

#### 1.2.2. Cấu trúc dựa trên Snapshot

- Tất cả thay đổi dữ liệu được thực hiện bằng cách tạo các snapshot mới,
- Các snapshot hiện có được duy trì nguyên trạng, giúp hỗ trợ Du hành thời gian và Quay lại dễ dàng.

### 1.3. Các tính năng chi tiết

#### 1.3.1. Tiến hóa Schema

```sql
ALTER TABLE sales ADD COLUMN region STRING;
ALTER TABLE sales DROP COLUMN discount;
ALTER TABLE sales RENAME COLUMN price TO unit_price;
```

#### 1.3.2. Du hành thời gian

```sql
-- Query based on a specific snapshot
SELECT * FROM sales.snapshots WHERE snapshot_id = 123456789;
SELECT count(*) FROM sales FOR TIMESTAMP AS OF TIMESTAMP '2022-01-01 00:00:00.000000 Z';
SELECT count(*) FROM sales FOR VERSION AS OF 2188465307835585443;

-- Rollback to a specific time
CALL system.rollback_to_timestamp('sales', TIMESTAMP '2024-03-01 00:00:00');
```

#### 1.3.3. Đọc tăng dần

```sql
SELECT * FROM sales FOR CHANGES AS OF SNAPSHOT 123456789;
```

#### 1.3.4. Nén dữ liệu

```sql
CALL system.rewrite_data_files("sales");
```

#### 1.3.5. Cập nhật/Xóa bảng

```sql
-- Delete statement
DELETE FROM iceberg_table WHERE category='c3';

-- Update statement
UPDATE iceberg_table SET category='c2' WHERE category='c1';

-- Merge into statement
MERGE INTO accounts t 
    USING monthly_accounts_update s
    ON (t.customer = s.customer)
    WHEN MATCHED AND s.address = 'Centreville'
        THEN DELETE
    WHEN MATCHED
        THEN UPDATE
            SET purchases = s.purchases + t.purchases, address = s.address
    WHEN NOT MATCHED
        THEN INSERT (customer, purchases, address)
              VALUES(s.customer, s.purchases, s.address);
```

### 1.4. Tích hợp và tương thích

#### 1.4.1. Các công cụ xử lý dữ liệu

- Apache Spark
- Apache Flink
- Trino / Presto
- Apache Hive
- Snowflake
- AWS Athena
- Dremio
- Starburst
- ...

#### 1.4.2. Tương thích data lake

- S3
- GCS
- Azure Blob Storage

### 1.5. Tối ưu hóa

#### 1.5.1. Phân vùng dữ liệu

```sql
CREATE TABLE prod.db.sample (
    id bigint,
    data string,
    category string,
    ts timestamp)
USING iceberg
PARTITIONED BY (bucket(16, id), days(ts), category);
```

#### 1.5.2. Sắp xếp dữ liệu

```sql
CALL catalog_name.system.rewrite_data_files(
  table => 'db.sample', 
  strategy => 'sort', 
  sort_order => 'zorder(col_1, col_2)'
);
```

#### 1.5.3. Nén các tệp nhỏ

```sql
CALL catalog_name.system.rewrite_data_files('db.sample');
```

#### 1.5.4. Hết hạn các snapshot cũ

```sql
ALTER TABLE prod.db.sample SET TBLPROPERTIES (
    'history.expire.max-snapshot-age-ms'='432000000',
    'history.expire.min-snapshots-to-keep'='1'
);

CALL catalog_name.system.expire_snapshots('db.sample');
```

#### 1.5.5. Xóa các tệp mồ côi

```sql
CALL catalog_name.system.remove_orphan_files(table => 'db.sample');
```

#### 1.5.6. Quản lý lưu giữ snapshot

```sql
CREATE ICEBERG TABLE prod.db.sample()
    PARTITIONED BY $event_date
    TABLE_DATA_RETENTION = 5 days
```

#### 1.5.7. Tự động hóa các tác vụ tối ưu hóa

![Tự động hóa tối ưu hóa](/images/workshop/automation-screenshot.webp)

### 1.6. Apache Iceberg so với Delta Lake

Cả hai đều là định dạng bảng được thiết kế để hỗ trợ ACID trong Data lake.

#### 1.6.1. So sánh

| Tính năng | Apache Iceberg | Delta Lake |
| --- | --- | --- |
| Định nghĩa | Định dạng bảng Iceberg cung cấp hạ tầng có khả năng mở rộng với hỗ trợ nhiều công cụ xử lý. | Delta Lake là một lớp lưu trữ đáng tin cậy, đặc biệt phù hợp với hệ sinh thái Databricks. |
| Định dạng tệp | Iceberg hỗ trợ nhiều định dạng tệp, bao gồm Parquet, Avro và ORC. | Delta Lake chỉ hỗ trợ nguyên bản định dạng tệp Parquet. |
| Thuộc tính ACID | Iceberg hỗ trợ giao dịch ACID. | Delta Lake cung cấp các thuộc tính ACID mạnh mẽ. |
| Xử lý phân vùng | Iceberg hỗ trợ phân vùng động, nghĩa là phân vùng có thể được cập nhật mà không cần viết lại schema. | Phân vùng là cố định, và bạn nên định nghĩa chúng khi tạo bảng. Sửa đổi phân vùng đã định nghĩa có thể liên quan đến việc viết lại dữ liệu. |
| Du hành thời gian | Mỗi thay đổi được thực hiện trên bảng tạo ra một snapshot mới. | Cung cấp tính năng du hành thời gian thông qua nhật ký giao dịch, với các thay đổi được theo dõi trong tệp JSON. |
| Tích hợp | Iceberg hỗ trợ nhiều công cụ xử lý dữ liệu, như SQL, Spark, Trino, Hive, Flink, Presto, v.v. | Delta Lake được liên kết chặt chẽ với Apache Spark. |

#### 1.6.2. Trường hợp sử dụng

| Trường hợp sử dụng | Apache Iceberg | Delta Lake |
| --- | --- | --- |
| Linh hoạt công cụ | Tốt nhất khi sử dụng nhiều công cụ, bao gồm Apache Spark, Flink, Presto, Hive, v.v. Lý tưởng cho môi trường cần các công cụ khác nhau cho các tác vụ xử lý khác nhau. | Tốt nhất cho người dùng sử dụng Apache Spark nguyên bản, cung cấp tích hợp chặt chẽ và hiệu suất tối ưu trong hệ sinh thái Spark. |
| Truyền dữ liệu | Hỗ trợ nhập dữ liệu liên tục từ nhiều nguồn khác nhau, xử lý theo thời gian thực. | Thống nhất xử lý batch và stream, lý tưởng cho các trường hợp sử dụng yêu cầu cả hai trong một pipeline duy nhất. |

### 1.7. Các phương pháp tốt nhất

- Dọn dẹp tệp Manifest (Nén): Khi số lượng tệp dữ liệu tăng, hiệu suất có thể giảm, vì vậy hãy thực hiện nén thường xuyên.
- Thiết kế phân vùng cẩn thận: Ngay cả với Phân vùng ẩn, hãy chọn khóa phân vùng phù hợp xem xét hiệu suất truy vấn.
- Quản lý Catalog cẩn thận: Khi sử dụng AWS Glue hoặc Hive Metastore, quản lý quyền và chính sách phù hợp là điều cần thiết.
- Lựa chọn định dạng dữ liệu: Mặc dù Iceberg hỗ trợ Parquet, ORC và Avro, Parquet thường được khuyến nghị.

## 2. Amazon S3 Tables là gì?

### 2.1. Bối cảnh

Chúng ta đã sử dụng S3 cho nhiều workload khác nhau cho đến nay.

Trong số vô số workload đa dạng, tỷ lệ sử dụng S3 như một kho dữ liệu dạng bảng đang tăng đều đặn.

Đặc biệt, các tệp Parquet là một trong những loại dữ liệu phổ biến nhất và phát triển nhanh nhất trong S3.

Nói cách khác, các trường hợp sử dụng tải tệp Parquet lên S3 dựa trên định dạng bảng Apache Iceberg và sử dụng chúng thông qua các công cụ truy vấn khác nhau đang phát triển nhanh chóng.

Tuy nhiên, để duy trì và sử dụng dữ liệu dựa trên Apache Iceberg trên S3 cho phân tích, người dùng không thể tránh khỏi việc phải chú ý đến nhiều khía cạnh.

Ví dụ,

- Khi quy mô tăng, cần hiệu suất tốt hơn.
- Duy trì bảo mật ở cấp bảng không phải là vấn đề đơn giản.
- Tối ưu hóa chi phí lưu trữ có thể trở thành gánh nặng vận hành không mong đợi.

Amazon S3 Tables là dịch vụ được quản lý ra mắt để cho phép người dùng tập trung vào các thao tác dữ liệu mà không phải lo lắng về những khía cạnh này.

### 2.2. Lợi ích

- Khả năng mở rộng
  Đơn giản hóa data lake ở mọi quy mô trong môi trường Iceberg.
- Hiệu suất nâng cao
  Cung cấp hiệu suất truy vấn nhanh hơn tới 3 lần thông qua tối ưu hóa bảng liên tục so với các bảng Iceberg không được quản lý.
  Có thể xử lý nhiều giao dịch mỗi giây hơn tới 10 lần so với các bảng Iceberg được lưu trữ trong S3 bucket mục đích chung.
- Được quản lý hoàn toàn
  Thực hiện các tác vụ bảo trì bảng liên tục như nén, quản lý snapshot và xóa các tệp không được tham chiếu.
  Tự động tối ưu hóa hiệu quả truy vấn và chi phí theo thời gian.
- Tích hợp liền mạch
  Dễ dàng tích hợp với các dịch vụ AWS Analytics như Amazon Athena, Redshift, EMR thông qua Glue Catalog và tích hợp S3 Tables.
  Cũng tương thích với các công cụ mã nguồn mở phổ biến khác.
- Bảo mật đơn giản hóa
  Dễ dàng quản lý quyền truy cập vào bảng bằng cách tạo chúng dưới dạng tài nguyên AWS và áp dụng quyền thông qua tích hợp với Lake Formation.

### 2.3. Cấu trúc chi phí

https://aws.amazon.com/ko/s3/pricing/

- Lưu trữ S3: Chi phí lưu trữ S3 cho việc lưu trữ dữ liệu và metadata
  Giám sát: USD 0.025 cho mỗi 1000 đối tượng
  Đến 50TB/tháng: USD 0.0265/GB
  50-450TB/tháng: USD 0.0265/GB
  500TB+/tháng: USD 0.0242/GB
  Đối với S3 standard,
  Đến 50TB/tháng: USD 0.023/GB
  50-450TB/tháng: USD 0.022/GB
  500TB+/tháng: USD 0.021/GB
- Cuộc gọi API: Chi phí cuộc gọi API cho việc tạo, sửa đổi, xóa bảng và các thao tác truy vấn
  Yêu cầu PUT, POST, LIST mỗi 1000: USD 0.005
  Yêu cầu GET và tất cả yêu cầu khác mỗi 1000: USD 0.0004
- Bảo trì
  Nén - USD 0.004 cho mỗi 1000 đối tượng
  Nén - USD 0.05 cho mỗi GB dữ liệu được xử lý
