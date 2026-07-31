---
title: "Tạo S3 Tables"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>4.3. </b>"
---

## Truy cập EMR workspace và mở notebook

{{% notice info %}}
Đây là phần về việc truy cập EMR Workspace đã được thiết lập trong 'Điều kiện tiên quyết > Thiết lập EMR Workspace'.
{{% /notice %}}

1. Điều hướng đến trang [EMR Console](https://console.aws.amazon.com/emr/home).

2. Chọn 'EMR Serverless' từ menu bên trái.

3. Trong phần 'EMR Studio' bên phải, chọn 'workshop_emr_studio' và nhấp nút 'Manage applications'.

4. Sau khi nhấp nút này, một cửa sổ mới với trang EMR Studio sẽ mở ra.

5. Nhấp vào 'Workspaces' trong menu bên trái của EMR Studio.

6. Nhấp vào 'workshop-workspace' đã tạo trước đó.

7. Một cửa sổ mới với trang JupyterLab sẽ mở ra.

8. Nhấp nút '+' và chọn 'Notebook > PySpark' để tạo notebook mới.

9. Trong notebook đã tạo, nhấp nút '+' để thêm một Cell.

## Thiết lập cấu hình Spark

{{% notice info %}}
Đây là phần về việc thiết lập cấu hình Spark để sử dụng Amazon S3 Tables, AWS Glue và Apache Iceberg.
{{% /notice %}}

1. Để kiểm tra giá trị ARN của Table Bucket đã tạo trước đó, mở cửa sổ trình duyệt mới và truy cập trang [S3 Console](https://console.aws.amazon.com/s3/home).

2. Chọn Table buckets từ menu bên trái.

3. Sao chép giá trị ARN (Amazon Resource Name) của Table bucket đã tạo.

![Table bucket ARN](/images/workshop/table_bucket_arn.webp)

{{% notice info %}}
Có hai cách để cấu hình catalog trong mã Configuration.

(Option-1) là phương pháp thiết lập S3TablesCatalog,

(Option-2) thiết lập catalog dựa trên REST.

(Lưu ý) Phương pháp Option-2 hiện chưa hỗ trợ tạo bảng từ Dataframes.

Sử dụng df.write.saveAsTable(...) sẽ dẫn đến lỗi `Stage-create is currently not supported.`.

Với phương pháp này, bạn cần tạo bảng bằng spark.sql("CREATE TABLE ...") rồi chèn dữ liệu bằng df.write.InsertInto(...).

Cả hai phương pháp đều hoạt động, nhưng để thuận tiện trong lab này, chúng ta sẽ tiến hành với phương pháp (Option-1).
{{% /notice %}}

4. Nhấp nút '+' để tạo Cell mới và thêm mã Configuration sau.

**(Option-1)** Đây là phương pháp cấu hình với S3TablesCatalog.

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://<bucket>/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.defaultCatalog": "s3table",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}        
```

- Giá trị `<bucket>` cần được thay thế bằng tên S3 bucket của bạn.
- Giá trị `<table_bucket_arn>` cần được thay thế bằng ARN của Table Bucket bạn đã tạo trước đó.
- `spark.driver.extraJavaOptions` và `spark.executor.extraJavaOptions` được thêm vào để ngăn chặn các vấn đề tiềm ẩn với mã hóa ký tự tiếng Hàn.

**(Option-2)** Đây là phương pháp cấu hình sử dụng giao diện REST của Amazon S3 Tables.

```python
%%configure -f
{
    "conf": {
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.sql.defaultCatalog": "s3table",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.type": "rest",
        "spark.sql.catalog.s3table.uri": "https://s3tables.<region>.amazonaws.com/iceberg",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.catalog.s3table.rest.sigv4-enabled": "true",
        "spark.sql.catalog.s3table.rest.signing-name": "s3tables",
        "spark.sql.catalog.s3table.rest.signing-region": "<region>",
        "spark.sql.catalog.s3table.io-impl": "org.apache.iceberg.aws.s3.S3FileIO",
        "spark.sql.catalog.s3table.rest-metrics-reporting-enabled": "false",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}
```

- Mã trên cấu hình Spark để cho phép các tính năng sau:
- Thiết lập catalog kiểu REST để sử dụng Amazon S3 Tables
- Thêm thư viện cần thiết để sử dụng Apache Iceberg
- Thay thế giá trị `<table_bucket_arn>` bằng ARN của Table Bucket đã tạo trước đó.
- Đặt giá trị `<region>` thành 'ap-northeast-2' hoặc 'us-east-1'.

5. Nhấp nút Run để thực thi ô notebook.

Bây giờ bạn đã sẵn sàng sử dụng mã PySpark với Amazon S3 Tables, AWS Glue và Apache Iceberg.

{{% notice warning %}}
Nếu bạn gặp lỗi liên quan đến bộ nhớ hoặc thông báo khởi động lại kernel sau khi chạy mã cấu hình rồi thực thi mã Spark, khuyến nghị khởi động lại EMR Serverless Application như sau:

1. Trong menu bên trái của EMR Studio, vào 'Serverless' > 'Applications'.
2. Chọn 'workshop_emr_application', sau đó nhấp nút 'Stop application' để dừng.
3. Sau đó, nhấp nút 'Start application' để khởi động EMR Serverless Application.
4. Trong notebook JupyterLab, đảm bảo cluster được gắn đúng cách, sau đó tiếp tục công việc.
{{% /notice %}}

## Đọc dữ liệu CSV từ S3 và tạo bảng

{{% notice info %}}
Phần này mô tả cách sử dụng Spark để đọc dữ liệu CSV từ S3 và tạo bảng.
{{% /notice %}}

1. Truy cập EMR Workspace (JupyterLab) mà bạn đã tạo trước đó.

2. Nhấp nút '+', sau đó chọn 'Notebook > PySpark' để tạo notebook mới.

3. Trong notebook mới tạo, nhấp nút '+' lần nữa để thêm ô mới.

4. Sao chép và dán mã bên dưới để thiết lập các biến cần thiết cho tác vụ, và cập nhật giá trị `<bucket>` cho phù hợp với môi trường của bạn.

```python
datasets = [
    "allergies",
    "careplans",
    "conditions",
    "devices",
    "encounters",
    "imaging_studies",
    "immunizations",
    "medications",
    "observations",
    "organizations",
    "patients",
    "payer_transitions",
    "payers",
    "procedures",
    "providers"
]

bucket = "<bucket>"
catalog = "s3table"
namespace = "workshop_namespace"
```

- Thay thế `<bucket>` bằng tên S3 bucket phù hợp với môi trường của bạn.

5. Nhấp nút Run để thực thi ô notebook.

6. Nhấp nút '+' để thêm ô mới.

7. Dán mã bên dưới vào ô mới thêm.

```python
for dataset in datasets:
    df = spark.read.option("header", True).csv(f"s3://{bucket}/data/coherent/{dataset}.csv")
    df = df.toDF(*[col.lower() for col in df.columns])
    df.write \
        .saveAsTable(f"{catalog}.{namespace}.{dataset}")     
```

- Mã trên đọc 15 tệp CSV đã tải lên S3 vào DataFrame, sau đó tạo bảng Iceberg trong Amazon S3 Tables.
- Nó lặp qua 15 bộ dữ liệu và tuần tự tạo các bảng.

{{% notice info %}}
Khi tạo bảng cho mục đích sản xuất thực tế, khuyến nghị chỉ định các cột phân vùng như bên dưới.

Dữ liệu được lưu trữ theo phân cấp dựa trên các cột phân vùng.

Cấu trúc này cho phép cắt tỉa dữ liệu dựa trên các cột phân vùng trong quá trình truy vấn, ảnh hưởng đáng kể đến hiệu suất truy vấn trong tương lai.

Hiện tại, chúng ta sẽ không chỉ định phân vùng để đảm bảo trải nghiệm thực hành suôn sẻ. (Đây chỉ là thông tin tham khảo)
{{% /notice %}}

```python
# from pyspark.sql.functions import dayofmonth, substring, col

# df = df.withColumn("day", dayofmonth("reg_date")) \
#     .withColumn("id_prefix", substring(col("id").cast("string"), 1, 2))

# df.write \
#     .partitionBy("day", "id_prefix") \
#     .saveAsTable("database_name.table_name")    
```

8. Nhấp nút Run để thực thi ô notebook.

Các tệp CSV trong S3 đã được tải vào Spark DataFrames và các bảng đã được tạo thành công.

## (Tùy chọn) Kiểm tra bảng

{{% notice info %}}
Hãy kiểm tra xem các bảng đã tạo trước đó có được tạo đúng cách không.
{{% /notice %}}

1. Nhấp nút '+' để thêm ô mới.

2. Sao chép và dán mã bên dưới vào ô mới thêm.

```sql
%%sql
SHOW TABLES FROM s3table.workshop_namespace
```

3. Nhấp nút Run để thực thi ô notebook.

4. Nếu các bảng được hiển thị như bên dưới, quy trình đã hoàn tất thành công.

![Kiểm tra bảng](/images/workshop/table-check.webp)

5. Chạy các truy vấn bên dưới để kiểm tra dữ liệu trong các bảng.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.observations
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.conditions
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.medications
```

Dữ liệu CSV trong S3 đã được đọc thành công vào Spark DataFrame và tạo thành các bảng Iceberg.
