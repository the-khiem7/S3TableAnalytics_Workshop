---
title: "Tùy chọn 2: Không dùng Glow"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>6.4. </b>"
---

## Truy cập EMR workspace và mở notebook

{{% notice info %}}
Phần này hướng dẫn truy cập EMR Workspace đã thiết lập trong Điều kiện tiên quyết > Thiết lập EMR Workspace.
{{% /notice %}}

1. Truy cập trang [EMR Console](https://console.aws.amazon.com/emr/).

2. Chọn 'EMR Serverless' từ menu bên trái.

3. Trong phần 'EMR Studio' bên phải, chọn 'workshop_emr_studio' và nhấp nút 'Manage applications'.

4. Nhấp nút này sẽ mở trang EMR Studio trong cửa sổ mới.

5. Nhấp 'Workspaces' trong menu bên trái của EMR Studio.

6. Nhấp vào 'workshop-workspace' đã tạo trước đó.

7. Một cửa sổ mới với trang JupyterLab sẽ mở ra.

8. Trong menu bên trái của JupyterLab, nhấp để mở notebook 'workshop-workspace.ipynb' đã được tạo.

9. Chọn 'Pyspark' trong tùy chọn 'Select Kernel'.

![Mở notebook](/images/workshop/emr-open-notebook.webp)

Bạn đã sẵn sàng tạo Namespace và Table trong S3 Tables dựa trên EMR Serverless.

## Thiết lập Cấu hình Spark

{{% notice info %}}
Phần này thiết lập cấu hình Spark để sử dụng Amazon S3 Tables, AWS Glue và Apache Iceberg.
{{% /notice %}}

Nếu bạn gặp lỗi liên quan đến bộ nhớ hoặc được yêu cầu khởi động lại kernel sau khi thực thi mã configuration, khuyến nghị khởi động lại EMR Serverless Application như sau:

1. Chọn 'Serverless' > 'Applications' từ menu bên trái trên màn hình EMR Studio.
2. Chọn 'workshop_emr_application' và nhấp nút 'Stop application' để dừng.
3. Sau đó nhấp nút 'Start application' để khởi động EMR Serverless Application.
4. Xác nhận rằng cluster đã được gắn đúng trong notebook JupyterLab trước khi tiếp tục công việc.

Truy cập EMR Workspace (JupyterLab) đã tạo trước đó.

1. Tạo Notebook mới với Pyspark Kernel.

2. Thêm mã Configuration sau vào Cell đầu tiên của Notebook mới:

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://${bucket}/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "${table_bucket_arn}",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.hadoop.io.compression.codecs": "io.projectglow.sql.util.BGZFCodec"
    }
}
```

Mã này cấu hình Spark để sử dụng các thành phần sau:
- Cài đặt Catalog để sử dụng AWS Glue và Amazon S3 Tables
- Thêm thư viện để sử dụng Apache Iceberg

Giá trị `${bucket}` và `${table_bucket_arn}` cần được thay đổi phù hợp với môi trường của bạn. Đặc biệt, thay đổi giá trị `${table_bucket_arn}` thành giá trị ARN của Table bucket bạn đã sao chép trước đó.

3. Nhấp nút Run để thực thi Cell notebook.

![Cấu hình Spark](/images/workshop/lab3-2-configure.webp)

Bạn đã sẵn sàng sử dụng mã Pyspark tham chiếu đến Amazon S3 Tables, Glue và Apache Iceberg.

## Tải Dữ liệu VCF

{{% notice info %}}
Phần này tải dữ liệu VCF sử dụng Spark.
{{% /notice %}}

1. Nhấp nút '+' để thêm Cell.

2. Đọc dữ liệu VCF từ 1000genomes:

```python
from pyspark.sql.types import StructType, StringType 

vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr17.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

vcf_raw_schema = StructType() \
    .add("chrom", StringType()) \
    .add("pos", StringType()) \
    .add("id", StringType()) \
    .add("ref", StringType()) \
    .add("alt", StringType()) \
    .add("qual", StringType()) \
    .add("filter", StringType()) \
    .add("info", StringType()) \
    .add("format", StringType()) \
    .add("sample", StringType())

vcf_df = spark.read \
    .option('delimiter', '\t') \
    .schema(vcf_raw_schema) \
    .csv(vcf_file)
