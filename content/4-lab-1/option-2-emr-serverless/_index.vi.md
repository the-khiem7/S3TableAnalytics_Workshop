---
title: "Tùy chọn 2: EMR Serverless"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>5.4. </b>"
---

## Truy cập EMR workspace và mở notebook

{{% notice info %}}
Phần này hướng dẫn truy cập EMR Workspace đã thiết lập trong Điều kiện tiên quyết > Thiết lập EMR Workspace.
{{% /notice %}}

Truy cập trang [EMR Console](https://console.aws.amazon.com/emr).

Chọn 'EMR Serverless' từ menu bên trái.

Trong phần 'EMR Studio' bên phải, chọn 'workshop_emr_studio' và nhấp nút 'Manage applications'.

Nhấp nút này sẽ mở trang EMR Studio trong cửa sổ mới.

Nhấp 'Workspaces' trong menu bên trái của EMR Studio.

Nhấp vào 'workshop-workspace' đã tạo trước đó.

Một cửa sổ mới với trang JupyterLab sẽ mở ra.

Trong menu bên trái của JupyterLab, nhấp để mở notebook 'workshop-workspace.ipynb' đã được tạo.

Chọn 'Pyspark' trong tùy chọn 'Select Kernel'.

![Mở notebook](/images/workshop/emr-open-notebook.webp)

Bạn đã sẵn sàng tạo Namespace và Table trong S3 Tables dựa trên EMR Serverless.

## Thiết lập cấu hình Spark

{{% notice info %}}
Phần này thiết lập cấu hình Spark để sử dụng Amazon S3 Tables, AWS Glue và Apache Iceberg.
{{% /notice %}}

Để kiểm tra giá trị ARN của Table Bucket đã tạo trước đó, mở cửa sổ trình duyệt mới và truy cập trang [S3 Console](https://console.aws.amazon.com/s3).

Chọn Table buckets từ menu bên trái.

Sao chép giá trị ARN (Amazon Resource Name) của Table bucket đã tạo.

![ARN của Table bucket](/images/workshop/table-bucket-arn.webp)

Nhấp nút '+' để tạo Cell mới và thêm mã Configuration sau:

**(Tùy chọn-1)** Đây là phương pháp thiết lập S3TablesCatalog.

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://<bucket>/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}        
```

- Giá trị `bucket` cần được thay đổi thành tên S3 bucket của bạn.
- Giá trị `table_bucket_arn` cần được thay đổi thành giá trị ARN của Table Bucket bạn đã tạo trước đó.
- `spark.driver.extraJavaOptions` và `spark.executor.extraJavaOptions` được thêm vào để tránh các vấn đề mã hóa ký tự tiếng Hàn tiềm ẩn.

**(Tùy chọn-2)** Đây là phương pháp thiết lập Amazon S3 Tables sử dụng REST.

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

Mã này cấu hình Spark để sử dụng các thành phần sau:
- Cấu hình catalog kiểu REST để sử dụng Amazon S3 Tables
- Thêm thư viện để sử dụng Apache Iceberg
- Giá trị `table_bucket_arn` cần được thay đổi thành giá trị ARN của Table Bucket bạn đã tạo trước đó.
- Nhập 'ap-northeast-2' hoặc 'us-east-1' cho giá trị region.

{{% notice info %}}
Có hai cách để thiết lập catalog trong mã Configuration.

(Tùy chọn-1) là phương pháp thiết lập S3TablesCatalog,

(Tùy chọn-2) thiết lập catalog kiểu REST.

Cả hai phương pháp đều hoạt động, nhưng để thuận tiện, chúng ta sẽ tiến hành với (Tùy chọn-1) trong lab này.

(Lưu ý) Phương pháp Tùy chọn-2 chưa hỗ trợ tạo bảng từ Dataframes.

Sử dụng `df.write.saveAsTable(...)` sẽ dẫn đến lỗi `Stage-create is currently not supported.`

Với phương pháp này, bạn cần tạo bảng bằng `spark.sql("CREATE TABLE ...")` rồi chèn dữ liệu bằng `df.write.InsertInto(...)`.
{{% /notice %}}

Nhấp nút Run để thực thi Cell notebook.

Bạn đã sẵn sàng sử dụng mã Pyspark tham chiếu đến Amazon S3 Tables, Glue và Apache Iceberg.

## Đọc Dữ liệu CSV từ S3

{{% notice info %}}
Phần này hướng dẫn đọc dữ liệu CSV từ S3 bằng Spark.

Nếu bạn gặp lỗi liên quan đến bộ nhớ hoặc được yêu cầu khởi động lại kernel sau khi thực thi mã configuration và chạy mã Spark, khuyến nghị khởi động lại EMR Serverless Application như sau:

Chọn 'Serverless' > 'Applications' từ menu bên trái trên màn hình EMR Studio.

Chọn 'workshop_emr_application' và nhấp nút 'Stop application' để dừng.

Sau đó nhấp nút 'Start application' để khởi động EMR Serverless Application.

Xác nhận rằng cluster đã được gắn đúng trong notebook JupyterLab trước khi tiếp tục công việc.
{{% /notice %}}

Truy cập EMR Workspace (JupyterLab) đã tạo trước đó.

Mở 'workshop_workspace.ipynb'.

Nhấp nút '+' để thêm Cell.

Đọc dữ liệu CSV đã tải lên S3:

```python
from pyspark.sql.types import StructType, StringType, DoubleType, DateType

bucket = '<bucket>'

schema = StructType() \
    .add("id", StringType()) \
    .add("legal_dong_code", StringType()) \
    .add("legal_dong_name", StringType()) \
    .add("category_code", StringType()) \
    .add("category_name", StringType()) \
    .add("land_number", StringType()) \
    .add("reference_year", StringType()) \
    .add("refernece_month", StringType()) \
    .add("disclosure_price", DoubleType()) \
    .add("disclosure_date", DateType()) \
    .add("is_standard", StringType()) \
    .add("base_date", DateType()) \
    .add("sido_code", StringType())

df = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv(f"s3://{bucket}/data/AL_D151_11_20240125_UTF8.csv")
```

- Thay thế `<bucket>` bằng tên S3 bucket phù hợp với môi trường của bạn.
- Định nghĩa schema rồi đọc dữ liệu CSV từ S3.

![Đọc file CSV](/images/workshop/emr-read-csv-data.webp)

Nhấp nút '+' để thêm Cell.

Kiểm tra dữ liệu file CSV với mã sau:

```python
df.show(5, False)
```

Gọi hàm `show()` để hiển thị 5 dòng dữ liệu.

Nhấp nút '+' để thêm Cell.

Kiểm tra số lượng dòng trong dữ liệu CSV với mã sau:

```python
df.count()
```

Gọi hàm `count()` để xuất số lượng dòng.
Bạn sẽ thấy có 897963 dòng.

File CSV từ S3 đã được tải vào Spark DataFrame.

## Tạo Bảng Dữ liệu

{{% notice info %}}
Phần này hướng dẫn tạo bảng trong S3 Tables từ dữ liệu CSV đã tải.
{{% /notice %}}

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới:

```python
catalog = "s3table"
namespace = "workshop_namespace"
table = "individually_disclosed_building_price_by_emr"

df.write \
    .option("encoding", "UTF-8") \
    .saveAsTable(f"{catalog}.{namespace}.{table}") 
```

Mã này gọi hàm `saveAsTable()` để tạo bảng có tên `individually_disclosed_building_price_by_emr` từ dữ liệu CSV đã tải trước đó.

Nhấp nút 'Run' để tạo bảng từ dữ liệu CSV.

![Tạo bảng](/images/workshop/emr-create-table.webp)

Để kiểm tra dữ liệu trong bảng đã tạo, nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

Gọi hàm `count()` để kiểm tra số lượng dòng dữ liệu.
Bạn sẽ thấy có 897963 dòng, giống như file gốc.

Bảng đã được tạo thành công trong S3 Tables.

## Merge Dữ liệu CSV

{{% notice info %}}
Phần này trình bày thực hiện truy vấn merge.
{{% /notice %}}

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
other_df = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv(f"s3://{bucket}/data/AL_D151_41_20250120_200000_UTF8.csv")

other_df.createOrReplaceTempView("other_source_table")
```

- Đọc dữ liệu file CSV khác (AL_D151_41_20250120_200000_UTF8.csv)
- File này kết hợp dữ liệu giá đất công khai 100.000 bản ghi từ Seoul và 100.000 bản ghi từ Gyeonggi-do.
- Nghĩa là, nó chứa tổng cộng 200.000 bản ghi, trong đó 100.000 trùng lặp với dữ liệu đã chèn vào S3 Tables trước đó.
- `other_df.createOrReplaceTempView("other_source_table")` — Phần này đăng ký DataFrame đã tải từ file CSV dưới dạng Temp View. Sau khi đăng ký, nó có thể được tham chiếu như một bảng trong các câu lệnh Spark SQL.

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
spark.sql(f"""
    MERGE INTO {catalog}.{namespace}.{table} t
    USING other_source_table s
    ON (s.id=t.id and s.legal_dong_code=t.legal_dong_code and 
        s.legal_dong_name=t.legal_dong_name and s.category_code=t.category_code and 
        s.category_name=t.category_name and s.land_number=t.land_number and 
        s.reference_year=t.reference_year and s.refernece_month=t.refernece_month and 
        s.disclosure_price=t.disclosure_price and s.disclosure_date=t.disclosure_date and 
        s.is_standard=t.is_standard and s.base_date=t.base_date and s.sido_code=t.sido_code)
    WHEN NOT MATCHED THEN
    INSERT (id, legal_dong_code, legal_dong_name, 
            category_code, category_name, land_number, 
            reference_year, refernece_month, 
            disclosure_price, disclosure_date, 
            is_standard, base_date, sido_code)
    VALUES (s.id, s.legal_dong_code, s.legal_dong_name, 
            s.category_code, s.category_name, s.land_number, 
            s.reference_year, s.refernece_month, 
            s.disclosure_price, s.disclosure_date, 
            s.is_standard, s.base_date, s.sido_code)
""")
```

Mã này thực thi truy vấn Merge sử dụng `spark.sql()`.
Trong truy vấn Merge:
- `{catalog}.{namespace}.{table}` là bảng Target để merge.
- `other_source_table` là bảng Source đã đăng ký làm Temp View trước đó.
- Mệnh đề ON đặt điều kiện khớp. Trong truy vấn này, nó kiểm tra xem tất cả các cột có giống nhau không.
- `WHEN NOT MATCHED THEN` theo sau bởi câu lệnh INSERT được thực thi cho dữ liệu không khớp điều kiện. Nghĩa là dữ liệu mới sẽ được chèn vào bảng target.

![Merge dữ liệu](/images/workshop/emr-merge-data.webp)

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

- Tải lại bảng.
- Kiểm tra số lượng dòng trong bảng.
- Như đã giải thích trước đó, dữ liệu sử dụng cho Merge chứa tổng cộng 200.000 bản ghi, trong đó 100.000 đã được tải trong bảng.
- Do đó, số lượng dòng chỉ nên tăng thêm 100.000 bản ghi.
- Xác nhận rằng 897963 + 100000 trở thành 997963 dòng, chứng minh Merge đã hoàn thành thành công.

Chúng ta đã merge thành công dữ liệu CSV vào bảng đã tạo trước đó. Nếu có thời gian, bạn có thể thử tải một file CSV khác lên S3 và merge nó.

## Cập nhật / Xóa Dữ liệu

{{% notice info %}}
Phần này hướng dẫn cập nhật hoặc xóa dữ liệu trong bảng S3 Tables.
{{% /notice %}}

### Cập nhật Dòng

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("legal_dong_code = '1111010100'").count()
s3tables_df.filter("legal_dong_code = '1111010100'").show(5, False)
```

Mã này gọi `filter("legal_dong_code = '1111010100'")` để đếm số dòng có 'legal_dong_code' là '1111010100'.
Bạn sẽ thấy có 821 dòng.

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
spark.sql(f"""
UPDATE {catalog}.{namespace}.{table} 
SET legal_dong_name = 'Seoul Jongno-gu Cheongunbong-dong' 
WHERE legal_dong_code = '1111010100'
""")
```

Mã này cập nhật `legal_dong_name` thành 'Seoul Jongno-gu Cheongunbong-dong' cho các dòng có `legal_dong_code` là '1111010100'.

![Cập nhật bảng](/images/workshop/emr-update-table.webp)

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("legal_dong_code = '1111010100'").show(5, False)
```

- Tải lại bảng.
- Kiểm tra các dòng có giá trị cột `legal_dong_code` là '1111010100'.
- Bạn sẽ thấy giá trị đã được thay đổi thành 'Seoul Jongno-gu Cheongunbong-dong'.

Việc cập nhật đã hoàn thành thành công.

### Xóa Dòng

{{% notice warning %}}
Ghi lại thời điểm bạn thực hiện truy vấn xóa. Điều này sẽ được sử dụng trong bài thực hành liên quan đến snapshot sau này.
{{% /notice %}}

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("is_standard = 'Y'").count()
s3tables_df.filter("is_standard = 'Y'").show(5, False)
```

Mã này gọi `filter("is_standard = 'Y'")` để đếm số dòng có 'is_standard' là 'Y'.
Bạn sẽ thấy có 30861 dòng.

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
spark.sql(f"""
DELETE FROM {catalog}.{namespace}.{table}
WHERE is_standard = 'Y'
""")
```

Mã này thực thi truy vấn DELETE cho các dòng có giá trị cột `is_standard` là 'Y'.

![Xóa bảng](/images/workshop/emr-delete-table.webp)

Nhấp nút '+' để thêm Cell.

Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("is_standard = 'Y'").count()
```

- Tải lại bảng.
- Kiểm tra số lượng dòng có giá trị cột `is_standard` là 'Y'.
- Xác nhận rằng đầu ra là 0 vì tất cả đã bị xóa.

Việc xóa đã hoàn thành thành công.

## Time Travel

### Lấy Lịch sử Bảng (Snapshots)

```python
spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.history
""").show(5, False)
```

```
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2025-04-02 02:26:45.356|1996392549628528971|NULL               |true               |
|2025-04-02 02:27:16.224|2842948084683911314|1996392549628528971|true               |
|2025-04-02 02:27:37.664|4907549559143542142|2842948084683911314|true               |
|2025-04-02 02:28:08.795|5291307565494698844|4907549559143542142|true               |
+-----------------------+-------------------+-------------------+-------------------+
```

Mã này xuất lịch sử Snapshot của bảng.
Nếu bạn thực hiện theo thứ tự của workshop,
- Dòng trên cùng là Snapshot khi bảng được tạo/Insert
- Dòng thứ hai là Snapshot khi thực hiện truy vấn Merge
- Dòng thứ ba là Snapshot khi thực hiện truy vấn Update
- Dòng thứ tư là Snapshot khi thực hiện truy vấn Delete.

### Lấy Thông tin Snapshot

```python
snapshot_id = "<snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.snapshots
WHERE snapshot_id = {snapshot_id} 
""").show(5, False)
```

Mã này xuất vị trí file metadata và dữ liệu liên quan đến file của Snapshot.

### Truy vấn Dòng với Update Snapshot

```python
update_snapshot_id = "<update_snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table} 
FOR VERSION AS OF {update_snapshot_id}
WHERE legal_dong_code = '1111010100'
""").show(5, False)
```

- Đặt giá trị `<snapshot_id>` thành ID Snapshot trước khi cập nhật.
- Bạn có thể xác nhận rằng dữ liệu đã được cập nhật thành 'Seoul Jongno-gu Cheongun-dong' được xuất với giá trị trước khi cập nhật.

### Truy vấn Dòng trước Thời điểm Xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE is_standard = 'Y'
""").show(5, False)
```

- Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu.
- Bạn có thể kiểm tra các Dòng đã bị xóa.

### Khôi phục sau Xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL {catalog}.system.rollback_to_timestamp('{namespace}.{table}', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM {catalog}.{namespace}.{table}
WHERE is_standard = 'Y'
""").show()
```

- Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu.
- Bạn có thể xác nhận rằng dữ liệu đã xóa trước đó đã được khôi phục.

Chúng ta đã cùng tìm hiểu các truy vấn Time Travel của Apache Iceberg.