```

Không giống như Glow, phương pháp này tải file VCF bằng cách chỉ định schema. Nó đọc file VCF với schema bao gồm các cột chrom, pos, id, ref, alt, qual, filter, info, format và sample. Nó đọc file dưới dạng file CSV với `\t` làm dấu phân cách.

3. Nhấp nút '+' để thêm Cell.

4. Kiểm tra dữ liệu file VCF với mã sau:

```python
vcf_df.show(5, False)
```

Gọi hàm `show()` để kiểm tra 5 dòng dữ liệu.

![Tải file VCF](/images/workshop/lab3-2-load-vcf-data.webp)

Bạn đã tải file VCF từ S3 vào Spark Dataframe.

## Lấy dữ liệu body VCF và xử lý

{{% notice info %}}
Phần này biến đổi dữ liệu VCF đã tải thành dạng thuận tiện hơn cho phân tích.
{{% /notice %}}

1. Nhấp nút '+' để thêm Cell.

2. Thêm mã sau vào Cell mới và 'Run':

```python
sample_id = vcf_df \
    .filter(vcf_df.chrom.startswith("#CHROM")) \
    .selectExpr('sample') \
    .collect()[0]['sample']

sample_id
```

Mã này lấy giá trị sample_id từ file VCF.

3. Nhấp nút '+' để thêm Cell.

4. Thêm mã sau vào Cell mới và 'Run':

```python
from pyspark.sql.functions import struct, split, arrays_zip, map_from_entries, collect_list, col, lit

vcf_df = vcf_df \
    .filter(~(vcf_df.chrom.startswith("#"))) \
    .withColumn("sample_id", lit(sample_id)) \
    .withColumn("formats", split(col("format"), ':')) \
    .withColumn("samples", split(col("sample"), ':'))

vcf_df = vcf_df \
    .withColumn('zipped', arrays_zip(col('formats'), col('samples'))) \
    .withColumn('genomic', map_from_entries(col('zipped'))) \
    .select('sample_id', 'chrom', 'pos', 'id', 'ref', 
            'alt', 'qual', 'filter', 'info', 'genomic')
            
vcf_df = vcf_df \
    .groupBy("chrom", "pos", "id", "ref", "alt", "qual", "filter", "info") \
    .agg(collect_list(struct('sample_id', 'genomic')).alias("genomics"))
```

Mã này lọc các dòng mà cột chrom không bắt đầu bằng '#' để lấy dữ liệu body VCF. Các cột format và sample chứa giá trị chuỗi có thể tách bằng ':'. Các giá trị đã tách này có ánh xạ 1:1. Mã này biến đổi dataframe để đạt được ánh xạ 1:1 này.

5. Nhấp nút '+' để thêm Cell.

6. Thêm mã sau vào Cell mới và 'Run':

```python
vcf_df.show(5, False)
vcf_df.count()
```

Gọi hàm `show()` để kiểm tra dữ liệu đã biến đổi thực tế. Gọi hàm `count()` để kiểm tra số lượng dòng của dữ liệu đã xử lý. Kết quả nên xuất **1046733** dòng.

![Lấy dữ liệu body VCF](/images/workshop/lab3-2-get-vcf-data.webp)

Bạn đã biến đổi Spark Dataframe đã tải từ file VCF để lấy dữ liệu body.

## Tạo Bảng Dữ liệu VCF

{{% notice info %}}
Phần này tạo bảng trong S3 Tables từ dữ liệu VCF đã đọc.
{{% /notice %}}

1. Nhấp nút '+' để thêm Cell.

2. Thêm mã sau vào Cell mới:

```python
catalog = "s3table"
namespace = "vcf"
table = "vcf_delta_2"

vcf_df \
    .write \
    .saveAsTable(f"{catalog}.{namespace}.{table}")
```

Mã này gọi hàm `saveAsTable()` để tạo bảng `vcf_delta_2` từ dữ liệu VCF đã đọc trước đó.

3. Nhấp nút 'Run' để tạo bảng với dữ liệu VCF.

4. Để kiểm tra dữ liệu trong bảng đã tạo, nhấp nút '+' để thêm Cell.

5. Thêm mã sau vào Cell mới và 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

Gọi hàm `count()` để kiểm tra số lượng dòng dữ liệu. Bạn sẽ thấy **1046733** dòng, giống với số lượng dòng của file gốc.

![Tạo bảng VCF](/images/workshop/lab3-2-create-vcf-table.webp)

Bạn đã hoàn thành việc tạo bảng trong S3 Tables.

Từ đây trở đi, bạn có thể thực hiện các truy vấn phân tích bằng cách đọc dữ liệu vào Spark Dataframe, giống như đã làm với Glow.
